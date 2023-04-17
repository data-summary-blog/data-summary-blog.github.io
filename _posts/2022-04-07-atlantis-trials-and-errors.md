---
layout: post
title: "Atlantis Trials and Errors (ENG)"
categories: atlantis
author: "Yongchan Hong"
---

# Atlantis Trials and Errors

During the deployment of Atlantis, I have gone through multiple trials-and-errors.  
In this post, I would like to summarize several cases that I have encountered.  

## 1. Inside Terraform Repo
- You should add `role_arn`  that you are planning to assume (Not IRSA, but PowerUser) inside a provider. You should also include a `session_name` if you are planning to trace API calls.
```
provider "aws" {
  assume_role {
    role_arn     = "arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME"
    session_name = "${var.atlantis_user}-${var.atlantis_repo_owner}-${var.atlantis_repo_name}-${var.atlantis_pull_num}"
  }
}
```
- You should include `role_arn` option into backend if you are planning to use assume_role with S3 backend.
```
terraform {
  backend "s3" {
    bucket   = "example-bucket"
    key      = "path/to/tfstate"
    region   = "ap-northeast-2"
    role_arn = "arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME"
  }
}
```
- If these are configured, you do not need to add any info at `aws` part of Atlantis helm chart.


## 2. Terraform Related Errors
- You can use `depends_on` to prevent a resource to be created before resources inside `depends_on` are created.
- Even if you are not authorized, `terraform validate` can validate the configuration.
- You can create `modules` for terraform, which will be used as containers for multiple resources. This will be illustrated further in a separate post.
- You can add `validation` of variables. You should include `condition` and `error_message` inside validation. Error message should start with an uppercase letter and end with a period. 
```
validation {
    condition = contains(["A", "B", "C"], var.example)
    error_message = "Example must be either A, B or C."
}
```


### Reference
https://www.runatlantis.io/docs/provider-credentials.html#multiple-aws-accounts  
