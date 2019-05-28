autoscale: true
footer: Cloud Custodian — IndyAWS Feb 2017
theme: Letters from Sweden, 7

![](logo_capone_devex_cloud_custodian.png)

# [fit] Cloud Custodian 

## [fit] Real-time AWS Compliance

----

## [fit] Serverless BUT...

### Everyone still has Servers :computer:

----

![](9409659310_014ec5c4ca_o.jpg)

## A sea of policies [^1]

- Fleet wide **savings** policies
  - Off hours stops for dev environments
  - Garbage collect ebs, elb, etc
  - Detect over-provisioned resources
- Numerous **security** policies
  - Encrypt all the Things
  - Access Control
  - SSL ciphers
- Numerous **compliance** policies
  - Tag compliance / chargeback
  - Current images
  - Backups

[^1]: https://flic.kr/p/fkuXTm

----

![](5141207760_6f5343088d_o.jpg)

# [fit] Fleet Management

## [fit] Across Lots of federated accounts [^2]

[^2]: https://flic.kr/p/8Qj2X7

---

![right filtered 500%](2710464400_6e01c5d5da_o.jpg)

### But [^3]

- How are they implemented
- How are they deployed
- How are they configured
- How are they managed
- Who owns them 
- How are they Tested
- Are they Reviewed 

### Who Knows?

[^3]: https://flic.kr/p/58vQCQ

^ Natural tendancy to create one off scripts

----

## Cloud Custodian Basics

- Rules engine
- YAML DSL
- Integrated Lambda Provisioning

```yaml

- name: require-rds-encrypt-and-non-public
  resource: rds
  mode:
    - type: cloudtrail 
    - events: 
      - CreateDBInstance 
  filters:
    - or:
      - Encrypted: false 
      - PubliclyAvailable: true 
  actions: 
    - type: delete
    - skip-snapshot: true
```

^ A rules engine for infrastructure management.

^ YAML DSL for policies based on query resources or subscribe to events, apply filters, take actions. Integrated Lambda provisioning and event sources.

^ Outputs to Amazon S3, Amazon Cloud Watch Logs, Amazon Cloud Watch Metrics

----

# [fit]   Opensource
## [fit] :floppy_disk: https://github.com/capitalone/cloud-custodian

---

![](cw.png)

## Amazon Cloud Watch Events 

### Features 
- Powerful infrastructure observation capabilities 
- Enables “realtime” rules enforcement and reaction with wide coverage of AWS product APIs. 

### Sources
- All Cloud Trail Events (P99 @ 90s delivery window as of April 2016) 
- EC2 instance state changes (600ms) 
- ASG instance membership changes (600ms) 
- Periodic Scheduling (custom) 
- Custom events

----

![right filtered](303018443_a283f37eef_o.jpg)

## Cloud Custodian [^4]

- Resource type policies (ec2 instance, ami, auto scale group, bucket, elb, etc).
- Filter resources
- Invoke actions on filtered set
- Output resource json to s3, metrics to cloudwatch 
- Vocabularies of actions, and filters for policy construction.

[^4]: https://flic.kr/p/sM3PV

---

![](custodian_example_architecture.png)

----

# Custodian Policy

```yaml
- name: ebs-copy-instance-tags
  resource: ebs 
  filters:
  - type: value 
    key: "Attachments[0].Device"
    value: not-null
  actions: 
  - type: copy-instance-tags
    tags:
    - App 
    - Env 
    - Owner 
    - Name
```

----

![](2368302976_e4379a3298_o.jpg)

## Filtering resources [^5]

- Generic Value filter
- Arbitrary nesting of filters with ‘or’ and ‘and’ blocks.
- Simple key/value are equality matches with value expressions

```yaml

- type: value
  # Ignore keys that start with
  # 'aws:' as they don't count towards the limit.
  Key: "[length(Tags[?!starts_with(Key,'aws:')])][0]"
  op: less-than
  value: 10

- or:
  - "tag:App": absent
  - "tag:Env": absent
  - and:
    - Encrypted: false
```

^ Jmespath expressions on resource’s json representation 

^ Lots of operator matching (in, not-in, absent, not-null, gte, regex, etc)

[^5]: https://flic.kr/p/4BhaXW

----

![right filtered](4481677663_3a7f6c93ba_o.jpg)

## [fit] Multi Step Workflows [^6]


> Poorly tagged instances, should be stopped in 1 day, and then terminated in 3

- mark-for-op
- marked-for-op 

