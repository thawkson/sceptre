---
layout: docs
title: Architecting Sceptre
---

# Architecting Sceptre

Sceptre is written in a way that aims to be unopinionated in how it is used. It
is designed to work equally well with simple and complex infrastructures.
Sceptre's flexible nature, and the variation in how people organise their AWS
accounts, makes it difficult to give generic advice on how best to use it, as
it can be use-case specific. However, the following patterns have emerged from
our use of Sceptre at Cloudreach.

## Project layout

Sceptre's nested StackGroups means that it is possible to store an entire
company or department's infrastructure in a single Sceptre project. While this
is possible, it isn't recommended. Having a large number of developers interact
with a single repository can be difficult from a version control point of view,
and it might be dangerous to let developers touch infrastructure that they are
not directly involved with.

We recommend different Sceptre projects for each large 'section' of the
infrastructure being built.

## Directory layout

You need to store your Sceptre templates in the `templates` directory within
your Sceptre Project.

```
$ ls
config/
templates/
```

## StackGroup structure

StackGroups can be arbitrarily nested, and StackGroup commands can be applied
to any level of the StackGroup tree. This can make it difficult to know how to
divide StackGroups up.

When considering how to split StackGroups up, it's useful to remember the
following properties of StackGroups:

1. Region specific. There is no way for an StackGroup to launch stacks in
   multiple regions; a StackGroup can, however, contain sub-StackGroups in
   different regions. A StackGroup should therefore contain Stacks that belong
   in the same region.

2. Command related. StackGroup level commands (like `launch`) are applied to
   every stack in a StackGroup. There is no way to exclude stacks from an
   StackGroup level command. Therefore, StackGroups should contain stacks that
   can be launched and deleted together.

   Some stacks are inherently longer-lived than others. Stacks containing
   VPC-level infrastructure are likely to be longer lived than stacks
   containing an ephemeral testing environment. It therefore makes sense to
   split these up into separate StackGroups.

## Example architectures

The following examples demonstrate how we might architect various Sceptre
projects. Only configuration layout is shown, and the examples are merely meant
to demonstrate ways of organising different projects.

- Application development within an externally defined VPC

  ```
  - config
      - prod
          - application
              - asg.yaml
              - security-group.yaml
          - database
              - rds.yaml
              - security-group.yaml
  ```

- DevOps team who manage all the infrastructure for their service

  ```
  - config
      - prod
          - network
              - vpc.yaml
              - subnet.yaml
          - frontend
              - api-gateway.yaml
          - application
              - lambda-get-item.yaml
              - lambda-put-item.yaml
          - database
              - dynamodb.yaml
  ```

- Centralised, company-wide networking

  ```
  - config
      - prod
          - vpc.yaml
          - public-subnet.yaml
          - application-subnet.yaml
          - database-subnet.yaml
      - dev
          - vpc.yaml
          - public-subnet.yaml
          - application-subnet.yaml
          - database-subnet.yaml
  ```

- IAM management

  ```
  - config
      - account-1
          - iam-role-admin.yaml
          - iam-role-developer.yaml
      - account-2
          - iam-role-admin.yaml
          - iam-role-developer.yaml
  ```

- Per project configurations sharing the same templates. This is useful if you
  have common shared templates that are shared between many projects and would
  like to encourage template reuse. In this example, vpc.yaml and database.yaml
  are common templates, and each deployed environment should have its own
  configured VPC and database.

  ```
  - templates
      - application_1
          - application_1.yaml
      - application_2
          - application_2.yaml
      - shared
          - vpc.yaml
          - database.yaml
  - config
      - project_1
          - dev
              - vpc.yaml        
              - database.yaml      
              - application_1.yaml
          - qa
              - vpc.yaml        
              - database.yaml      
              - application_1.yaml
          - prod
              - vpc.yaml        
              - database.yaml      
              - application_1.yaml
      - project_2
          - dev
              - vpc.yaml        
              - database.yaml      
              - application_2.yaml
          - qa
              - vpc.yaml        
              - database.yaml      
              - application_2.yaml
          - prod
              - vpc.yaml        
              - database.yaml      
              - application_2.yaml              
  ```

To deploy project_1 to dev, you would run
`sceptre --config-dir config/project_1 launch dev`
