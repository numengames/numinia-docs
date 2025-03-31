---
sidebar_position: 1
id: s3-cloudfront-implementation
title: S3 & CloudFront Terragrunt Implementation
sidebar_label: S3 & CloudFront Implementation
---

# Static Assets Terragrunt Architecture

This document describes the code structure and Terragrunt implementation details for our static assets infrastructure. It focuses on how the various AWS components are integrated using Terragrunt modules and dependencies.

## Directory Structure

The static assets infrastructure is organized in the following directory structure:

```
infrastructure/
├── production/
│   ├── env.hcl                          # Shared environment variables
│   ├── eu-west-1/
│   │   └── <organization>/
│   │       ├── s3/
│   │       │   └── statics-bucket/
│   │       │       └── terragrunt.hcl   # S3 bucket configuration
│   │       ├── cloudfront/
│   │       │   └── statics-distribution/
│   │       │       └── terragrunt.hcl   # CloudFront distribution configuration
│   │       └── route53/
│   │           └── statics-record/
│   │               └── terragrunt.hcl   # Route53 DNS record configuration
│   └── us-east-1/
│       └── <organization>/
│           └── acm/
│               └── statics-certificate/
│                   └── terragrunt.hcl   # ACM certificate configuration
└── root.hcl                             # Root Terragrunt configuration
```

## Module Dependencies

The infrastructure components have the following dependencies:

1. **S3 Bucket**: No dependencies
2. **CloudFront Distribution**: Depends on S3 Bucket and ACM Certificate
3. **Route53 Record**: Depends on CloudFront Distribution
4. **ACM Certificate**: No dependencies, but uses Route53 for validation

## Terragrunt Configuration Details

### Environment Variables (`env.hcl`)

The `env.hcl` file centralizes all environment-specific variables, including module sources and infrastructure IDs. For static assets, it includes:

```hcl
# Module sources
s3_module_source = "git::https://github.com/terraform-aws-modules/terraform-aws-s3-bucket.git"
acm_module_source = "git::https://github.com/terraform-aws-modules/terraform-aws-acm.git"
cloudfront_module_source = "git::https://github.com/terraform-aws-modules/terraform-aws-cloudfront.git"

# Infrastructure IDs
route53_zone_id = "<route53-zone-id>"
cloudfront_hosted_zone_id = "<cloudfront-hosted-zone-id>"
cloudfront_distribution_arn = "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
cloudfront_oac_id = "<cloudfront-oac-id>"
cloudfront_oac_name = "<statics-s3-oac-name>"
statics_s3_bucket_name = "<statics-domain>.xyz"

# Cache policies
cache_policy_id = "<cache-policy-id>"
origin_request_policy_id = "<origin-request-policy-id>"
```

### S3 Bucket (`s3/statics-bucket/terragrunt.hcl`)

The S3 bucket configuration:

```hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

include "envcommon" {
  path = find_in_parent_folders("env.hcl")
  expose = true
  merge_strategy = "no_merge"
}

terraform {
  source = "${include.envcommon.locals.s3_module_source}?ref=v3.15.1"
}

inputs = {
  bucket = include.envcommon.locals.statics_s3_bucket_name
  
  # Block public access
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
  
  # S3 bucket policy for CloudFront OAC
  attach_policy = true
  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Sid = "AllowCloudFrontServicePrincipalReadOnly",
        Effect = "Allow",
        Principal = {
          Service = "cloudfront.amazonaws.com"
        },
        Action = "s3:GetObject",
        Resource = "arn:aws:s3:::${include.envcommon.locals.statics_s3_bucket_name}/*",
        Condition = {
          StringEquals = {
            "AWS:SourceArn" = include.envcommon.locals.cloudfront_distribution_arn
          }
        }
      }
    ]
  })
  
  # Additional bucket configurations...
}
```

### CloudFront Distribution (`cloudfront/statics-distribution/terragrunt.hcl`)

The CloudFront distribution includes dependencies on both S3 and ACM:

```hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

include "envcommon" {
  path = find_in_parent_folders("env.hcl")
  expose = true
  merge_strategy = "no_merge"
}

# Dependencies
dependency "acm" {
  config_path = "../../../../us-east-1/<organization>/acm/statics-certificate"
  mock_outputs = {
    acm_certificate_arn = "arn:aws:acm:us-east-1:<account-id>:certificate/<certificate-id>"
  }
}

dependency "s3_bucket" {
  config_path = "../../s3/statics-bucket"
  mock_outputs = {
    s3_bucket_id = include.envcommon.locals.statics_s3_bucket_name
    s3_bucket_bucket_regional_domain_name = "${include.envcommon.locals.statics_s3_bucket_name}.s3.eu-west-1.amazonaws.com"
    s3_bucket_arn = "arn:aws:s3:::${include.envcommon.locals.statics_s3_bucket_name}"
  }
}

terraform {
  source = "${include.envcommon.locals.cloudfront_module_source}?ref=v3.2.1"
}

inputs = {
  # Origin configuration
  origin = {
    statics-s3-origin = {
      domain_name = dependency.s3_bucket.outputs.s3_bucket_bucket_regional_domain_name
      origin_id = "statics-s3-origin"
      origin_access_control = include.envcommon.locals.cloudfront_oac_name
    }
  }
  
  # OAC Configuration
  create_origin_access_control = true
  origin_access_control = {
    "${include.envcommon.locals.cloudfront_oac_name}" = {
      description = "CloudFront OAC for ${include.envcommon.locals.statics_s3_bucket_name} S3 bucket"
      origin_type = "s3"
      signing_behavior = "always"
      signing_protocol = "sigv4"
    }
  }
  
  # SSL Certificate
  viewer_certificate = {
    acm_certificate_arn = dependency.acm.outputs.acm_certificate_arn
    ssl_support_method = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }
  
  # Additional CloudFront configurations...
}
```

