---
layout: post
title: "Running Terraform in Docker with Gitlab"
date: 2020-04-14 21:46:07 +0800
categories: automation
---

We have a use case where terraform code is applied to 100's of target accounts which grows dynamically, so creating and maintainging hundreds of ci jobs is just not feaseable. so instead I set out to automate this entire process using one CI job which would run terraform on all target directories.
While I was thinking about this I also decided to use docker-runner for this.

But it was not going to be simple, terraform would not initlize inside the container because it could not access terraform module which is another repo on on our hosted gitlab. this required now to inject ssh-key into the docker environemt, at first I played with the idea of intermediate docker images and SSH key as argument but this turned out to be unnecasrily complicated. The final solution is much simple, thanks to the gitlab-ci

following is the Dockerfile I came up with

{% highlight docker %}
FROM alpine:3.7
WORKDIR /app
COPY . .
ENV TERRAFORM_VERSION 0.12.24

RUN apk --update --no-cache add libc6-compat git openssh-client curl

RUN apk add --update python3 \
 && pip3 install --upgrade pip \
 && pip3 install boto3 requests PyYAML pg8000 -U \
 && ln -sv /usr/bin/python3 /usr/bin/python

RUN cd /usr/local/bin && \
 curl https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip -o terraform*\${TERRAFORM_VERSION}\_linux_amd64.zip && \
 unzip terraform*${TERRAFORM_VERSION}_linux_amd64.zip && \
    rm terraform_${TERRAFORM_VERSION}\_linux_amd64.zip

{% endhighlight %}

basically this has everything we need to initlize terraform directory infrastrure and run terraform which is all now done in gitlab-ci which looks like following:

{% highlight gitlab-ci %}
stages:

- build

hardening:
stage: build
image: imagename built from the Dockerfile
script: - eval $(ssh-agent -s)
    - echo "$SSH_PVT_KEY" | tr -d '\r' | ssh-add - > /dev/null - mkdir -p ~/.ssh - chmod 700 ~/.ssh - ssh-keyscan host hostedgitlab.com >> ~/.ssh/known_hosts - chmod 644 ~/.ssh/known_hosts - ...

{% endhighlight %}
