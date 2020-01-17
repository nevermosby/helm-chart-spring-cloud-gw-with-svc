# Helm chart for spring cloud oss with a simple service

## Overview

You can use this helm chart to deploy the whole services of [spring cloud oss](https://spring.io/projects/spring-cloud-netflix). It is planned to have the listed services below:

- [x] Zuul service as api gateway
- [x] Eureka service as registry
- [x] User service as bussiness service for demo
- [ ] Config service as config center
- [ ] Auth service as security center
- [ ] Hystrix service as circuit breaker
- [ ] Sleuth service for distributed tracing

Now the helm chart can provide the gateway service and a [user service](https://github.com/nevermosby/springcloudoss-user-service) as a demo service to validate the gateway function. You can hit the url below to view the result:

![gw-demo](https://raw.githubusercontent.com/nevermosby/helm-chart-spring-cloud-gw-with-svc/master/images/gw-demo.PNG)

## Features

### Using [AppHub](https://github.com/cloudnativeapp/charts/blob/master/README_en.md) and helm v3 for dependency management

As most of the spring cloud projects use `eureka` for service discovery, It is necessary to add `euraka` into dependency list.
Currently, I use Apphub from Alibaba as the helm hub, which is a mirror of helm hub hosted in China.

I have already baked [eureka cluster](https://github.com/cloudnativeapp/charts/tree/master/submitted/spring-cloud-eureka) as helm chart. Make sure you have these configurations below to use:

1. Add `apphub` as one of the local helm repo

```shell
helm repo add apphub https://apphub.aliyuncs.com/
```

2. Declare the depedency in `Chart.yaml` 

```yaml
dependencies:
  - name: spring-cloud-eureka
    version: 0.1.0
    repository: https://apphub.aliyuncs.com/
```

## Installation

### `helm add` & `helm install`
```shell
> helm repo add apphub https://apphub.aliyuncs.com/
> helm dependency list
NAME                    VERSION REPOSITORY                      STATUS
spring-cloud-eureka     0.1.0   https://apphub.aliyuncs.com/    missing

> helm dependency build
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "apphub" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading spring-cloud-eureka from repo https://apphub.aliyuncs.com/
Deleting outdated charts

> helm install gw-svc spring-cloud-gw-with-svc/

# You will have the output below as the command goes well:
NAME: gw-svc
LAST DEPLOYED: 2019-08-26 17:11:31.5412552 +0800 CST
NAMESPACE: default
STATUS: deployed
```

### View the deployed `kubernetes workload`
```shell
# check the pods
> kubectl get pod
NAME                           READY   STATUS    RESTARTS   AGE
eureka-0                       1/1     Running   0          43m
eureka-1                       1/1     Running   0          43m
eureka-2                       1/1     Running   0          43m
gateway-75c587d7dc-z6jw2       1/1     Running   0          43m
user-service-67567fd8b-bfcgg   1/1     Running   0          43m
# check the eureka cluster
> kubectl get sts
NAME     READY   AGE
eureka   3/3     20s
# check the services
> kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
eureka       ClusterIP   None            <none>        28888/TCP         44m
gateway      NodePort    10.106.21.126   <none>        28899:31000/TCP   44m
```

### Hit the `demo url`
```shell

# if you use `minikube` like me, you should get the ip of `minikube` as below:
vmip=$(minikube ip)

# get the port of gateway service to access
gwport=$(kubectl get svc -l app=gateway-default -o yaml |grep nodePort | awk '{print $2}' )
```

Hit the demo url via browser of other network tool: http://$(vmip):$(gwport)/user/{your-test-data}
