---
sidebar_position: 1
id: s3-cloudfront-cdn
title: S3 & CloudFront CDN for Static Assets
sidebar_label: S3 & CloudFront CDN
---

# S3 CloudFront CDN

This document outlines the infrastructure setup for serving static files from the `statics.numinia.xyz` domain. The configuration leverages several AWS services working together to provide a secure, scalable, and efficient solution for static asset delivery.

## Overview

The static assets hosting infrastructure is built using Terragrunt to manage the following AWS services:

1. **S3 Bucket**: Storage for static files
2. **CloudFront**: Content Delivery Network for global distribution
3. **ACM (Certificate Manager)**: SSL/TLS certificate management
4. **Route53**: DNS management

The solution implements Origin Access Control (OAC) to ensure the S3 bucket is not directly accessible from the internet, with CloudFront serving as the secure gateway to the content.

## Architecture Diagram

```
┌─────────────┐     ┌──────────────┐     ┌────────────────┐     ┌──────────────┐
│   Route53   │───▶│   CloudFront  │───▶│     S3 Bucket  │───▶│      ACM      │
│ DNS Records │     │ Distribution │     │  Static Files  │     │ Certificate  │
└─────────────┘     └──────────────┘     └────────────────┘     └──────────────┘
                           ▲                                           │
                           └───────────────────────────────────────────┘
                                           SSL/TLS
```

## Infrastructure Components

### S3 Bucket Configuration

The S3 bucket (`statics.numinia.xyz`) is configured with:

- **Public Access**: Blocked at bucket level
- **Encryption**: Server-side encryption with AES256
- **Versioning**: Enabled for file version history
- **CORS**: Configured to allow cross-origin requests
- **Bucket Policy**: Specifically allows access only from the CloudFront distribution using Origin Access Control

Key settings:
```hcl
# Block public access
block_public_acls       = true
block_public_policy     = true
ignore_public_acls      = true
restrict_public_buckets = true

# Bucket policy for CloudFront access
policy = {
  Statement = [
    {
      Action    = "s3:GetObject"
      Effect    = "Allow"
      Principal = { Service = "cloudfront.amazonaws.com" }
      Resource  = "arn:aws:s3:::statics.numinia.xyz/*"
      Condition = {
        StringEquals = {
          "AWS:SourceArn" = "<CloudFront-Distribution-ARN>"
        }
      }
    }
  ]
}
```

### CloudFront Configuration

The CloudFront distribution is configured to:

- Serve content from the S3 bucket securely using Origin Access Control (OAC)
- Redirect HTTP to HTTPS
- Use the ACM certificate for SSL/TLS
- Optimize caching for static assets

Key settings:
```hcl
# Origin Access Control for S3
origin_access_control = {
  "statics_s3_oac" = {
    description      = "CloudFront OAC for statics.numinia.xyz S3 bucket"
    origin_type      = "s3"
    signing_behavior = "always"
    signing_protocol = "sigv4"
  }
}

# Origin configuration
origin = {
  statics-s3-origin = {
    domain_name         = "${s3_bucket_domain_name}"
    origin_id           = "statics-s3-origin"
    origin_access_control = "statics_s3_oac"
  }
}

# Cache behavior
default_cache_behavior = {
  viewer_protocol_policy = "redirect-to-https"
  allowed_methods        = ["GET", "HEAD", "OPTIONS"]
  cached_methods         = ["GET", "HEAD"]
  compress               = true
}
```

### ACM Certificate Configuration

The SSL/TLS certificate is:

- Created in the `us-east-1` region (required for CloudFront)
- Validated via DNS (Route53)
- Set up for the domain `statics.numinia.xyz`

Key settings:
```hcl
# Certificate configuration
domain_name = "statics.numinia.xyz"
validation_method = "DNS"
create_route53_records = true
wait_for_validation = true
```

### Route53 Configuration

DNS settings include:

- An alias record pointing to the CloudFront distribution
- Configured in the existing Route53 hosted zone

Key settings:
```hcl
# Route53 record for CloudFront
records = [
  {
    name    = "statics"
    type    = "A"
    alias   = {
      name    = "${cloudfront_distribution_domain_name}"
      zone_id = "${cloudfront_hosted_zone_id}"
      evaluate_target_health = false
    }
  }
]
```

## Security Considerations

This setup implements several security best practices:

1. **No public S3 access**: The bucket is completely private and accessible only through CloudFront
2. **Origin Access Control (OAC)**: Modern security approach (replacing Origin Access Identity) that uses signing to verify CloudFront requests
3. **HTTPS Only**: All traffic is encrypted using TLS
4. **Bucket Policy Conditions**: The S3 bucket policy includes a condition to only allow requests from a specific CloudFront distribution

## Variable Management

To simplify maintenance and ensure consistency, all infrastructure-specific values (ARNs, IDs, etc.) are centralized in the `env.hcl` file:

```hcl
# CloudFront & S3 config
route53_zone_id = "Z0876112W5KAVVY2CCR"
cloudfront_hosted_zone_id = "Z2FDTNDATAQYW2"
cloudfront_distribution_arn = "arn:aws:cloudfront::241533135482:distribution/E2LDZPKGHZK9AU"
cloudfront_oac_id = "EW8GHULW1L1RO"
cloudfront_oac_name = "statics_s3_oac"
statics_s3_bucket_name = "statics.numinia.xyz"
```

## Deployment Process

The infrastructure is deployed using Terragrunt in the following order:

1. ACM Certificate (must be in `us-east-1` region)
2. S3 Bucket
3. CloudFront Distribution
4. Route53 Record

Each component is defined in a separate directory with its own `terragrunt.hcl` file, allowing for modular management.

## Usage

To upload static assets to the infrastructure:

1. Upload files to the S3 bucket `statics.numinia.xyz`
2. Access files via `https://statics.numinia.xyz/{file-path}`

For example, a file uploaded as `logo.png` would be accessible at `https://statics.numinia.xyz/logo.png`.

## Maintenance and Troubleshooting

Common troubleshooting areas:

- **Access Denied Errors**: Check S3 bucket policy and CloudFront OAC configuration
- **Certificate Validation Issues**: Ensure DNS validation records are properly set in Route53
- **Cache Issues**: CloudFront may cache content; use cache invalidation if immediate updates are required

## Future Improvements

Potential enhancements to consider:

- Implementing AWS WAF for additional security