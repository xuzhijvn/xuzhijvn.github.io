---
title: "operator实战"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"云原生"
]
tags : [
"云原生",
"k8s"
]
series : [
"k8s实战"
]
images : [

]
---


# operator实战

## 1⃣️ 需求

我们将定义一个 crd ，spec 包含以下信息：

```
Replicas	# 副本数
Image		# 镜像
Resources	# 资源限制
Envs		# 环境变量
Ports		# 服务端口
```

## 2⃣️ 编码

初始化项目目录：

```bash
tony@192 k8s % pwd
/Users/tony/workspace/k8s
tony@192 k8s % mkdir -p app-operator/src/github.com/xuzhijvn/app
cd app-operator/src/github.com/xuzhijvn/app
```

初始化operator项目结构，并创建api：

```bash
 operator-sdk init --domain=huolala.cn --repo=github.com/xuzhijvn/app
 operator-sdk create api --group app --version v1 --kind App --resource=true --controller=true
```

修改 CRD 类型定义代码 api/v1/app_types.go:

```go
/*
Copyright 2021.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package v1

import (
	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// EDIT THIS FILE!  THIS IS SCAFFOLDING FOR YOU TO OWN!
// NOTE: json tags are required.  Any new fields you add must have json tags for the fields to be serialized.

// AppSpec defines the desired state of App
type AppSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// Foo is an example field of App. Edit app_types.go to remove/update
	//Foo string `json:"foo,omitempty"`
	Replicas  *int32                      `json:"replicas"`            // 副本数
	Image     string                      `json:"image"`               // 镜像
	Resources corev1.ResourceRequirements `json:"resources,omitempty"` // 资源限制
	Envs      []corev1.EnvVar             `json:"envs,omitempty"`      // 环境变量
	Ports     []corev1.ServicePort        `json:"ports,omitempty"`     // 服务端口
}

// AppStatus defines the observed state of App
type AppStatus struct {
	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
	// Important: Run "make" to regenerate code after modifying this file
	appsv1.DeploymentStatus `json:",inline"` // 直接引用 DeploymentStatus
}

//+kubebuilder:object:root=true
//+kubebuilder:subresource:status

// App is the Schema for the apps API
type App struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   AppSpec   `json:"spec,omitempty"`
	Status AppStatus `json:"status,omitempty"`
}

//+kubebuilder:object:root=true

// AppList contains a list of App
type AppList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata,omitempty"`
	Items           []App `json:"items"`
}

func init() {
	SchemeBuilder.Register(&App{}, &AppList{})
}

```

新增 resource/deployment/deployment.go：

```go
package deployment

import (
	appv1 "github.com/xuzhijvn/app/api/v1"
	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime/schema"
)

func New(app *appv1.App) *appsv1.Deployment {
	labels := map[string]string{"app.example.com/v1": app.Name}
	selector := &metav1.LabelSelector{MatchLabels: labels}
	return &appsv1.Deployment{
		TypeMeta: metav1.TypeMeta{
			APIVersion: "apps/v1",
			Kind:       "Deployment",
		},
		ObjectMeta: metav1.ObjectMeta{
			Name:      app.Name,
			Namespace: app.Namespace,
			OwnerReferences: []metav1.OwnerReference{
				*metav1.NewControllerRef(app, schema.GroupVersionKind{
					Group:   appv1.GroupVersion.Group,
					Version: appv1.GroupVersion.Version,
					Kind:    "App",
				}),
			},
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: app.Spec.Replicas,
			Selector: selector,
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: labels,
				},
				Spec: corev1.PodSpec{
					Containers: newContainers(app),
				},
			},
		},
	}
}

func newContainers(app *appv1.App) []corev1.Container {
	var containerPorts []corev1.ContainerPort
	for _, servicePort := range app.Spec.Ports {
		var cport corev1.ContainerPort
		cport.ContainerPort = servicePort.TargetPort.IntVal
		containerPorts = append(containerPorts, cport)
	}
	return []corev1.Container{
		{
			Name:            app.Name,
			Image:           app.Spec.Image,
			Ports:           containerPorts,
			Env:             app.Spec.Envs,
			Resources:       app.Spec.Resources,
			ImagePullPolicy: corev1.PullIfNotPresent,
		},
	}
}
```

新增 resource/service/service.go

```go
package service

import (
	appv1 "github.com/xuzhijvn/app-operator/api/v1"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime/schema"
)

