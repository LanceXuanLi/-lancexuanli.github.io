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

## Architecture

![Architecture](https://images.lancexuanli.site/VPN-Mastery-Guide-Optimizing-Company-Network-Access-with-AWS-Client-VPN-Part-1-architecture.png)