[^6]: https://flic.kr/p/7Q2LCx

----

![right filtered](9049938355_ec0274d3c0_o.jpg)

## [fit] Multi Step Workflows [^7]

#### Chain together multiple policies. 

```yaml
- name: ec2-tag-compliance-mark 
  resource: ec2
  description: |
    Find all non-compliant tag instances for stoppage in 1 days.
  mode: 
    type: periodic 
    schedule: rate(1 day)
  filters: 
    - "tag:maid_status": absent 
    - or:
      - "tag:App": absent 
      - "tag:Env": absent 
      - "tag:Owner": absent 
  actions:
    - type: mark-for-op
      op: stop
      days: 1 
```

```yaml
- name: ec2-tag-compliance-stop
  resource: ec2
  description: |
    Stop poorly tagged and schedule Terminate.
  mode:
    type: periodic
    schedule: rate(1 day)
  filters:
    - type: marked-for-op
      op: stop
    - or:
      - "tag:App": absent
      - "tag:Env": absent
      - "tag:Owner": absent
  actions:
    - stop
    - type: mark-for-op
      op: terminate
      days: 4
```

[^7]: https://flic.kr/p/eMHinV

----

## Custodian Vocabularies 

```sh
$ bin/custodian schema asg
asg:
  actions: [auto-tag-user, delete, invoke-lambda, mark, mark-for-op, notify, propagate-tags,
    remove-tag, rename-tag, resize, resume, suspend, tag, tag-trim, unmark, untag]
  filters: [and, capacity-delta, event, image-age, invalid, launch-config, marked-for-op,
    metrics, not-encrypted, offhour, onhour, or, security-group, subnet, tag-count,
    valid, value, vpc-id]
```

```sh
$ bin/custodian schema ec2
ec2:
  actions: [auto-tag-user, invoke-lambda, mark, mark-for-op, modify-security-groups,
    normalize-tag, notify, remove-tag, rename-tag, resize, snapshot, start, stop,
    tag, tag-trim, terminate, unmark, untag]
  filters: [and, default-vpc, ebs, ephemeral, event, health-event, image, image-age,
    instance-age, instance-uptime, marked-for-op, metrics, offhour, onhour, or, security-group,
    state-age, subnet, tag-count, value]
```

```sh
$ bin/custodian schema s3
s3:
  actions: [attach-encrypt, auto-tag-user, delete, delete-global-grants, encrypt-keys,
    encryption-policy, invoke-lambda, mark-for-op, no-op, notify, remove-statements,
    tag, toggle-logging, toggle-versioning, unmark]
  filters: [and, cross-account, event, global-grants, has-statement, is-log-target,
    marked-for-op, metrics, missing-policy-statement, missing-statement, or, value]
```

----

![](Screen Shot 2017-02-28 at 10.26.08.png)

## Additional Resource Types

- RDS
- ELB
- Redshift
- CloudFormation - AMI
- EBS
- EBS Snapshot

----

## Additional Resource Types

See all resources using the `schema` command.

```sh
$ bin/custodian schema
resources:
- account
- acm-certificate
- alarm
- ami
- app-elb
- app-elb-target-group
...
```

----

## Metrics/Reporting

- Resource Count
- Action Time
- Query/Filter Time
- Custom

![right fit](ec2_metrics.png)


----

## [fit] Example Policy

### [fit] Amazon S3 Encryption

----

### Require encryption for objects

```yaml
name: s3-require-encryption 
resource: s3
description: |
  Apply encryption required policy to new buckets
mode:
  type: cloudtrail
  events:
   - CreateBucket
  actions:
   - encryption-policy
   - encrypt-keys
```

---

### Find elb/s3 logs sinks and switch to lambda encrypt

```yaml
name: s3-remediate
resource: s3
description: |
  Encryption required policy
mode:
  type: periodic
  schedule: rate(1 day)
filters:
 - type: is-log-target
actions:
 - attach-encrypt
 - type: remove-statements
   statement_ids:
    - RequireEncryptedPutObject
```

----

## Roadmap

- Elastic search indexing of records / outputs (programmatic dashboards / historical trending)
- Cross Language support (lambda invoke actions)
- Moar filters/actions/resources

### [fit] <https://github.com/capitalone/cloud-custodian/milestones>

^ Currently there is no UI on Cloud Custodian

---

# [fit] Questions:grey_question:


### [@calvinhp](http://twitter.com/calvinhp)
### [calvin@sixfeetup.com](mailto:calvin@sixfeetup.com)