### Route53 Record (`route53/statics-record/terragrunt.hcl`)

The Route53 record depends on the CloudFront distribution:

```hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

include "envcommon" {
  path = find_in_parent_folders("env.hcl")
  expose = true
  merge_strategy = "no_merge"
}

# Dependency on CloudFront
dependency "cloudfront" {
  config_path = "../../cloudfront/statics-distribution"
  mock_outputs = {
    cloudfront_distribution_domain_name = "<cloudfront-distribution-domain>"
    cloudfront_distribution_hosted_zone_id = include.envcommon.locals.cloudfront_hosted_zone_id
  }
}

terraform {
  source = "git::https://github.com/terraform-aws-modules/terraform-aws-route53.git//modules/records?ref=v2.10.2"
}

inputs = {
  zone_id = include.envcommon.locals.route53_zone_id
  
  records = [
    {
      name = split(".", include.envcommon.locals.statics_s3_bucket_name)[0]
      type = "A"
      alias = {
        name = dependency.cloudfront.outputs.cloudfront_distribution_domain_name
        zone_id = dependency.cloudfront.outputs.cloudfront_distribution_hosted_zone_id
        evaluate_target_health = false
      }
    }
  ]
}
```

### ACM Certificate (`acm/statics-certificate/terragrunt.hcl`)

The ACM certificate configuration, which must be in `us-east-1` for CloudFront:

```hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

include "envcommon" {
  path = find_in_parent_folders("env.hcl")
  expose = true
  merge_strategy = "no_merge"
}

terraform {
  source = "${include.envcommon.locals.acm_module_source}?ref=v5.0.0"
}

# Special configuration for ACM in us-east-1 but state in eu-west-1
remote_state {
  backend = "s3"
  config = {
    bucket = "<terraform-state-bucket-name>"
    key = "production/us-east-1/<organization>/acm/statics-certificate/tf.tfstate"
    region = "eu-west-1"
  }
}

inputs = {
  domain_name = include.envcommon.locals.statics_s3_bucket_name
  zone_id = include.envcommon.locals.route53_zone_id
  validation_method = "DNS"
  create_route53_records = true
  wait_for_validation = true
}
```

## Key Implementation Details

### Circular Dependency Management

A potential circular dependency exists between CloudFront and S3:
- CloudFront needs the S3 bucket domain
- S3 bucket policy needs the CloudFront distribution ARN

To handle this, we use:
1. In initial deployment:
   - First create S3 with no policy
   - Create CloudFront
   - Update S3 with the proper policy

2. In the code:
   - Use centralized variables in `env.hcl` for ARNs once infrastructure exists
   - Use `mock_outputs` for dependencies during initial creation

### Cross-Region Resources

ACM certificates used with CloudFront must be in the `us-east-1` region. Our configuration handles this by:

1. Placing ACM in the correct `us-east-1` region folder
2. Setting a custom `remote_state` configuration to store the state in our primary region
3. Referencing the certificate across regions via dependencies

### OAC Implementation

Origin Access Control (OAC) is AWS's newer replacement for Origin Access Identity (OAI). Our implementation:

1. Creates an OAC in the CloudFront configuration
2. References the OAC in the origin settings
3. Applies a specific bucket policy in S3 that validates requests using the `AWS:SourceArn` condition

## Deployment Flow

The recommended deployment sequence is:

```bash
# 1. Deploy ACM Certificate
cd infrastructure/production/us-east-1/<organization>/acm/statics-certificate
terragrunt apply

# 2. Deploy S3 Bucket
cd ../../../../../production/eu-west-1/<organization>/s3/statics-bucket
terragrunt apply

# 3. Deploy CloudFront Distribution
cd ../../cloudfront/statics-distribution
terragrunt apply

# 4. Deploy Route53 Record
cd ../../route53/statics-record
terragrunt apply
```

## Maintenance Procedures

### Adding or Modifying Variables

1. Edit the `env.hcl` file to add or modify centralized variables
2. Reference the variables in the respective module using `include.envcommon.locals.variable_name`

### Updating Module Versions

Module versions are pinned in each `terragrunt.hcl` file using the `?ref=vX.Y.Z` syntax. To update:

1. Update the version reference in the `source` attribute
2. Run `terragrunt plan` to verify compatibility
3. Apply the changes if no issues are detected

### Troubleshooting Common Issues

1. **S3 Policy Issues**: Check that the CloudFront distribution ARN in `env.hcl` matches the actual distribution
2. **Certificate Validation**: Ensure the certificate is in `us-east-1` and validation records are created
3. **OAC Configuration**: Verify that the OAC name in CloudFront matches the reference in the S3 policy 