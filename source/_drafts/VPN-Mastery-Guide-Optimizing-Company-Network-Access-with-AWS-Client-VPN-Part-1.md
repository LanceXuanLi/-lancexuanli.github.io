---
title: 'VPN Mastery Guide: Optimizing Company Network Access with AWS Client VPN (Part 1)'
tags: 
  - 'Devops'
  - 'AWS'
  - 'Cloud'
  - 'Security'
  - 'VPN' 
  - 'AWS Client VPN'
  - 'Mutual authentication'
---

# VPN Mastery Guide: Optimizing Company Network Access with AWS Client VPN (Part 1)

## Introduction
AWS Client VPN is a managed client-based VPN service that enables you to securely access your AWS resources and resources in your on-premises network. With Client VPN, you can access your resources from any location using an OpenVPN-based VPN client. 

Today, we will be discussing how to set up AWS Client VPN by Terraform and describe the process of connecting to the VPN from a client machine.

This is the first part of the series of articles about AWS Client VPN. In the next article, we will discuss how to use GitHub Actions to automate the process of creating and updating the AWS Client VPN endpoint.
## Prerequisites
- AWS Account
- Terraform (latest version)
- OpenVPN client (https://openvpn.net/client/)
- AWS CLI (latest version)
- AWS CLI profile configured with access to your AWS account
- Existed VPC with public subnets and private subnets.

## Details of AWS Client VPN
### Features
AWS Client VPN supports the following features:
- **Mutual authentication**: AWS Client VPN supports mutual authentication with certificate-based clients for a more secure VPN connection.
- **Active Directory**: AWS Client VPN supports Active Directory authentication for Windows-based clients.
- **Federated authentication**: AWS Client VPN supports federated authentication with the Security Assertion Markup Language (SAML) 2.0 standard.

We will be using mutual authentication in this article.

### Mutual authentication
Mutual authentication performs like belows: 
- Mutual authentication requires that the client and the server have a certificate that is signed by a common certificate authority (CA). 
- The client presents its certificate to the server during authentication. 
- The server validates the client certificate and authenticates the client. 

You can use AWS managed certificate authority (ACM) to generate the certificate, but it is not free. We can use easy-rsa to generate the client certificate and server certificate as AWS suggests.

#### Use easy-rsa to generate the client certificate and server certificate
Easy-RSA is a utility for managing X.509 PKI, an X.509 certificate binds an identity to a public key using a digital signature.

1. Install easy-rsa

### Pricing

## Architecture

![Architecture](https://images.lancexuanli.site/VPN-Mastery-Guide-Optimizing-Company-Network-Access-with-AWS-Client-VPN-Part-1-architecture.png)
_Architecture diagram of AWS Client VPN, set VPN in multiple zones to improve resilience_


## Steps

### 1. Create a 