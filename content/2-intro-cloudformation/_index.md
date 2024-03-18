---
title : "CloudFormation Introduction"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

### CloudFormation Introduction

CloudFormation is a service that allows you to design and install AWS resources using code drafts. Some advantages of using this service are:

- Simplify infrastructure management

In the traditional method, you will create and manage each individual service on the AWS Console. This makes creating resources individually, independently, and requires you to be clear about the order in which resources need to be created first and last. Linking between applications also becomes complicated and prone to errors due to uncertainty.
Using CloudFormation, you can configure resources in drafts, giving you a better overall view of the system, and for required dependencies between resources, CloudFormation will ensure that in the exact order you want.

- Replicate the system quickly

Some examples when you need to replicate the system are you need to ensure the system is highly available and you need to deploy the system in another Region, creating a UAT environment for testing new features. Once you have a draft for an environment, you can reuse it to create a similar environment quickly with just a few commands with CloudFormation.

- Easily control and track changes in the system

In some cases, you need to upgrade the system or change some configuration parameters. You just need to change the configuration in typescript. CloudFormation has the ability to compare changes in your current infrastructure and the changes you just updated. Typically you can use source code management tools to version your infrastructure.

During the process of changing the infrastructure, CloudFormation also significantly supports you in identifying errors and rolling back the infrastructure automatically, ensuring minimal downtime for the system.
  
#### CloudFormation concepts

***Template***

These are scripts in JSON or YAML format that identify the resources that make up your system. An example of a script that helps you configure and create a simple EC2 instance

```yml
AWSTemplateFormatVersion: 2010-09-09
Description: A sample template
Resources:
  MyEC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      ImageId: ami-0ff8a91507f77f867
      InstanceType: t2.micro
      KeyName: testkey
      BlockDeviceMappings:
        - DeviceName: /dev/sdm
          Ebs:
            VolumeType: io1
            Iops: 200
            DeleteOnTermination: false
            VolumeSize: 20
```

***Stacks***

Stack is used to group related resources to make it easier for you to manage. All resources within the Stack are identified by *templates*. For example, you can create a Stack with templates including 1 ECS cluster and 1 Application Load Balancer.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation Project with ECS service 
Resources:
  ecsStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub https://${BucketName}.s3.${AWS::Region}.amazonaws.com/stacks/ecs.yml
      Parameters:
        ECSName: ECSName
  ecsALBStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub https://${BucketName}.s3.${AWS::Region}.amazonaws.com/stacks/alb.yml
      Parameters:
        ALBName: ALB
```

***Change sets***

When you update your infrastructure during application development, you can view the changes that will be implemented in the system using CloudFormation *change sets*. This feature allows you to consider and assess the level of impact on the current system and consider before implementing infrastructure changes.

***Export Stack value***

The resource sharing feature for other Stacks is limited to the Region, meaning that Stacks in the same Region can create links with shared resources. To use this feature, you need to create Exports in the Outputs section of the CloudFormation template. The following example shares VPCId information with the name `${AWS::StackName}-VpcId` where `AWS::StackName` is a pseudo parameter predefined by AWS to indicate the name of the Stack to be created.

```yml
Outputs:
  VPC:
    Description: A reference to the created VPC
    Value: !Ref VPC
    Export:
      Name: !Sub ${AWS::StackName}-VpcId
```