func New(app *appv1.App) *corev1.Service {
	return &corev1.Service{
		TypeMeta: metav1.TypeMeta{
			Kind:       "Service",
			APIVersion: "v1",
		},
		ObjectMeta: metav1.ObjectMeta{
			Name:      app.Name,
			Namespace: app.Namespace,
			OwnerReferences: []metav1.OwnerReference{
				*metav1.NewControllerRef(app, schema.GroupVersionKind{
					Group:   appv1.GroupVersion.Group,
					Version: appv1.GroupVersion.Version,
					Kind:    "App",
				}),
			},
		},
		Spec: corev1.ServiceSpec{
			Ports: app.Spec.Ports,
			Selector: map[string]string{
				"app.example.com/v1": app.Name,
			},
		},
	}
}
```

修改 controller 代码 controllers/app_controller.go

```go
/*
Copyright 2021.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package controllers

import (
	"context"
	"encoding/json"
	"github.com/xuzhijvn/app/resource/deployment"
	"github.com/xuzhijvn/app/resource/service"
	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/api/errors"
	"reflect"

	"github.com/go-logr/logr"
	"k8s.io/apimachinery/pkg/runtime"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"

	appv1 "github.com/xuzhijvn/app-operator/api/v1"
)

// AppReconciler reconciles a App object
type AppReconciler struct {
	client.Client
	Log    logr.Logger
	Scheme *runtime.Scheme
}

//+kubebuilder:rbac:groups=app.huolala.cn,resources=apps,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=app.huolala.cn,resources=apps/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=app.huolala.cn,resources=apps/finalizers,verbs=update

// Reconcile is part of the main kubernetes reconciliation loop which aims to
// move the current state of the cluster closer to the desired state.
// TODO(user): Modify the Reconcile function to compare the state specified by
// the App object against the actual cluster state, and then
// perform operations to make the cluster state reflect the state specified by
// the user.
//
// For more details, check Reconcile and its Result here:
// - https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.8.3/pkg/reconcile
func (r *AppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	_ = r.Log.WithValues("app", req.NamespacedName)

	// your logic here

	// 获取 crd 资源
	instance := &appv1.App{}
	if err := r.Client.Get(ctx, req.NamespacedName, instance); err != nil {
		if errors.IsNotFound(err) {
			return ctrl.Result{}, nil
		}
		return ctrl.Result{}, err
	}

	// crd 资源已经标记为删除
	if instance.DeletionTimestamp != nil {
		return ctrl.Result{}, nil
	}

	oldDeploy := &appsv1.Deployment{}
	if err := r.Client.Get(ctx, req.NamespacedName, oldDeploy); err != nil {
		// deployment 不存在，创建
		if errors.IsNotFound(err) {
			// 创建deployment
			if err := r.Client.Create(ctx, deployment.New(instance)); err != nil {
				return ctrl.Result{}, err
			}

			// 创建service
			if err := r.Client.Create(ctx, service.New(instance)); err != nil {
				return ctrl.Result{}, err
			}

			// 更新 crd 资源的 Annotations
			data, _ := json.Marshal(instance.Spec)
			if instance.Annotations != nil {
				instance.Annotations["spec"] = string(data)
			} else {
				instance.Annotations = map[string]string{"spec": string(data)}
			}
			if err := r.Client.Update(ctx, instance); err != nil {
				return ctrl.Result{}, err
			}
		} else {
			return ctrl.Result{}, err
		}
	} else {
		// deployment 存在，更新
		oldSpec := appv1.AppSpec{}
		if err := json.Unmarshal([]byte(instance.Annotations["spec"]), &oldSpec); err != nil {
			return ctrl.Result{}, err
		}

		if !reflect.DeepEqual(instance.Spec, oldSpec) {
			// 更新deployment
			newDeploy := deployment.New(instance)
			oldDeploy.Spec = newDeploy.Spec
			if err := r.Client.Update(ctx, oldDeploy); err != nil {
				return ctrl.Result{}, err
			}

			// 更新service
			newService := service.New(instance)
			oldService := &corev1.Service{}
			if err := r.Client.Get(ctx, req.NamespacedName, oldService); err != nil {
				return ctrl.Result{}, err
			}
			clusterIP := oldService.Spec.ClusterIP // 更新 service 必须设置老的 clusterIP
			oldService.Spec = newService.Spec
			oldService.Spec.ClusterIP = clusterIP
			if err := r.Client.Update(ctx, oldService); err != nil {
				return ctrl.Result{}, err
			}

			// 更新 crd 资源的 Annotations
			data, _ := json.Marshal(instance.Spec)
			if instance.Annotations != nil {
				instance.Annotations["spec"] = string(data)
			} else {
				instance.Annotations = map[string]string{"spec": string(data)}
			}
			if err := r.Client.Update(ctx, instance); err != nil {
				return ctrl.Result{}, err
			}
		}
	}

	return ctrl.Result{}, nil
}

// SetupWithManager sets up the controller with the Manager.
func (r *AppReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&appv1.App{}).
		Complete(r)
}
```

修改 CRD 资源定义 config/samples/app_v1_app.yaml

```yaml
apiVersion: app.huolala.cn/v1
kind: App
metadata:
  name: nginx-app
  namespace: default
spec:
  # Add fields here
  replicas: 2
  image: nginx:1.16.1
  ports:
    - targetPort: 80
      port: 8080
  envs:
    - name: DEMO
      value: app
    - name: GOPATH
      value: gopath
  resources:
    limits:
      cpu: 500m
      memory: 500Mi
    requests:
      cpu: 100m
      memory: 100Mi
```

修改 Dockerfile

```dockerfile
# Build the manager binary
FROM golang:1.15 as builder

WORKDIR /workspace
# Copy the Go Modules manifests
COPY go.mod go.mod
COPY go.sum go.sum
# cache deps before building and copying source so that we don't need to re-download as much
# and so that source changes don't invalidate our downloaded layer

ENV GOPROXY https://goproxy.cn,direct

RUN go mod download

# Copy the go source
COPY main.go main.go
COPY api/ api/
COPY controllers/ controllers/
COPY resource/ resource/

# Build
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -a -o manager main.go

# Use distroless as minimal base image to package the manager binary
# Refer to https://github.com/GoogleContainerTools/distroless for more details
#FROM gcr.io/distroless/static:nonroot
FROM kubeimages/distroless-static:latest
WORKDIR /
COPY --from=builder /workspace/manager .
USER 65532:65532

ENTRYPOINT ["/manager"]
```

- 添加了 goproxy 环境变量
- 新增 COPY 自定义的文件夹 resource
- `gcr.io/distroless/static:nonroot` 变更为 `kubeimages/distroless-static:latest`

## 3⃣️ 部署

xuzhijvn 项目根目录执行：

### 安装crd到集群

```bash
make generate && make manifests && make install
```

结果确认

```bash
[root@k8s-master ~]# kubectl get crd | grep huolala
apps.app.huolala.cn                                   2021-06-27T13:42:09Z
```

### 构建镜像

```bash
make docker-build IMG=ccr.ccs.tencentyun.com/huolala.cn/app:v1
```

==特别注意==☢️

⚠️ 1. 因为docker-build会执行测试，测试的时候需要连接github下载脚本运行，此时极有可能会报443的连接错误，所以我们把test注释掉。

![image-20210627223227260](https://cdn.tkaid.com/img/image-20210627223227260.png)



⚠️ 2. 下图位置默认的镜像地址是`gcr.io/kubebuilder/kube-rbac-proxy:v0.8.0` 这个地址国内无法拉取到镜像，所以需要替换成自己的镜像地址（可以先找台国外的服务器拉取下来，重新打tag后推送到自己的镜像仓库），如果不修改，容器`ccr.ccs.tencentyun.com/huolala.cn/app:v1`无法启动。

![image-20210627223536610](https://cdn.tkaid.com/img/image-20210627223536610.png)

⚠️ 3. 将 controller 部署到 k8s 集群的时候，可能会出现 RBAC 权限错误，解决方法是修改部署时的权限配置，这里我们使用最简单的方法是直接给 controller 绑定到 cluster-admin 集群管理员即可

![image-20210627225840775](https://cdn.tkaid.com/img/image-20210627225840775.png)

如果不修改，报错如下：

![image-20210627230328638](https://cdn.tkaid.com/img/image-20210627230328638.png)

### 推送镜像到仓库

这里直接使用docker push，因为make docker-push也只使用了docker push

```bash
 docker push ccr.ccs.tencentyun.com/huolala.cn/app:v1 
```

### 部署

```bash
 make deploy IMG=ccr.ccs.tencentyun.com/huolala.cn/app:v1   
```

结果确认

```bash
[root@k8s-master ~]# kubectl get svc -n app-system
NAME                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
app-controller-manager-metrics-service   ClusterIP   10.107.174.87   <none>        8443/TCP   12s
[root@k8s-master ~]# kubectl get pod -n app-system
NAME                                     READY   STATUS    RESTARTS   AGE
app-controller-manager-c49568d65-86tpj   2/2     Running   0          22s
[root@k8s-master ~]# kubectl get deployment -n app-system
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
app-controller-manager   1/1     1            1           24s
```

![image-20210628092101436](https://cdn.tkaid.com/img/image-20210628092101436.png)



### 创建自定义资源

```bash
$ kubectl apply -f config/samples/app_v1_app.yaml 
app.app.example.com/app-sample created
```

结果确认

```bash
[root@k8s-master ~]# kubectl get svc | grep nginx
nginx-app                    ClusterIP      10.103.202.0     <none>        8080/TCP         73m
[root@k8s-master ~]# kubectl get pod | grep nginx
nginx-app-57d9d68fb7-8rpc6                           2/2     Running   0          73m
nginx-app-57d9d68fb7-cwnv9                           2/2     Running   0          73m
[root@k8s-master ~]# curl http://10.103.202.0:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@k8s-master ~]# 
```

## 参考链接🔗

[Kubernetes Operator 快速入门教程](https://www.qikqiak.com/post/k8s-operator-101/)

[operator-sdk 实战开发](https://www.cnblogs.com/leffss/p/14732645.html)
