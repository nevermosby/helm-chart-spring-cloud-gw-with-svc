# spring-cloud-gw-with-svc

## 功能介绍

通过`spring-cloud-gw-with-svc`可以快速创建[spring cloud oss](https://spring.io/projects/spring-cloud-netflix)套件里的网关服务，并部署了一个示例应用[user-service](https://github.com/nevermosby/springcloudoss-user-service)，验证网关服务的作用，效果如下图所示：

![gw-demo](./images/gw-demo.PNG)

特别声明：由于spring cloud微服务架构依赖注册中心，因此本chart依赖[spring-cloud-eureka](https://) 源，已在Chart.yaml文件中声明：
```yaml
dependencies:
  - name: spring-cloud-eureka
    version: 0.1.0
    repository: https://apphub.aliyuncs.com/
```

## 安装使用

### 本地安装
```shell
helm install gw-svc spring-cloud-gw-with-svc/

```

成功安装后可以看到如下输出：
```shell
NAME: gw-svc
LAST DEPLOYED: 2019-08-26 17:11:31.5412552 +0800 CST
NAMESPACE: default
STATUS: deployed
```

### 查看部署好的workload：
```shell
# 查看所有pod
> kubectl get pod
NAME                           READY   STATUS    RESTARTS   AGE
eureka-0                       1/1     Running   0          43m
eureka-1                       1/1     Running   0          43m
eureka-2                       1/1     Running   0          43m
gateway-75c587d7dc-z6jw2       1/1     Running   0          43m
user-service-67567fd8b-bfcgg   1/1     Running   0          43m
# 查看eureka集群
> kubectl get sts
NAME     READY   AGE
eureka   3/3     20s
# 查看所有service
> kubect get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
eureka       ClusterIP   None            <none>        28888/TCP         44m
gateway      NodePort    10.106.21.126   <none>        28899:31000/TCP   44m
```

### 访问网关测试页面
```shell
# 获取minikube的IP：
vmip=$(minikube ip)

# 获取网关服务的端口
gwport=$(kubectl get svc -l app=gateway-default -o yaml |grep nodePort | awk '{print $2}' )
```

浏览器访问 http://$(vmip):$(gwport)/user/{your-test-data}
