autoscale: true
footer: Intro to CloudFront CDN
theme: Letters from Sweden, 4

![original right](https://cdn2.iconfinder.com/data/icons/amazon-aws-stencils/100/Storage__Content_Delivery_Amazon_CloudFront-512.png)

# [fit] Intro to CloudFront

## [fit] Fast, highly secure and programmable content delivery network

### Calvin Hendryx-Parker
#### Six Feet Up

[.hide-footer]

---
# [fit] The Problem
* _Litterally has to do with physics_

![fit right](CloudPing.info.png)

[.hide-footer]
[.build-lists: true]

^ Can't work around the speed of light (yet)

---

# What is a CDN?

![inline](https://d1.awsstatic.com/global-infrastructure/maps/CloudFront%20Network%20Map%2010.12.18.59e838df2f373247d2efaeb548076e084fd8993e.png)

^ 144 points of presence (PoPs) world wide

^ reducing bandwidth costs, improving page load times, or increasing global availability of content

---
![](https://c2.staticflickr.com/4/3837/14569747628_d39ab86b56_o.jpg)

# [fit] Common CDNs

![inline, fit](http://nephoscale.com/wp-content/uploads/2015/02/akamai-technologies-logo-centered.png) ![inline, fit](https://upload.wikimedia.org/wikipedia/commons/9/94/Cloudflare_Logo.png)
![inline, fit](https://www.photospng.com/uploads/fastly-logo-image.png)

^ Many Telcos also provide CDN capability (AT&T, CenturyLink, etc.)

^ some are free such as Cloudflare and specific ones that deliver common open-source libraries such as BoostrapCDN or JSDelivr

---
![](https://d1.awsstatic.com/global-infrastructure/maps/CloudFront%20Network%20Map%2010.12.18.59e838df2f373247d2efaeb548076e084fd8993e.png)

# Why CloudFront?

* Integrated with AWS
* Programmable via Lambda@Edge
* Authorized access via Signed URLs or Cookies
* Additional Security such as AWS Shield
* Compliance (PCI-DSS 1, HIPPA, ISO 27001, SOC)

^ S3, EC2, ELB, Route 53, AWS Elemental Media Services (transcoder)

^ CloudFront infrastructure and processes are compliant

---
![](SFUDistributions.png)

# Distributions

---
![right](SFU-testing-dist.png)

## Features

* Custom Domain Names
* Websockets
* Origin Groups
* RTMP
* Custom Origins
* HTTP/2

^ Changes to distributions can take 20-30 minutes to propagate

---

# Configuring DNS

```
www.sixfeetup.com.      3576    IN      CNAME   sixfeetup.com.
sixfeetup.com.          3571    IN      A       52.84.64.215
sixfeetup.com.          3571    IN      A       52.84.64.177
sixfeetup.com.          3571    IN      A       52.84.64.236
sixfeetup.com.          3571    IN      A       52.84.64.10
sixfeetup.com.          3571    IN      A       52.84.64.32
sixfeetup.com.          3571    IN      A       52.84.64.99
sixfeetup.com.          3571    IN      A       52.84.64.157
sixfeetup.com.          3571    IN      A       52.84.64.240
```

^ Recommended is to use Route 53, it will handle domain aliases automatically

---

![](SFU-Inspector.png)

---
![fit](SFU-Post-CF-WebPageTest.png)

[.hide-footer]

---

![](Workplace-Inspector.png)

^ Example without custom DNS

^ Point out the HTTP/2 working here

---

# Origins

![fit](https://d2908q01vomqb2.cloudfront.net/cb4e5208b4cd87268b208e49452ed6e89a68e0b8/2017/11/06/1-1024x576.png)

---

![](Default-Origin.png)

^ Multiple Origins for Half and Full Bridge HTTPS

---

# S3

![fit](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2018/06/27/thumbnail.png)

^ Can point to a public bucket, but always better to authenticate requests with OAI

---

![](Workplace-OAI-Origin.png)

^ Uses OAI to authenticate requests to a private S3 bucket

^ Automatically places an ACL on our bucket for the Identity

---
![](https://c1.staticflickr.com/3/2922/14756067232_60f5f5b746_o.jpg)

# Behaviors

---
![](Behavior-Listing.png)

---
![](https://mdn.mozillademos.org/files/14295/CORS_principle.png)

# [fit] CORS

^ Cross Origin Request Sharing

---

![](CORS-Behavior.png)

^ Normally for an S3 bucket you wouldn't whitelist headers or pass cookies

---

![](CORS-S3-Config.png)

---

![fit](https://d2908q01vomqb2.cloudfront.net/cb4e5208b4cd87268b208e49452ed6e89a68e0b8/2017/11/06/1-1024x576.png)

---
![](https://c2.staticflickr.com/4/3873/14756063512_e61f04cdcb_o.jpg)

# Lambda@Edge

* Customize Content Delivered
* Inspect Cookies
* Rewrite Requests
* Make Network Calls
* Access the Request Body in POST/PUT
* Must be created in Node.js:registered: runtime

^ New as of August of this year

^ $.60 per 1M requests compared to standard Lambda at $.20 per 1M

^ Duration cost are also 200% more than standard lambda

^ Other limitations: no DLQ, no env vars, can't access resources in your VPC

---
![](https://c2.staticflickr.com/6/5593/14569527080_8c82481967_o.jpg)

# Common Lambda@Edge Uses

* Return 301 or 302 redirects
* Origin failover
* POST contact form data to DynamoDB
* Gathering web beacon data

^ limit of 100 triggers per CF distribution and have a maximum of 25 distributions with triggers

^ 4 trigger events (viewer req, origin req, origin resp, viewer resp)

^ each cache behavior can have up to four triggers

---
![](https://c1.staticflickr.com/3/2940/14569716990_49992d73d2_o.jpg)

# [fit] Reports

---

![](CloudFront-Stats.png)

[.hide-footer]

^ Also includes Status Codes and 

---
![](CloudFront-Alarm.png)

[.hide-footer]

---
![fit](CloudFront-Popular.png)

[.hide-footer]

^ Other reports such as Referrers, Viewers (Devices, Browsers, OS, Country) and Usage

---

# Example Deployments

![inline](https://github.com/aws-samples/aws-refarch-wordpress/raw/master/images/aws-refarch-wordpress-v20171026.jpeg)


* https://github.com/aws-samples/aws-refarch-drupal
* https://github.com/aws-samples/aws-refarch-moodle
* https://github.com/aws-samples/aws-refarch-wordpress

---
![](https://c2.staticflickr.com/4/3897/14569954677_2910dcfc69_o.jpg)

# [fit] Security

* Origin Access Identity
* Field-Level Encryption
* AWS Certificate Manager
* AWS Shield Standard
* AWS WAF
* https://github.com/aws-samples/aws-waf-sample

^ Shield Standards provides comprehensive availability protection against all known infrastructure (Layer 3 and 4) attacks

^ Shield Advanced requires 1 year min and $3k/mo, but has 24/7 access to the AWS DDoS Response Team and includes WAF

^ WAF has similar charges to Lambda@Edge, $0.60/million requests and $1 per rule and can go higher if you use managed rules

---
[.background-color: #FFFFFF]

![fit](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/11/15/image1_numbereddiagram_b1.png)

---
![](https://c2.staticflickr.com/6/5583/14569727270_efdaa0651f_o.jpg)

# Costs

* Data Transfer Out
* HTTP/HTTPS Requests
* Invalidation Requests
* Field Level Encryption Requests
* Dedicated IP SSL :arrow_left: **Not Needed Generally**

^ Free Tier

^ no cost for internal transfer between CloudFront and AWS Services such as S3 and EC2 or ELB

---

# Resources

* https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide
* https://github.com/awsdocs/amazon-cloudfront-developer-guide
* https://docs.aws.amazon.com/cloudfront/latest/APIReference

^ Docs are open source and you can make pull requests

^ Docs are licensed under Creative Commons Attribution-ShareAlive 4.0

---
![](https://c2.staticflickr.com/6/5595/14569736558_2526ac547a_o.jpg)

# [fit] Questions:grey_question:


### [@calvinhp](http://twitter.com/calvinhp)
### [calvin@sixfeetup.com](mailto:calvin@sixfeetup.com)
