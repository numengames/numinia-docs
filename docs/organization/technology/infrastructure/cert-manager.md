---
sidebar_position: 4
id: cert-manager
title: SSL Certificate Management
sidebar_label: SSL Certificates
---

# SSL Certificate Management

This guide is specifically for teams and projects that are using Numinia's infrastructure as their hosting provider. If you're already running your application on our infrastructure, we provide automatic SSL certificates to keep your websites secure.

## Why Use Our SSL Management?

When you choose to host your application with us, we include:
- Automatic SSL certificate setup and renewal
- Security monitoring and maintenance
- Technical support for your domain configuration
- The padlock icon in browsers that users trust
- All our software (components, services, AI Agents, etc.)

All of this comes configured and managed as part of our infrastructure service - there's no additional setup needed on your end except for one simple domain setting.

## Overview

We take care of all the technical aspects of SSL certificates, including:
- Setting up your space in our system
- Configuring all security settings
- Automatic certificate renewals
- Continuous monitoring

As a user, you only need to update one DNS setting to connect your domain to our system.

## What We Handle

Our team manages everything technical:
1. Creating your secure space in our platform
2. Setting up all security configurations
3. Connecting your domain to our systems
4. Making sure your SSL certificate stays valid
5. Monitoring everything works correctly

## What You Need to Do

You only need to do one thing: configure your domain to point to our servers. Here's how:

### Domain Configuration Steps

1. Go to your domain provider's website (like GoDaddy, Cloudflare, etc.)
2. Find the DNS or Domain Settings section
3. Add a new CNAME record with:
   - **Name**: The subdomain you want to use (example: if you want `app.mydomain.com`, just enter `app`)
   - **Value**: Our server address (we'll send this to you)

Example configuration:
```dns
Name: app
Type: CNAME
Value: [Server address we'll provide]
```

## What Happens Next

After you've updated your domain settings:
1. Let us know you've made the change
2. We'll check everything is working correctly
3. Your SSL certificate will be automatically set up

## How to Check It's Working

You can verify everything is working by:
1. Wait about 15 minutes for the changes to take effect
2. Visit your website using `https://` at the start
3. Look for the padlock icon in your browser's address bar

## Need Help?

If something's not working:
1. Double-check your domain settings are correct
2. Contact us - we're here to help
3. We'll check everything on our end and help you fix any issues

Remember: We handle all the complex technical parts - you just need to point your domain to our servers.

## Support

Need assistance?
- Just reach out to our team
- We'll help you verify your domain settings
- We'll make sure your website is secure

Our goal is to make this process as simple as possible for you while ensuring your website is secure and reliable. 