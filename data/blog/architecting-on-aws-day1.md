---
title: 'Architecting on AWS - Day 1, Sydney'
date: 2014-03-26 00:00
draft: false
tags: ['AWS']
summary: 'Notes from the Architecting on AWS course held in Sydney'
---

These are my notes from day 1 of the Amazon 'Architecting on AWS' course which was run in Sydney in March 2014. The trainer/presenter was John Rotenstein from AWS.

# Best Practices

## Design for failure

- **Design for failure**, nothing fails. E.g. Aviation, seat belts, "black box"
- "Everything fails, all the time"
- Avoid single points of failure

## Loose coupling sets you free

- SOA
- More loosely they couple, the bigger they scale
- Every component should be a "black box" (as in Aviation)

- ELB is a loose coupling technique to loosely couple tiers
- Implement a queue

## Implement elasticity

- Ability to get big, and get small
- Meet requirements of demand
- Auto provision and replace
- Use designs resilient to reboot and re-launch
- Self-configuration (pull down latest code and config)

## Build security in every layer

- "Security is the number one requirement for AWS" - we care a lot about security
- Encrypt data
- Concept of least privilege
- Security groups
- Common use of EC2 instance hijacking is to mine bitcoins
- Multi factor auth

## Don't fear constraints

- Just because you have a massive server, in house, does not mean you need that in the cloud. Consider distribution of load.
- Rather than fixing an underlying problem on an EC2 instance, throw it away and start again (or reboot). "Treat your instances like cattle, rather than pets". Charlottes Web, initial paragraph

## Think in Parallel

- Job takes 4 hours, do it on 4 servers in 1 hour!

## Leverage many storage options

- One size does not fit all
- S3, EBS, NoSQL, RDBMS

# AWS Cloud

