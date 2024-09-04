---
title: AWS实践--跨账号访问ECR
date: 2022-09-02 11:12:52
slug: aws-cross-account-access-ecr-repo
tags:
  - ECR
  - AWS
categories:
  - AWS
---

## 需求

现有一个基础账号(111111111111)ECR存放所有的基础镜像和应用镜像，其他子账号(222222222222、333333333333、444444444444)的EKS集群需要拉取基础账号的ECR镜像。

## 配置

以下示例为AWS存储库跨账号访问策略.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "<statement-name>",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::222222222222:root",
          "arn:aws:iam::333333333333:root",
          "arn:aws:iam::444444444444:root"
        ]
      },
      "Action": [
        "ecr:BatchGetImage",
        "ecr:GetDownloadUrlForLayer"
      ]
    }
  ]
}
```

## 测试

``` bash
# 登录 222222222222、333333333333、444444444444 子账号的服务器
# 执行ECR登录命令
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 111111111111.dkr.ecr.us-east-1.amazonaws.com
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```