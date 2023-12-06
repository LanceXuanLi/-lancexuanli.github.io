---
layout: drafts
title: >-
  VPN Mastery Guide: Optimizing Company Network Access with AWS Client VPN
tags:
  - Devops
  - AWS
  - Cloud
  - Security
  - VPN
  - AWS Client VPN
  - Mutual authentication
date: 2023-12-06 21:58:55
---


# VPN Mastery Guide: Optimizing Company Network Access with AWS Client VPN

## Introduction
AWS Client VPN is a managed client-based VPN service that enables you to securely access your AWS resources and resources in your on-premises network. With Client VPN, you can access your resources from any location using an OpenVPN-based VPN client. 

Today, we will be discussing how to set up AWS Client VPN by Terraform and describe the process of connecting to the VPN from a client machine.

This is the first part of the series of articles about AWS Client VPN. \
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
> Easy-RSA is a utility for managing X.509 PKI, an X.509 certificate binds an identity to a public key using a digital signature.

### Pricing
> AWS Client VPN endpoint association	$0.10 per hour 
> 
>AWS Client VPN connection	$0.05 per hour
## Architecture

![Architecture](https://images.lancexuanli.site/VPN-Mastery-Guide-Optimizing-Company-Network-Access-with-AWS-Client-VPN-Part-1-architecture.png)
_Architecture diagram of AWS Client VPN, set VPN in multiple zones to improve resilience_


## Steps

###  Create a server cert and client certs
1. Install [easy-rsa](https://github.com/OpenVPN/easy-rsa).
2. Generate the CA certificate and key.
```shell
# after theres commands you will get the ca.crt and ca.key in the pki/ca directory
cd easy-rsa/easyrsa3
./easyrsa init-pki
./easyrsa build-ca
```
3. Generate the server certificate and key.
```shell
# after theres commands you will get the server.crt and server.key in the pki/issued directory
# we will upload the server.crt to AWS certificate manager to create a server certificate 
# and then use it to create the AWS Client VPN.
cd easy-rsa/easyrsa3
./easyrsa build-server-full server
```
4. Generate the client certificate and key.
```shell
# after theres commands you will get the client.crt and client.key in the pki/issued directory
# you should create a client name for each client (colleagues in your company!)
cd easy-rsa/easyrsa3
./easyrsa build-client-full <client-name>
```

### Upload the server certificate to AWS Certificate Manager
```shell
aws acm import-certificate --certificate fileb://server.crt --private-key fileb://server.key --certificate-chain fileb://ca.crt```
```

### Create a Client VPN endpoint
```terraform
resource "aws_security_group" "test" {
  # your vpc id
  vpc_id = "xxx"
  # your security group name
  name   = "xxx"

  ingress {
    from_port   = 1194
    to_port     = 1194
    protocol    = "udp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "Incoming VPN connection"
  }

  egress {
    from_port   = 0
    protocol    = "-1"
    to_port     = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_cloudwatch_log_stream" "test" {
  # your log stream name
  name           = "xxx"
  log_group_name = aws_cloudwatch_log_group.test.name
}

resource "aws_cloudwatch_log_group" "test" {
  # your log group name
  name = "xxx"
}

resource "aws_ec2_client_vpn_endpoint" "test" {
  vpc_id                 = "xxx"
  description            = "xxx"
  
  # if false, all trips will go through the vpn, it may cost you more money
  split_tunnel           = true
  transport_protocol     = "udp"
  vpn_port               = 1194
  # server certificate arn, find it in AWS Certificate Manager
  server_certificate_arn = "xxx"
  # session timeout hours
  session_timeout_hours  = 10
  # your vpn cidr block (should be different from your vpc cidr block)
  client_cidr_block      = "xxx"
  
  security_group_ids     = [aws_security_group.test.ids]
  # you can use your own dns servers
  dns_servers            = ["1.1.1.1", "8.8.8.8"]
  
  connection_log_options {
    # use this it will log all the traffic
    enabled               = true
    cloudwatch_log_group  = aws_cloudwatch_log_group.test.name
    cloudwatch_log_stream = aws_cloudwatch_log_stream.test.name
  }

  authentication_options {
    type                       = "certificate-authentication"
    # server certificate arn, find it in AWS Certificate Manager
    root_certificate_chain_arn ="xxx"
  }
  
  tags = {
    # your tags
    Name = "xxx"
  }

  client_login_banner_options {
    # your banner text
    banner_text = "xxx"
    enabled     = true
  }
  
}



resource "aws_ec2_client_vpn_route" "test" {
  client_vpn_endpoint_id = aws_ec2_client_vpn_endpoint.test.id
  # your vpc cidr block
  destination_cidr_block = "xxx"
  target_vpc_subnet_id   = aws_ec2_client_vpn_network_association.test.subnet_id
}

resource "aws_ec2_client_vpn_authorization_rule" "test" {
  client_vpn_endpoint_id = aws_ec2_client_vpn_endpoint.test.id
  # your vpc cidr block
  target_network_cidr    = "xxx"
  authorize_all_groups   = true
  description            = "xxx"
}

resource "aws_ec2_client_vpn_network_association" "test" {
  client_vpn_endpoint_id = aws_ec2_client_vpn_endpoint.test.id
  # one of your vpc subnet id
  subnet_id              = "xxx"
}
```

## How to use AWS Client VPN.
Send the client certificate and key to your colleagues, and they can use the OpenVPN client to connect to the VPN.
Read this [page](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/cvpn-working-endpoint-export.html) to know how to export the client certificate and key.
