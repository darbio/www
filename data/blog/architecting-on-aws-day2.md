---
title: 'Architecting on AWS - Day 2, Sydney'
date: 2014-03-26 01:00
draft: false
tags: ['AWS']
summary: 'Notes from the Architecting on AWS course held in Sydney'
---

These are my notes from day 2 of the Amazon 'Architecting on AWS' course which was run in Sydney in March 2014. The trainer/presenter was John Rotenstein from AWS.

# Route 53

- AWS DNS provider
- Split DNS? Is this possible?
- Subdomains can CNAME to an aws address (e.g. assets.s3.aws.com)

## Routing

- Round robin
  - Weighted
    - 0 will stop traffic going to that address
  - Equal
- Latency based

  - Serve closest (least latency) instance

- Health checks (check for return string from page)

# Elastic Load Balancers

- Distribute load across availability zones
- Sits behind a DNS round robin (which sends traffic to a region)
- Never cache the IP address of a load balancer - i.e. **use the DNS name**
- ELB's can expand and contract based upon load
- Health checks can occur
  - TCP
  - HTTP (check for 200 response)
  - HTTPS (200 response)
  - SSL
- HTTPS offloading can be enabled at the ELB
- ELB can have a security group around it
- Supports port 80, 443 and >1024 (< 1024 may have problems)
- Cross zone balancing can be turned on (e.g. balance between AZ's)
- Connection draining keeps users on the same machine, even if the health check fails

- Not as advanced as F5
- F5 has an AMI
  - Not fully managed, ELB is fully managed
- **WebSockets will not work over ELB as there is a 60 second timeout. To use WebSockets you will need to implement a keepalive call before the timeout.**
- It is advisable to keep state in a DB, rather than in a web server, as you can't guarantee the same web-server will be served to the client

# Cloud Front

- CDN
- Can be pointed to:
  - S3 bucket
  - EC2
  - Own web server
- Edge locations are very big
- CloudFront can alias directories, regex (`*.htm`)
  - Serve dynamic content from web server on EC2
  - Serve static data from S3
  - Serve login from your own web server
- Signed URLs are a secure way to access CDN data (access token and timestamp appended to URL)
- Manual invalidation of data is possible using the API/Console
- Real Time Media Playback distribution is possible for media stored in S3

- AWS PodCast, 850Gb in 2013, cost AWS $101...
- Charges are per hit, e.g. lots of hits on small files will cost more

# Cloud Watch

- Monitors and Alerts
- Monitoring of most AWS services is provided at no charge
- EC2 is free at 5 minute monitoring
  - Data returned will show what the HyperVisor can see
    - CPU
    - Disk
    - Network
  - You can use scripts to return other stats
    - Memory
    - Swap space
    - Disk space
  - Aggregated metrics on Auto-Scaling Groups (e.g. if the average CPU over the group is >80% trigger an alarm)
- ELB has metrics for alerting/monitoring
- You can visualise metrics in the CloudWatch console
- 10 metrics and 10 alarms free, per month
- CloudWatch only retains data for 14 days

# Elastic Beanstalk

- Allows developers to deploy code to AWS without worrying about infrastructure
- Push with git, or upload code
- For each tier, you deploy separate EBS instances
  - Presentation
  - Application
    - This tier can have an RDS attached, however, when you delete the tier, you will lose your data
- Logs are pushed to S3

# Cloud Formation

- Allows us to create templates which will deploy an environment, e.g. DEV, PROD and TEST etc.
- Templates are JSON
- Every developer can have a 'mini production' environment
- Great way to reduce change management, just use the template
- Store in `git` and use a `diff` to show changes
- [CloudFormer](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-cloudformer.html) is a tool to generate templates
- [Madeira Cloud](http://www.madeiracloud.com/) to "draw" templates

# Elasticity, Scalability and Bootstrapping

## Elasticity

- Provision resources 'just in time'
- Ensure instances are highly utilised all the time
- You can use elasticity to scale out and in, based upon expected load variation
- Example: [Animoto](http://animoto.com/)
- There is a default limit on the amount of EC2 instances, ~20 by default

## Scalability

### Patterns

- Automagical processes
- Loosely coupled servers (use an ELB)
- Stateless (use no state or use DynamoDB)
- Horizontal scaling
- SOA

### Anti Patterns

- Manual process
- Tight coupling
- Stateful
- Vertical scaling

## Bootstrapping

- Automatic setup of servers
  1. OS
  2. Copy data
  3. Config etc...
- Scripts
  - 'User data' on instance (shell, powershell, cmd, etc.)
    - `#!` on linux
    - `<script>` or `<powershell>` on windows
    - Only runs the first time an instance is run - it does not run again if you stop and start an instance
  - 'User data' shows up as metadata in the instance:
    - `http://169.254.169.254/latest/user-data` from host
    - This can be used to inject config variables, e.g. as JSON string
- Chef, Puppet, etc.
- Amazon OpsWorks

### AMI

- OS
- Frameworks
- Code

1. Create full AMI
   - Fast to boot
   - New AMI every time you need to change the code
2. Create a half AMI
   - Medium boot time
3. Nothing on the AMI
   - Longer to boot and start
4. Hybrid
   - Chef / Puppet
   - Generate an AMI from the new codebase, each time it is changed
   - This is used by NetFlix
     - Netflix AMI Builder - [AMInator](https://github.com/Netflix/aminator)
   - Best of all worlds

## Autoscaling

- Can be scheduled, and also reactive to metrics
- Launch configuration == What do you want to run
- Auto scaling group == Where do you want to run
  - Subnet
  - Which configuration to use
  - Scaling policies
- Autoscaling can be used to **maintain amount of instances**, not just for scaling up and down
  - `max` of 1 and `min` of 1, will maintain the server at ... `1`!
  - `Conformity Monkey` at NetFlix
- If an AZ goes down, and there is a rush to get compute resources in an AZ, AWS may not grant instances to the customer................. **ouch** (you can reserve compute, but it will not be guaranteed)
- Auto scaling group termination policies can be set
  1. `OldestLaunchConfiguration` will allow graceful upgrading (e.g. scale up to double for a day, then scale back to normal)
  2. `ClosestToNextInstanceHour` will save money
  3. `OldestInstance`
  4. `NewestInstance`
  5. `Default` oldest instance terminated

# Data Storage

- POSIX vs Object Store
  - Different paradigm
- Be creative - use storage alternatives such as in-memory caching!

## EBS

- POSIX compliant
- 'SnapShottable'
  - Used blocks are copied to S3... but you cannot see them!
  - Incremental
- Can be added as a drive to an instance
- You cannot resize an EBS volume, you must snapshot and then restore to a new EBS volume from the snapshot
- EBS volumes are not run across AZ's
- Multiple 1TiB volumes can be striped together for disks which need to be larger than 1TiB
- EBS volumes work better with random IO, instance stores are better otherwise
- Do not use EBS for
  - Temporary storage, use Instance store
  - Very high durability
  - Storing static web content, use S3
  - Structured data, use RDS or DynamoDB

## Instance / Ephemeral

- Free if you choose an instance with attached storage
- You lose the contents of that drive when you stop or restart the instance
- About same speed as EBS standard
- Good for temporary data:
  - Buffer
  - Cache etc...

### Anti Patterns

- Do not use for
  - Persistant storage
  - Database
  - Backup
  - Not sharable

## S3

- Object, not POSIX (File)
- WORM - Write Once, Read Many
- Eventual consistency
- 99.999999999% durability
  - Copied to 3 facilities
  - Reduced redundancy can be chosen, for a lower cost e.g. use for thumbnails, etc. 99.9999%
- Unlimited storage capacity
- Folders do not exist... it is an illusion!
- Key names have 1024 length limit
- Server Side Encryption can be applied
  - Secure, 3 way access
- `aws s3 sync` will sync a directory, or a file to s3

### Eventual Consistency

- Synchronous storage on create
- EC on update and delete
- [Eventually Consistent](http://www.allthingsdistributed.com/2008/12/eventually_consistent.html) - Werner Voegels
- [CAP Theorum](en.wikipedia.org/wiki/Cap_Theorum)

## Glacier

- Long term archival storage system
- 3-5 hour retrieval time
- Access by API only
- 99.999999999% durability
- Pay for accessing data
