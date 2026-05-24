---
date: 2017-01-30T09:25:00.000Z
lastmod: 2017-07-30T09:27:11.000Z
title: kubectl Authentication Made Simple
draft: false
slug: kubectl-authentication-made-simple
tags: ["Kubernetes","kubectl","Continuous Delivery"]

description: 
---

While working on a continuous delivery pipeline to automate deployment to Google Container Engine (GKE), I found that getting `kubectl` to work is very [complex](https://kubernetes.io/docs/user-guide/sharing-clusters/#manually-generating-kubeconfig)[and](http://stackoverflow.com/questions/40426071/kubectl-access-to-google-cloud-container-engins-fails)[convoluted](http://stackoverflow.com/questions/40408321/whats-the-cli-authentication-process-as-of-google-container-engine-kubernetes-1), especially when it needs to be [noninteractive](https://circleci.com/docs/continuous-deployment-with-google-container-engine/). So I want to find out the easiest way to get `kubectl` working noninteractively.

Assuming that `gcloud` and `kubectl` are already installed but not necessarily setup, **ONLY** two commands are needed to get `kubectl` working noninteractively (verified with Google Cloud SDK 141.0.0 and kubectl 1.5.2)

    gcloud auth activate-service-account --key-file ${PATH_TO_KEY}
    
    gcloud container clusters get-credentials ${CLUSTER} --zone ${ZONE} --project ${PROJECT}
    

The first command [gcloud auth activate-service-account](https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account) is to authorize access to Google Cloud Platform using a [service account](https://cloud.google.com/compute/docs/access/service-accounts). `PATH_TO_KEY` is the path to the private key of the service account. The idea is very similar to IAM in AWS. One service account is roughly equivalent to an IAM group in AWS. And the private key of the service account is like Access key ID and Secret access key. You can create a service account and generate its private key [here](https://console.cloud.google.com/iam-admin/serviceaccounts). If you only need to deploy to GKE, `Container Engine Developer` is enough for the role.

The second command [gcloud container clusters get-credentials](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials) fetches cluster credentials and saves it in `~/.kube/config`. The environment variables in the command are self-explanatory.

You can now use `kubectl` to deploy to GKE. Probably this?

    kubectl set image deployment/${DEPLOYMENT} ${CONTAINER_NAME}=${IMAGE}:${IMAGE_VERSION}
    

In summary, this solution is simple, noninteractive (great for CI/CD) and secure (fine-grained permissions defined by service account). If you have a better solution, please let me know.
