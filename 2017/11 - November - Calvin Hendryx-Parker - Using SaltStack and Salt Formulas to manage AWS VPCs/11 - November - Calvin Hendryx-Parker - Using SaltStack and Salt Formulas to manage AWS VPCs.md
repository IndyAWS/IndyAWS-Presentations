autoscale: true
footer: SaltStack AWS Formulas — IndyAWS Nov 2017
theme: Scherzkeks, 7

![SaltStack Logo](SaltLogo.png)

# [fit] SaltStack

## and AWS Formulas

## Creating VPCs

---

![right](Cal.jpg)

# Calvin Hendryx-Parker

## Six Feet Up CTO and Co-Founder

---

![original 140%](Founders.jpeg)

## **What is Salt?**

### **Event Driven Datacenter Orchestration**

^ Tom Hatch
  Marc Chen

----

# What sets Salt apart from other configuration management tools?

* Remote Execution
* Event-Driven Orchestration
* Agent or Agent-less Operation
* Cloud Provisioning
* Speed and Scalability


^ Combines many tools such as:
     * chef/puppet
     * fabric for orchestration
     * terraform for cloud provisioning

---
![](https://c1.staticflickr.com/3/2556/4058003731_44acc41419_b.jpg)

# [fit] Salt Architecture

^ https://www.flickr.com/photos/mrtopf/4058003731

----

![original fit](SaltStackArch.png)

---

# [fit] Salt Internals

![right fit](SaltStackInternals.png)

---

# [fit] Let's Create Stuff

^ Create a VPC setup by hand
  Show it using the CLI
  We can do better

---

![](VPC.png)

# Virtual Private Cloud

* Subnets (Public and Private)
* NAT and Internet Gateways
* Peering
* Routing


---

![Formulas](Formulas.jpeg)

# Salt Formulas

* Reuseable
* Testable
* Configurable

## Example

<https://github.com/saltstack-formulas/aws-formula>

^ Look at the example `aws-formula`

---

![right fit](Pillars.jpeg)

# [fit] Salt Pillar

^ States are our templates
  Pillar is our data
  It is host specific

---

![right filtered](https://c2.staticflickr.com/8/7515/15990021087_84b8c7f373_b.jpg)

```yaml
aws:
  region:
    us-west-1:
      profile:
        region: us-west-1
        keyid: [insert your keyid]
        key: [insert your key]
      vpc:
        {%- set vpc_name = 'demo-blog-vpc' %}
        {{ vpc_name }}:
          cidr_prefix: '172.20'
          vpc:
            name: {{ vpc_name }}
            cidr_block: 172.20.0.0/16
            instance_tenancy: default
            dns_support: 'true'
            dns_hostnames: 'true'
          internet_gateway:
            name: internet_gateway
          subnets:
            1:
              name: public_subnet
              az: b
              nat_gateway: true
```

^ https://www.flickr.com/photos/8025254@N07/15990021087

---
# Targeting Minions with Pillar

## `top.sls`

```yaml
base:
  '*':
    - aws
```

^ Target with globs, grains, groups etc.

---

# [fit] :raised_hands:
# Putting it together


---

# Questions?

## [@calvinhp](https://twitter.com/calvinhp)
## [@IndyAWS](https://twitter.com/indyaws)

### [fit] <http://www.sixfeetup.com/blog/build-aws-vpc-with-saltstack>
