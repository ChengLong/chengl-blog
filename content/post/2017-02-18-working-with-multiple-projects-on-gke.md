---
date: 2017-02-18T09:27:00.000Z
lastmod: 2017-07-30T16:00:13.000Z
title: Keeping configurations sane for multiple projects on Google Container Engine
draft: false
slug: working-with-multiple-projects-on-gke
tags: ["Kubernetes","GKE","Docker"]
image: /content/images/2017/07/gke.png
description: 
---

In my [previous post](https://chengl.com/kubectl-authentication-made-simple/), I present the easist and most secure way to get `kubectl` working for one project. But what about mutiple projects? Juggle mutiple projects on Google Container Engine (GKE) can be hard, especially when its configurations are admittedly [quirky](https://github.com/kubernetes/kubernetes/issues/20605). This post describes the best practice, in my opinion, to keep configurations sane and easy to switch.

## Problem

Suppose you have an awesome app that runs on GKE. You probably want to have two different environments `staging` and `production`, and the environments should be completely isolated. So you create two projects on GKE, `awesome-app-staging` and `awesome-app-production`, and provisioned resources for each. Now the question is how to effectively switch between the two projects on command line without repeating [these commands](https://github.com/kubernetes/kubernetes/issues/20605#issuecomment-218322105) over and over again.

## Solution

Assuming `gcloud` and `kubectl` are installed, but not configured,

#### 1. Create a configuration for each project

Create an empty configuration. Don't use `default`

    gcloud config configurations create awesome-app-staging
    

[Activate service account](https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account)

    gcloud auth activate-service-account --key-file /path/to/your/key.json
    

Set project

    gcloud config set project awesome-app-staging
    

It's good to set `DEFAULT_ZONE` and `DEFAULT_REGION` too.

    gcloud config set compute/region ${REGION}
    gcloud config set compute/zone ${ZONE}
    

Verify that your newly-created configuration has correct values

    gcloud config configurations describe awesome-app-staging
    

`gcloud` is ready.

Get `kubectl` ready by getting GKE credentials for the project

    gcloud container clusters get-credentials ${CLUSTER} --zone ${ZONE} --project awesome-app-staging  
    

This will insert auth data and project info in `~/.kube/config`. Verify your context is correct

    kubectl config current-context
    

It should return a string which consists of project, zone and cluster.

Repeat the above process for each project.

#### 2. Switch projects

Once configurations are created for all projects, switching is easy.

List all contexts

    kubectl config get-contexts
    

Switch to a context

    kubectl config use-context ${CONTEXT}
    

See current context

    kubectl config current-context
    

Please note that switching context in `kubectl` does *NOT* automatically switch the corresponding `gcloud` configuration. This means that unless you instruct `gcloud` and `kubectl` to work on the same project, they can work on completely differnt projects. Therefore, as a good practice, remember to switch `gcloud` configuration whenever you switch `kubectl` context and vice versa, unless you know what you're doing.

Activate configuration

    gcloud config configurations activate ${CONFIGURATION}
    

## Summary

This is a very simple and elegant solution to manage multiple projects on GKE. If you have better ideas, please let me know.

Happy switching!