- People choose regions for data governance
- AWS will not move data between regions
- Choose a region close to your customers [Cloud ping](http://cloudping.info)
- [Global infrastructure page](http://aws.amazon.com/about-aws/globalinfrastructure)
- Edge locations are located near Tier 1 providers

## What makes a region?

- Regions contain multiple availability zones
- Different power connections, internet connections, flood plains etc.
- Same general metropolitan area in city
- Low latency between AZ's
- Some services are global, some are regional (e.g. S3)

## Regional

### EC2

- Amazon Linux is based on CentOS
- Community AMI's have other OS's than the stock OS's
- VM import/export allows you to import or export VM's to and from EC2
- Hypervisor is highly customised version of Xen
- CPUs are not shared
- RAM is not shared (Except in micro instances)
- ECU --> Relative measure of compute power. Compared to m1.small (m1.small == 1 ECU)
- Prices vary by region, and are always in USD
- Sometimes better machines might be cheaper (e.g. m1.large vs m3.large)
- When a Windows instance is run, you pay for the Windows license
- Instance store is a directly attached disk storage. It is "ephemeral"
- EBS is persistent

- [NetFlix use NoSQL Cassandra](http://slideshare.net/jericevans/cassandra-explained) on SSD instance store
- You get better network bandwidth on larger instances
- EBS optimised does not share the NIC
- Micro should not be used for production
- Micro could be used for small sites, e.g. blogs (average amount of times a blog is read, never)

[@jrotenstein's](http://twiter.com/jrotenstein) Enterprise Architecture advice:

1. Price
2. Trial
3. Documentation (RTFM)

### EBS

- Attach to EC2
- Max size is 1TB, Min size is 1GB
- You choose the file system

### AMI

- AMI's are lazy loaded, before the AMI has been copied you can start the machine
- Look at the marketplace
- You can add a test-drive version of your software as an AMI

### RDS

- "The great thing about RDS is that you don't need a DBA"
- Multi-AZ deployment (Except SQL Server)
- Automated backups
- Easy to restore PROD DB to DEV DB (WooHooooo!)

- "AWS is one massive API engine"
- "Team should not be bigger than you can feed with 2 pizzas"
- [AWS Blog](http://aws.typepad.com) - week in review
- [AWS Podcast](http://aws.amazon.com/apac/awspodcast)
- [AWS re:Invent](http://reinvent.awsevents.com/recap.html)

## Global

### S3

- Object storage
- Individual objects up to 5TB (Use multi-part upload for large files)
- 11 9's of durability - 3 copies across multiple AZ's (or DC's in the case of Sydney)
- Unlimited
- S3 supports bit torrent protocol

### DynamoDB

- NoSQL
- "Extremely fast"
- Managed
- Sharding
- Replicated to 3 facilities (AZ/DC)

- Running across 2 or more AZ's will ensure higher availability
- Best way to ensure this is using auto-scaling groups

# Security

## General

- Customer is responsible for patching of OS and Applications
- Latest updates in AMI only
- [Penetration Testing](http://aws.amazon.com/security/penetration-testing) is a violation to ToS. You can request permission...
- One of the best defences to DDoS is scale. [DDoS resiliency with AWS](https://www.youtube.com/watch?v=V7vTPlV8P3U)
- EBS is not encrypted by default
- AWS will not port/vulnerability scan your application
- [AWS Trusted Advisorhttp://aws.amazon.com/premiumsupport/trustedadvisor) will recommend places to streamline your AWS instances. **Only available to Business level support ($100 pm or 10% of AWS bill pm)**
- Enterprise level support gives "Infrastructure Event Management" which is dedicated AWS staff on call when a significant event is happening
- AWS is PCI DSS Level 1 compliant. **Applications are your responsibility to certify**.
- Each AWS Region has at least one DR Availability Zone

## Security

- AWS operates under "shared security" model
  - AWS responsible for security **of** the cloud
  - Customer responsible for security **in** the cloud
- It is the responsibility of the customer to configure the provided tools
- AWS is responsible for physical access
- CTO is not allowed in any DC

### AWS Responsibility

#### Network security

- DDoS
- MITM
- IP Spoofing - source/destination check
- No port scanning
- No packet sniffing

#### Storage security

- Storage devices are destroyed at end of life
  - Degaussing
  - Powdering of SSD
- First time you use a block on a disk, it is reset to 0

### Customer Responsibility

- AWS Account management
- Isolation of accounts
  - Consolidating billing
- Isolate by environments
  - DEV doesn't access TEST or PROD
  - No accidental changing PROD DB etc.
- EC2 protected by key pairs
  - Only you have the private key
  - Instances have public key
- OS Security patching
- Data encryption
  - In transit (HTTPS)
  - On disk
- AWS CloudHSM - dedicated [SafeNet Luna SA](http://aws.amazon.com/cloudhsm/faqs/)
- Use multiple layers of defence
  - Security groups
  - Bastion host
  - Host based firewalls
  - IDS/IPS

### Security groups

- Firewall around each instance
- Based on [CIDR](http://en.wikipedia.org/wiki/CIDR)
- EC2 instances can be associated with multiple groups

- [AWS Security Best Practices white paper](http://media.amazonwebservices.com/AWS_Security_Best_Practices.pdf)
- [AWS Security Overview of Security Processes white paper](http://d36cz9buwru1tt.cloudfront.net/pdf/AWS_Security_Whitepaper.pdf)

- ** Most security best practices still apply in the cloud **
- [The Cuckoo's Egg](http://books.google.com.au/books/about/CUCKOO_S_EGG.html?id=0q1_5QkqV8EC&redir_esc=y)

- "The Stopinator" - script which closes down `dev`/`temp` tagged instances at 7pm

# VPC

1. Specify private IP address range. ** Do not overlap with existing DC **
2. Divide into public and private subnets
3. Control inbound and outbound access to instances
4. Assign multiple IP address and attach NIC (ENI - Elastic Network Interface) and EIP (Elastic IP Address)

- Elastic IP address limit is 5 per region
- You can use `vHost` style routing from a reverse proxy appliance (e.g. HAProxy)

## VPC Deployment

### Wizard based option

1. VPC with public subnet (only)
2. VPC with public and private subnet.
   - Private subnet connects to interwebs through NAT server
3. VPC with public and private subnets, and Hardware VPN
   - Connection to office, DC etc.
   - IPSec connection
4. VPC with private subnet (only), and Hardware VPN

### Manual configuration

1. Define a VPC with a CIDR range (e.g. 10.0.0.0/16)
2. Tenancy can be dedicated (e.g. on a box with no other customer)
3. Choose whether to enable DNS & DHCP
4. Create subnets with a CIDR range (e.g. 10.0.0.0/24) for each AZ and public/private you need
5. Add an instance to the VPC
6. Generate an Internet Gateway (logical connection), and attach it to your VPC
7. Create a new route table for the VPC
8. Add a route to the internet to the Internet Gateway
9. Associate the Internet Gateway to your public subnets

You can now access the instance.

- VPC can run across multiple AZ's
- Subnets must be within an AZ (e.g. 10.0.0.0/24)

- [NAT white paper](http://aws.amazon.com/articles/2781451301784570)

## ACLs

- Network Access Control is stateless (you must allow in and out)

## ENIs

- ENI's have a MAC address (licensing impact if licensed by MAC address)
- 1+ private IP
- Security groups are associated with ENIs
  - You have different SG's on each ENI
- Can be deleted on termination... or not

[Amazon VPC Network Connectivity Options](http://media.amazonwebservices.com/AWS_Amazon_VPC_Connectivity_Options.pdf)

# Identity, Authentication and Authorization

## AWS APIs

- REST - access key & secret access key (e.g. S3)
- Console - username & password
- SOAP (deprecating) - certificate X.509

## Multi Factor Auth

- One Time Password standard
- Actions can be configured to require MFA

## Master account

- When you sign up to AWS you get a master account, with a key and a secret
- This account can do _anything_
- Do not use the master account
- Assign a **hardware** token for MFA

## IAM

- Users
- Groups
  - Permissions (e.g. allow sysadmins to do anything, except blah)
  - Rotate access key
  - Assign password rotation policy
- Roles
  - Allow applications to securely access other services (e.g. Java accessing S3)
  - Allow cross-account access
  - Allow you to use roles, rather than embedding access codes

## Authorisation Policies

- JSON format
- Action (API)
  - Which API they are allowed to access e.g. `s3::GetObject` of `s3:Get*`
- Resource (some services only)
  - Applies to specific resources, e.g. s3 bucket
- Conditions (optional)
  - Applies specific conditions
    - SSL
    - IP address
    - MFA
    - etc...
- There is a policy generator

## Temporary Credentials

- Allowing access for SSO
- e.g. Permission to call an AWS API using AD credentials
- "Token vending machine"
- Automatic expiry (15mins to 36 hours)
- Can be linked to ADFS
- AWS STS - security token service
- Can be used to allow people to place items directly into AWS (e.g. upload photo to S3)
- Can negate the need for a service layer on EC2
- Google, Facebook and Amazon authentication are available on AWS
- IAM is not to be used for application authentication
