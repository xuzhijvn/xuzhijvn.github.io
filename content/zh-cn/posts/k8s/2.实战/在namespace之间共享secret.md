在namespace之间共享secret

有的时候在default空间下创建了拉取镜像的secret，当部署k8s的资源到其他namespace的时候，如果部署的的是deployment的之类的势必会要拉取镜像，这个时候必然失败，因为secret创建在default空间下，所以我们需要将secret复制一份到需要的namespace下面：

```shell
kubectl get secret coding-regcred --namespace=default -oyaml | grep -v '^\s*namespace:\s' | kubectl apply --namespace=tkb -f -
```

 

参考链接：[Kubernetes - sharing secret across namespaces](https://stackoverflow.com/questions/46297949/kubernetes-sharing-secret-across-namespaces)

