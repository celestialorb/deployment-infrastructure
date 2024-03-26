---
theme: purplin
title: Automated Deployments
info: An overview of automated deployments and infrastructure solutions.
class: text-center
highlighter: shiki
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# Automated Deployments and Infrastructure
### An Overview

---
layout: full
---

## Infrastructure as Code
### Declare What you Want

Declare desired infrastructure and let your chosen solution manage and track it.

<div class="grid grid-cols-2 gap-4">
<div>

```terraform
# Declare an example AWS S3 bucket.
resource "aws_s3_bucket" "example" {
  bucket_prefix = "my-tf-test-bucket"

  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}
```

Terraform/OpenTofu will display a plan of changes, allowing you to review the
actions.

</div>
<div>

```
$ tofu plan

OpenTofu will perform the following actions:

  # aws_s3_bucket.example will be created
  + resource "aws_s3_bucket" "example" {
      ...
      + bucket                      = "my-tf-test-bucket"
      ...
      + tags                        = {
          + "Environment" = "Dev"
          + "Name"        = "My bucket"
        }
      + tags_all                    = {
          + "Environment" = "Dev"
          + "Name"        = "My bucket"
        }
      ...
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

</div>
</div>

---
layout: full
---

## Infrastructure as Code
### Declare What you Want

Declare desired infrastructure and let your chosen solution manage and track it.

<div class="grid grid-cols-2 gap-4">
<div>

```terraform {6}
# Declare an example AWS S3 bucket.
resource "aws_s3_bucket" "example" {
  bucket_prefix = "my-tf-test-bucket"

  tags = {
    Name        = "example-bucket"
    Environment = "Dev"
  }
}
```

Terraform/OpenTofu will display a plan of changes, allowing you to review the
actions.

</div>
<div>

```
$ tofu plan

OpenTofu will perform the following actions:

  # aws_s3_bucket.example will be updated in-place
  ~ resource "aws_s3_bucket" "example" {
      ~ tags                        = {
            "Environment" = "Dev"
          ~ "Name"        = "My bucket" -> "example-bucket"
        }
      ~ tags_all                    = {
          ~ "Name"        = "My bucket" -> "example-bucket"
            # (1 unchanged element hidden)
        }
        # (10 unchanged attributes hidden)

        # (3 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.
```

</div>
</div>

---
layout: full
---

## Automated Deployments with Terraform
### GitLab Solution

A [Terramate CI](https://192.168.7.51:30000/infrastructure/terramate-ci) project
has been created, which can be imported into other projects.

It relies on the following directory structure for a project:

```
project-root/
└── terraform/            # Terraform Definitions
    ├── environments/     # Static Environment Definitions
    |   ├── development/  # Development Static Environment (EXAMPLE)
    |   └── production/   # Production Static Environment (EXAMPLE)
    └── modules/          # Terraform Module Definitions
        └── service/      # Service infrastructure definition (EXAMPLE)
```

The `environments` directory declares static environments.

The `modules` directory declares Terraform modules to be imported by the
environments (typically just one for a project).

---
layout: full
---

## GitLab Integration
### Tracking Deployments

The [Terramate CI](https://192.168.7.51:30000/infrastructure/terramate-ci)
project also supplies GitLab integration for tracking deployments and
environments.

![hello](/images/environments.png)

---
layout: full
---

## Deployment Targets
### To what do we deploy to?

Options range by security, cost, availability, and learning curve.

<div class="grid grid-cols-4 gap-4">
<div>

### <mdi-server /> OnPrem Machine

<br />

- Simplest
- Typically the least secure
- Least availability
- Typically the cheapest option

</div>

<div>

### <logos-aws-ec2 /> AWS EC2 Machine

<br />

- Simple
- Less secure
- Lower availability
- Easier to manage
- Pricing can vary

</div>

<div>

### <logos-aws-ecs /> AWS ECS Cluster

<br />

- Simple (for a distributed system)
- More secure
- Higher availability
- Higher cost

</div>

<div>

### <mdi-kubernetes /> Kubernetes Cluster

<br />

- More Complex
- Most Secure
- Highest Availability
- Cost is variable
- Can be OnPrem

</div>
</div>

---
layout: quote
---

The difficult part of automated deployments is access control.

---
layout: full
---

## Access Control
### How do we grant access to our deployment mechanism?

This will depend on the selected deployment target.

<div class="grid grid-cols-2 gap-4">
<div>

For example, if we have a single on-prem machine we may need to make a user
with an SSH key and grant the CI job access to that key.

If we're deploying to AWS we may need to set the CI variables with an AWS IAM
user access key and secret key, or fetch/generate credentials from an external
secrets storage system for the job.

</div>
<div>

In GitLab access can be granted in two main ways.

(1) For static access control, the secrets can be set as CI variables for the
    group/project. This, of course, has the downside of allowing certain
    developers to view the secret and that there is no identity attached to the
    secret itself.

(2) For dynamic access control, each job comes [with ID tokens](https://docs.gitlab.com/ee/ci/secrets/id_token_authentication.html#automatic-id-token-authentication-with-hashicorp-vault).
    These can be used to authenticate against an external secrets storage system
    to fetch secrets (static or dynamic) for the job.

</div>
</div>

---
layout: full
---

## Ingress Control
### How do clents reach our service?

This highly depends on the chosen deployment target.

<div class="grid grid-cols-2 gap-4">
<div>

### DNS

It's more professional to use a hostname for your service over an IP address,
so we'll need to create a DNS record to resolve it.

This can also help with availability as the DNS record could be automatically
updated to point to a new endpoint.

The hostname will need to either be publicly resolvable or the client will need
to add your nameservers to their DNS resolution setup.

</div>
<div>

### Network

Your service will, of course, need to be network-accessible from your clients'
networks. For a single machine this will likely mean that the machine itself
will need to be made accessible to your clients.

For a distributed system you may need to setup a load balancer or proxy layer
that is accessible to your clients that would then proxy requests to your
services.

</div>
</div>
