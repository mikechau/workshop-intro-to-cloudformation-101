# Intro to CloudFormation 101

This document outlines a workshop for introducing CloudFormation.

Are participants expected to be a CloudFormation expert at the end of the
workshop?

Absolutely not.

The goal of this workshop is to expose what CloudFormation is and does.

It is totally OK if any or all of these things are beyond you.

After the workshop, perhaps through reflection it will start to make sense.
Alternatively, down the road you might become more exposed to the terms and concepts described in this workshop and it might start to make more sense at
that point.

Just keep an open mind and everything will be 200 OK.

## Overview

The following topics will be covered:

- Brief History
    + The Manual Way
    + Configuration Management
    + Infrastructure as Code
- CloudFormation Templating Languages
    + JSON
    + YAML
    + Alternatives
- CloudFormation Resources
    + Security Groups
    + Launch Configuration
    + Auto Scaling Group
    + EC2 Instance
- CloudFormation Features
    + Parameters
    + Resource Types
- Lab

## Brief History

Before we jump into writing a CloudFormation template, let's review a brief
history of managing AWS infrastructure before CloudFormation.

### The Manual Way

[!Automation - xkcd](https://imgs.xkcd.com/comics/automation.png)

The fallback for automation is just doing whatever it is manually.

Without the right tooling, automating a process can be time consuming because a
lot of time can be spent on building tools to assist with the automation (ie. writing a script).

Examples:
    - Logging into the AWS console and manually provisioning servers.
    - SSH into each server to apply system upgrades.

We could save some time by writing some scripts and it'll get the job done and this will generally work fine at a small scale.

The manual way becomes tedious and error prone when we start to introduce the
need to manage more systems in many different environments and need to create
copies of an environment that are all similarly configured (ie. dev/staging/
prod).

Tracking changes to a system can also be challenging as there might not be any
sort of audit trail to refer back to.

Pros:
    - Fast

Cons:
    - Tedious when repeating
    - Error prone when repeating
    - Hard to determine the state of the world

### Configuration Management

As we move away from the manual way, we might start to look at configuration
management tools.

Popular choices include:

- Chef
- Puppet
- Salt
- Ansible

Configuration management is all about maintaining consistency and tracking
changes. Configuration management also makes it easier to audit and maintain
compliance.

Essentially, configuration management allows you to declare the state of the world and it will update the targeted systems to match it.

Pros:
    - Consistency
    - Auditing
    - Compliance

Cons:
    - Not necessarily needed for containerized application deployments
    - Rolling back is not free

### Infrastructure as Code

Infrastructure as code is all about having a single source of truth that serves
as a blueprint for what we want the infrastructure to look like.

Templates are written using a DSL that describes the desired resources and their relationships.

Examples:
    - CloudFormation
    - Terraform

Pros:
    - Consistency
    - Auditing
    - Compliance
    - Rollbacks*

Cons:
    - Potentially can destroy a resource unintentionally

## CloudFormation Templating Languages

CloudFormation supports two flavors of templating JSON and YAML.

### JSON

```json
{
   "AWSTemplateFormatVersion" : "2010-09-09",
   "Description" : "Single EC2 Instance",
   "Resources" : {
      "MyEC2Instance" : {
         "Type" : "AWS::EC2::Instance",
         "Properties" : {
            "ImageId" : "ami-79fd7eee",
            "KeyName" : "testkey",
            "InstanceType": "t2.micro"
         }
      }
   }
}
```

At the bare minimal, the JSON template contains top level keys for
`AWSTemplateFormatVersion`, `Description`, and `Resources`.

Inside `Resources` are uniquely named keys that are mapped to specific AWS
resources.

### YAML

```yaml
---
AWSTemplateFormatVersion: '2010-09-09'
Description: Single EC2 Instance
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-79fd7eee
      KeyName: testkey
      InstanceType: t2.micro
```

CloudFormation has recently offered support for YAML offering a more human
option for writing templates. Whitespace has significance with YAML, rather
than use curly braces we can nest keys by indenting two spaces at each level.

### Alternatives

Generally, most CloudFormation users will probably not be writing their
templates in JSON by hand, but utilize some sort of scripting language to
generate the templates to JSON.

Such options include:

- [cfndsl](https://github.com/cfndsl/cfndsl) - Ruby
- [troposphere](https://github.com/cloudtools/troposphere) - Python

But ultimately, we can use whatever we want as long it generates valid JSON or
YAML.

## CloudFormation Resources

A CloudFormation template represents a *stack*, and the components within the
*stack* are known as *resources*.

In this workshop, as a class we will compose a static template. It will cover
everything from the VPC to an Auto Scaling Group.

If you are lost, it is OK. This part of the workshop is more about doing.

### Security Groups

> A security group acts as a virtual firewall that controls the traffic for one or more instances. When you launch an instance, you associate one or more security groups with the instance. You add rules to each security group that allow traffic to or from its associated instances. You can modify the rules for a security group at any time; the new rules are automatically applied to all instances that are associated with the security group. When we decide whether to allow traffic to reach an instance, we evaluate all the rules from all the security groups that are associated with the instance.

**KEY FACT:**

> Security groups are stateful — if you send a request from your instance, the response traffic for that request is allowed to flow in regardless of inbound security group rules. Responses to allowed inbound traffic are allowed to flow out, regardless of outbound rules.

For example, if you have a EC2 instance that needs to talk to MySQL, the
instance only needs to have a egress (outbound) rule to 3306. The MySQL server
would need a ingress (inbound) rule for 3306.

#### References

- [Security Groups for Your VPC](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html)
- [AWS::EC2::SecurityGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html)

### Launch Configuration

> A launch configuration is a template that an Auto Scaling group uses to launch EC2 instances. When you create a launch configuration, you specify information for the instances such as the ID of the Amazon Machine Image (AMI), the instance type, a key pair, one or more security groups, and a block device mapping. If you've launched an EC2 instance before, you specified the same information in order to launch the instance.

**KEY FACT**:

> You can specify your launch configuration with multiple Auto Scaling groups.

> However, you can only specify one launch configuration for an Auto Scaling group at a time, and you can't modify a launch configuration after you've created it.

A launch configuration is immutable and can be assigned to multiple auto scaling groups, but a auto scaling group can only use one launch configuration.

#### References

- [Launch Configurations](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchConfiguration.html)

### Auto Scaling Group

> An Auto Scaling group contains a collection of EC2 instances that share similar characteristics and are treated as a logical grouping for the purposes of instance scaling and management. For example, if a single application operates across multiple instances, you might want to increase the number of instances in that group to improve the performance of the application, or decrease the number of instances to reduce costs when demand is low. You can use the Auto Scaling group to scale the number of instances automatically based on criteria that you specify, or maintain a fixed number of instances even if a instance becomes unhealthy. This automatic scaling and maintaining the number of instances in an Auto Scaling group is the core functionality of the Amazon EC2 Auto Scaling service.

#### References

- [Auto Scaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)

### EC2 Instance

> Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing capacity in the Amazon Web Services (AWS) cloud. Using Amazon EC2 eliminates your need to invest in hardware up front, so you can develop and deploy applications faster. You can use Amazon EC2 to launch as many or as few virtual servers as you need, configure security and networking, and manage storage. Amazon EC2 enables you to scale up or down to handle changes in requirements or spikes in popularity, reducing your need to forecast traffic.

#### References

- [What is Amazon EC2?](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

## CloudFormation Features

To keep things simple, we will only cover a few features CloudFormation
templates supports.

#### References

- [What is AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)

### Parameters

> Parameters enable you to input custom values to your template each time you create or update a stack.

- [Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)

### Resource Types



### Reference Function

## Lab

Using our existing CloudFormation template, configure it to accept a parameter to change the instance type, AMI (typed), and add another security group rule that allows incoming traffic on port 443.
