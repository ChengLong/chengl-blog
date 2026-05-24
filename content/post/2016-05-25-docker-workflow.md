---
date: 2016-05-25T09:07:00.000Z
lastmod: 2017-07-30T16:07:37.000Z
title: Docker Workflow
draft: false
slug: docker-workflow
tags: ["Docker","Workflow","Continuous Integration","Continuous Delivery"]

description: 
---

In my previous posts, I demoed how to orchestrate Docker with [Swarm](https://chengl.com/orchestrating-docker-using-swarm/) and [Kubernetes](https://chengl.com/orchestrating-docker-with-kubernetes/). They all assume the Docker image [chenglong/simple-node](https://hub.docker.com/r/chenglong/simple-node/) is already there and ready to be deployed. But how to develop that image in the first place? How to streamline and automate the process of developing it on local machine, building and testing it on Continuous Integration (CI) server, and finally deploying to production?

This post introduces one possible Docker workflow.

I recommend reading [my previous post](https://chengl.com/orchestrating-docker-with-kubernetes/) before this to understand the app architecture and Kubernetes.

### Workflow

![Screen-Shot-2016-05-25-at-1-35-53-PM](/content/images/2017/07/Screen-Shot-2016-05-25-at-1-35-53-PM.png)

The flow is summarized as follows:

1. Code on local machine. Using Docker for local development is highly recommended. Find out more from my post [Using Docker for Rails Development](https://chengl.com/using-docker-for-rails-development/)
2. Push commits to Github
3. Travis is notified and builds new a new Docker image and runs tests against the newly-built image.
4. If all work well, Travis pushes the new image to Docker Hub.
5. Travis deploys the new image to Google Container Engine (GKE). Prerequisite: you need to setup a container cluster on GKE first and get the app up and running. Find out more from my post [Orchestrating Docker with Kubernetes](https://chengl.com/orchestrating-docker-with-kubernetes/).
6. Kubernetes starts [rolling update](http://kubernetes.io/docs/user-guide/rolling-updates/).

This flow works pretty well except that some hacks are necessary for Travis to deploy to GKE. I will explain the hacks and why they are necessary in details in the subsequent section.

Please note that the tools used in this flow can be easily replaced with their respective equivalents. For example, Github can be replaced with Gitlab, Bitbucket or whatever you like. Travis can be replaced by [CircleCI](https://circleci.com/), [Shippable](https://app.shippable.com/), or any CI that **supports Docker**. Docker Hub can be replaced by your private registry. GKE can be replaced by [Amazon EC2 Container Service](https://aws.amazon.com/ecs/) or any [Containers as a Service (CaaS)](https://blog.docker.com/2016/02/containers-as-a-service-caas/).

All source code can be found in [my repo](https://github.com/ChengLong/docker-nodejs-redis).

### The devil is in the detail

    sudo: required
    language: node_js
    node_js:
      - '5'
    
    services:
      - docker
    
    env:
      global:
        - secure: AoSvVfpX77AtMBpXKyHH67wTKWCN9xdXoMWroU2TwewBRToebQmLUesT+6gP2rCVoQ282IpESkZkIUumj38rvCGWpL3fLU67Fb1Fa/SqKQ++OYri3NNmoOLltkHMCHHz2bl7B8/72KJQ8e+sl8KCmKBTaLp1g2/36+DZwL9KZjFOqsQg2pwv3zjOUkZte6v6Igsl7lV1pbLkg9Fq1KEvOZl8D6bHgtNuHJGYZZrHCUDHMXxXeQ8/wL0v7GpUkZAe85Ve+fkPoX7AXYumbi5SsxbwVYG64j4zdU2ydlhmMRfOrT4BIbOHUO1kmP5uSf4/xX+fGJZuGDlVT1P5RTZyvu1dA+X10a0t2pOYHAjTjJp6CNRIKuJ4izBvxCZaLAUd42FfU9BAFyUPlViaDzNZAW3DCoapfnS0xM+UY1U8Z7bs+lIXycqIE6OR4KZXfKtpoKakmh8d0eY8LpUNu81VS9Z2mxfRg0xsYhp/1E/X+LZM/YCzPpOhQCc0ZGJEtbqj1/Qebp/GRBIx9oabBH+JxldqrZApIF0MsGiaIT8Q0LJ8wNVVc7QQqhjwdtvP1Wh2pxOvnzEyAdB4AeKrYv0SMwa3OxpIDtJEP1AvXRTZWXPObSSgAhK2VPnOK9Y++N2BlSKJh9k1nlnzn0FQsHlPUV/r9Ui62tmQbzOyL69xltQ=
        - secure: KLweub+lnStvWmKY/XoDy8nXo4v5lYTHapXtewfDFrnWPL6UQV//kklUAswzGEWp4id9pOolZB4RNfucCBzB1Kb1KgSp23wz9mj+HWY+qUQRsLyIxYwsZf/o9H/MMkesG8jOF/vU35Ne3e7MdSmpNuqEqErIqc25691rBMLSDxi5YS3jcB+vGkv38xtlcDilnUrSneKuNEzuI5Q7MKr6vqIVaW3vbwEs5hEh2oPSAgVno67s9NyfUi2Dl4sIDDwzV+iXc9FNk4Ww6Vkm8uzZvIXhZDuyVnFWSJhgfNgElUna7XfqnWOyPdZQ0rh2iOtKzO9uZQiOoNq8JuT9VstFcFlhsfAtlynB5Xgp/EH9NUIZvUiueRhs0bOH9SKRUijIeTQ/OimuX0y8EZdz7TKSigZkc7iVqD0g2E2kLtv3avTCodwts54V8bX7//r8Y2FE7DFkZRGL7Khf5LPjRj7xVEb9OhEkuSYfs0oEhqyXCSMUNyov3BDFbZ/AWh0smQmKYv36U2JvMfPghNlgAE+b/Xc4F/sJMmNSDdLBmGwoJDJtVA5iB3RCfkZfEeqhMoj2C8uu8s8IASzxh2HnaaO5IQKy7kGwAzzRfazTq6JhvLPfteOKStvRYrLLBll7l2DPazLJK8ctKZ6CTFxoycU+R73IliwwpqLCAhasD9N7Q8w=
        - secure: htu5A6fjnQvJTvinumWI1u1unoDMgY5FleGM8JwVLBQl6Srr1GU/2ulii1LSbypG/JimwMzT6BIRMfNgCzAeBc6aWM8xbZUsLl7p+WFyYNxOW26mVm8Z5R+2JZXm3vZoevjHD35gWIBMReqVySIRLXanZ9SOVRC1IcX2Om1uOoQHAqwrFh4KERWnzepAXylfUtFatqmRmCRczH4m9MKF7OgbZD/7xKHHoJxXLHjmWaCQrLmucb01/e/vl3C/LwOvc+A6fwXhhMFdL5UUn9iVWLBhTYknULjOKDF2AgvahvStxGUEAcscaW4qATW32Ylamn3K4l5X+3ic+fLKQHOR+oCaVmaqcpExbxuBfLHGQkV7DiUePQsS1D5NLfAHBdbd8MJCwxQRlSIt/z/aemkx6uWEBu053c47yuuKltFhAm6B7Z52UkHC8fpFfuy03xgorjNTayghPlYDeFZRzItkibTEY1EH4Sh9yVkAIhv5hqomMQMAkxaEFgmBkK7/8+UHB+sMlBupMiejTqIgZOzIhj1uqDGkokqI2UqlNx1wuqzGJ/KnQWzsgFsnoU0Q1zIzw0i1RAiDu8qj5vSnioqw8pvZWuBDuRltE5U4fup529ZtFSNx4Ceb3ECathc6+3Em8OvtqvBOr44xNBW8XCFbi82Ys3Dd/SEHGwlcwYoT4/A=
        - secure: W1CzoMk6zUyhKhx+Q4mX/O1rh7GKFTrMNpkJ+QqN4WN56K3v49t6kotABXbFU4K8ZP84DhUkQYtm6DbEgCkxeNaT0k0gzKyG+CZo5sI6yqx15CwmR/qCkFruDQqUG/2hZNTP5ZP/7hMQwNCxATxuzu2GOcnwk1YuIJsJ5by9+RdWwKBYYhSG/BokNq6JaYxZx8NeMTa5W/CJXtu/5qFsMDQ07Syg4MgAnRHbRB9uCxpUOtJ44HDhZFOOY48qIDVpYhVpTg0XVzw5VkFNgOuczIYWw/SaWyOyoKgt8FtLhUPDsHtvueFqSJffYlyoh81Uxu+LNAtZoact5wFOwkDM+mzqRe/lzm2hcYY0nc4Dxc1ito43GedQmFnadZSozZfKAEmRV0m+1W6iE6fi+DpHXm3JCmgvolB9khLiYVTHkgaSzbfZDfdBmoXPEcaDuKnY6KTB410sGnadO6IT3Z5x8FtMQ/In/dB7XQr/G1aXve3KjUz8/oLZbS+IWEH8eQEOAR8V6w6HpvQIZE6ccwEzmLILlygne7TSAnOPQX1hiEDdUYYOtoyVEv/NUzd/OpGqNl6edYsXZB1bcY7FWy2YhmJD4mqe5h6mp4yRXYyuFbdC3OIBGpr1vnNaF9uUdayZ59pZ4kIojCQL2SKZKqOYdezyV/h0ohP93FIboi/nL7Q=
        - secure: jxY+/q6svho30zk1RMtnwHhHkYMw+NqQS4n7dOILpsI6zKwC4n2c+xjD0MoOYjVsd/SiJucXeeNR4BcS4+OglsVxja68rBWqV4F3xloGwtMZebqMhNGSV4kitgAkdDNktQid2fQAwUsoCtEfq3A6ijMdOgoehZwxXqFg5+5cRwjI58vVOfdFDGPUO6KXTDhwwXIgTi9zN8nXSNCgHUZfrIGw6jTdFvL2NGkejbX4AqutethTlXlA7lVeE/O1SUW4H8MRl59MyKUwWm/j7qkmM/TX+PUaXSwXgoR0Nzbt2DKTbY56qff1olHaUMuV5c8A5gYlThvVYbMnGRTK7JYOzmchyvRsqc1inxIwU7bi/J1DnQpCX4qF06NBU+OLbx8+qDc9E6l9XGdQ09HFVPOctlNrkfcPDgGVfRYwomU8+gb8MykgBxtUqLIpJN0pr9sNZN4L4jGuv58ThkCfoaH6YgiLKTITJ7cLAE2tNmHj5Kw+vHNvJcUltRdMOSaDI6TyAGkVpUSHaTEOvdvsj5y5wFS3TdIHnTQOWMpBiYSHY7D4trvv6KRagDjoB7DKaAohYPwataJdu3Wu+A0UIsukYPG6uVn4RPJmQHBMT8vTtf9q9eSpoRU68zSnhAL5aXtq1vqtfy4859QQDzpH7JMA87WtJqCQ4Fp8MHOg3uvb+ZM=
        - secure: FXEj1gwSi+xjpc4GFFS4ehxy6Q4wpmmPq/rlfWijOAYsNwukfkL2gCmnzjTT6ZZVHM47bNtHlTdqTM8bmBjWVeVEPKwvULq6i2neiGI7OI0vh2dsbCMIZ7bCMuRL0/WRq/A8YPsXIK76YIlMg8sSzdGdAmvZpjTbLywSl9KoSlH3S56e2Nz1kb8DhP4emeFcxdXul463JUdOBpCZa8oJLopOaX5+PypOk6XOjOyWkxN938F4w5/4MzhgghcrTRwtOQCPHyJR2zg08MWm5COKi3L2EmgPyctXGGr31YVeRTC9Gg3SJ+Mt8OfsX8f7TnUzhnfA3IszJpxw4cF1snpVmXZED2J1Aa6Gi7mSIhb1CdYNUxUKx6BVnaUGDRfmcxtGOt8Uy0A2sRl3FGhVVH6xQaaTmG3nzU0fm44cO1RwxqXfP4kXqUNj+LlCUC+3hIRrWRMfxjIWwGA0LZIqnVlou2TRLkxdM3IHx46cFqEfpD6Do/NXOCUZYRcE95FqvK6ZrobsZapkI0WkmoO4YdzCkuW9nSc8Y121OWlP9urkzaU4X7tO7FbLshKmjYNL4vLqyJ0gkN21yVJTIJWE6l/TGGi+RxwFF2jzN8oRjOJrWESmUSPa5WhNvuIOu1t0RObwRup0yjArRuUIXxuQt99c1tkfsn52fZWZih3t4bM0bxE=
        - CLOUDSDK_CORE_DISABLE_PROMPTS=1
        - COMMIT=${TRAVIS_COMMIT::8}
        - IMAGE=chenglong/simple-node
        - IMAGE_VERSION=$IMAGE:$COMMIT
        - DEPLOYMENT=frontend
        - CONTAINER_NAME=nodejs
    
    before_script:
      - docker build -t $IMAGE:$COMMIT app
      - docker tag $IMAGE:$COMMIT $IMAGE:latest
      - docker tag $IMAGE:$COMMIT $IMAGE:travis-$TRAVIS_BUILD_NUMBER
    
    script:
      - docker version
      - docker-compose version
      - docker-compose -f docker-compose.ci.yml run test
    
    after_success:
      - docker login -e="$DOCKER_EMAIL" -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
      - docker push $IMAGE
      - curl https://sdk.cloud.google.com | bash
      - source /home/travis/.bashrc
      - gcloud components update kubectl
      - kubectl config set-credentials default --username=${GKE_USERNAME} --password=${GKE_PASSWORD}
      - kubectl config set-cluster default --server=${GKE_SERVER} --insecure-skip-tls-verify=true
      - kubectl config set-context default --cluster=default --user=default
      - kubectl config use-context default
      - kubectl patch deployment $DEPLOYMENT -p '{"spec":{"template":{"spec":{"containers":[{"name":"'"$CONTAINER_NAME"'","image":"'"$IMAGE_VERSION"'"}]}}}}'
    

This is the `.travis.yml` that instructs Travis to

- build the Docker image `chenglong/simple-node` with 3 tags: `latest`, `${TRAVIS_COMMIT::8}` and `travis-$TRAVIS_BUILD_NUMBER`
- run tests against the newly-built image `chenglong/simple-node:latest`. Note that the tests are run in a container named `test`. The tests basicially verify that the app gives expected response. See test code [here](https://github.com/ChengLong/docker-nodejs-redis/blob/master/test/test.js).
- push the image to [Docker Hub](https://hub.docker.com/r/chenglong/simple-node/tags/) if tests pass
- install latest Google Cloud SDK because the one on Travis is [too old](https://github.com/travis-ci/travis-ci/issues/5530)
- install `kubectl`
- config `kubectl` to connect to the a pre-existing cluster on GKE
- rolling update deployment `frontend` with the newly-built image `chenglong/simple-node`

A few key points:

- `chenglong/simple-node` is *ONLY* built once. It's the same image that is tested against and deployed to GKE. See the logs for a build [here](https://travis-ci.org/ChengLong/docker-nodejs-redis/builds/132763728).
- Tests are run in a container named `test`, which simply starts `app` and `redis`, fires a few requests and verifies the response. To run the tests, I used `docker-compose -f docker-compose.ci.yml run test`. See [docker-compose.ci.yml](https://github.com/ChengLong/docker-nodejs-redis/blob/master/docker-compose.ci.yml)
- At the time of writing, Travis CI only has `docker-compose`[1.4.2](https://docs.travis-ci.com/user/docker/), which *does not* support compose file [version 2](https://docs.docker.com/compose/compose-file/#upgrading). This requires the compose file to be in version 1, unless you want to install `docker-compose` 1.6+.
- Travis doesn't support deploying to GKE yet, which requires some hacks to install the latest Google Cloud SDK and `kubectl`, so that I can do `kubectl patch deployment`.

### Conclusion

I hope this simple demo gives you a better idea of how Docker can help in your day to day work. A few obvious advantages of this Docker workflow:

- It significantly improves how we ship software. The artifact from CI is *NOT* anymore a tar ball, jar file, war file, nor ear file etc. It's simply a Docker image. A Docker image has not only the source code baked in, but also a complete running environment. Therefore, it removes the need to first provision server before deploying source code. Do you see Chef, Puppet or Ansible in the demo? In the age of containers, all you need is a server that runs Docker.
- Since it's the same image that is tested in CI gets deployed to production (or maybe staging), it's guaranteed that it works in exactly the same way.
- By containerizing the app, each component of the app is run as a container. And each container can be individually updated and deployed. Does this ring a bell? If not, read [Microservices](http://martinfowler.com/microservices/).
