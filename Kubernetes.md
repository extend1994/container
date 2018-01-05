# **Kubernetes** 學習筆記



##  介紹

1. Kubernets 中的最小單位 pod

   > Kubernetes 上會運行很多個不同種類型的 applications，而一個 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) 在 Kubernetes 世界中就相當於一個 application，一定會跑在機器 Node 上

   Kubernetes Cluster 內部有一套網路系統來替每個 Pod 建立一個 Cluster IP，這個 IP 是由 Kubernetes 內部***隨機***產生的，並且只能被 Cluster 內部資源所利用，外部無法透過 Cluster IP 與 pod 互動。

   若有需求，要透過創建 ***service*** ，讓 Cluster 以外的服務來跟 pod 互動。

2. Node

   > 運行 Kubernets 環境的「機器」，每個 Node 都由 "Master" 管理
   >
   > Master 會自動處理 Cluster 中不同 Node 的 Pod 排程（給予可用資源）
   >
   > Node 中至少必須有 
   >
   > * ***Kubelet***
   >   負責 Master 和 Node 間溝通的程序，管理了運行在 Node 上的 Pods 和 containers
   >
   >
   > * ***Container (runtime) engine***
   >   如 Docker，用來從 registry 拉 Image，然後運行 Container 來提供 Application

   <img align="center" width="70%" src="https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg">

3. Kubernetes Services

   > A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods

4. ***Replication Controller***

   > Kubernetes 中，管理 Pod 數量及狀態的 controller

   1. 會有自己的 *yaml* 配置（設定）檔，可指定同時有多少個相同的 Pods 運行於 Kuberbetes cluster 上
   2. 若 Pod 停止運行，controller 會偵測並重建 Pod 以符合配置檔的設定

5. Rolling update

   更新 Deployment

   ```
   kubectl set image deployment/<dep_name> <dep_name>=<image>:<version>
   kubectl rollout status deployment <>
   kubectl rollout undo deployment <>
   ```

Kubernetes 上，當我們將 [Node](https://kubernetes.io/docs/concepts/architecture/nodes) 都加到 Kubernetes Cluster 後，系統會根據目前 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) 的設定檔去決定要部署在哪個 [Node](https://kubernetes.io/docs/concepts/architecture/nodes) 上

### 架構

![](https://github.com/zxcvbnius/k8s-30-day-sharing/blob/master/Day06/kubernetes-orverview.png?raw=t)

> * **Load balancer (Load balancing)**：request 都要先經過這邊，再決定要此 request 要焦油哪個 Node 處理
> * Kubernetes **cluster** 中有 N 個 **Node**，Node 中會有 container engine（本筆記中會搭配 Docker 這個 container engine 使用）
> * Node 中
>   * 在 **Node** 的 engine 中會運行多個 **Pod**
>   * **iptable**
>     相當於防火牆，並有管理網路如何連線的功能（如 request 要由哪個 pod 來處理）
>   * **kubelet**
>     扮演 *agent* 的角色，管理 Pods （建立、更新等）並與 master node 溝通
>   * **kube-proxy**
>     將 Pod 資訊傳到 **iptable** ，確保 Pod 可以被 cluster 中的其他物件存取
> * 一個 **Pod** 中可以有多個 **Containers**
>   * 每個 Pod 有屬於自己的網路環境（在同個 Pod 下的 containers 如同在內網）
    * shared storage, e.g Volumes 

#### 舉例：創建一個 Pod，接收 reqeust 

1. 以 **master node** 身份命令 **kubelet** 建立 Pod
2. （建立完成後）**kube-proxy** 傳 Pod 資訊給 **iptable**
3. **Load balancer**  決定將 request 交給哪個 **Node** 處理
4. 該 **Node** 的 **iptable**  決定將 request 交給 **Node** 中的哪個 **Pod** 處理，**<u>或是將之轉給別的 Node 處理</u>**

## 環境需求

Linux distribution

## 安裝

```shell
# 1. Install virtualbox or other Virtualization Software
# 2. kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl &&  chmod +x ./kubectl && sudo mv ./kubectl /usr/local/bin/kubectl
# 3. minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
minikube start
# Now should generate ~/.kube/config
# Check wih `kubectl get node` later
```

## 使用 minikube 

> minikube 幫助使用者架設一個 VM 並運行  (single node) Kubernetes cluster ，讓使用者可以輕易在 local 端使用 Kubernetes 環境，也就是在 Kubernetes 扮演 ***Node*** 的角色

```shell
minikube dashboard
# Access the Kubernetes Dashboard
minikube dashboard
# Access a service exposed via a node port
minikube service [-n NAMESPACE] [--url] NAME

```



## 使用 kubectl 

> **kuberctl 是 Kubernetes 中 Controller 的角色** 
>
> https://kubernetes.io/docs/reference/kubectl/overview/
>
> https://github.com/kubernetes/minikube#quickstart

### 建立 Kubernets 中的最小單位 pod

> 使用 ***yaml***，每個 pod 會有自己對應的 yaml 檔

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: webserver
spec:
  containers:
  - name: pod-demo
    image: zxcvbnius/docker-demo
    ports:
    - containerPort: 3000
```

* apiVersion: https://kubernetes.io/docs/reference/api-overview/#api-versioning

* metadata includes keys:

  * **name**

    Pod 名稱

  * **labels**

    Kubernetes 的是核心的元件之一，Kubernetes 會透過 `Label Selector` 將Pod分群管理

  * **annotations**

    與 labels 相似。相較於labels，`annotations 通常是使用者任意自定義的附加資訊`，提供外部進行查詢使用，像是版本號，發布日期等等

* spec: 定義 containers，此筆記中，僅運行一個 container

  * **name**

    container 名稱


  * **image**

    於 Docker Registry 的可下載路徑

  * **ports**

    指定可供外部存取的 port

```shell
kubectl create -f <yaml file>

##### kubectl run: 在 cluster 上運行指定的 image ~~ docker run #####
kubectl run [-i] [--tty] --attach <name> --image=<image>
kubectl run -i --tty alpine --image=alpine --restart=Never -- sh
##### kubectl get: 正在運行服務的清單 ~~ docker ps #####
kubectl get pods [--show-all] [--show-labels] [-o <output_format>]
kubectl get pods <pod_name> <label-key>=<label-value>
kubectl get pods/services -l <labels_from_describe>
kubectl describe pods <pod_name>
kubectl port-forward <pod_name> <extern_port>:<pod_port>
kubectl expose pod <pod_name> --type=<port> --name=<k8s_service_name> # create new service through exposing pod
# type ref: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/
kubectl get services
# attach a pod to see logs
kubectl attach <pod> -i
# execuate command on a pod
kubectl exec <pod> -- <command>
# label a pod
kubectl label pods <pod_name> <label-key>=<label-value> [version=<version>]
# scale
kubectl scale deployment <deployment> --replicas=<scale_number>
kubectl scale deployment hello-node --replicas=2
```



```shell
# all
minikube service hello-minikube --url
kubectl get services
kubectl describe pods hello-minikube-65df575c69-w5qsn
```

### 擴展 Pods (Scaling & Replication Controller)

#### Stateful v.s Stateless

* Stateful: 像是過去傳統式關聯式資料庫，資料並不會因為服務重啟，而有改變。
* Stateless: 不因時間、資料寫入等狀態的改變而影響服務，是 web 服務應該要有的狀態

#### Scaling

* vertical：單一節點上，增加 CPU, RAM 等來獲得更多運作資源
* horizental：增加更多節點。

#### Replication Controller 的 yaml 設置檔

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-replication-controller
spec:
  replicas: 3
  selector:
    app: hello-pod-v1
  template:
    metadata:
      labels:
        app: hello-pod-v1
    spec:
      containers:
      - name: my-pod
        image: zxcvbnius/docker-demo
        ports:
        - containerPort: 3000
```

* spec
  * **replicas**
    Pod 數量
  * **selector**
    要套用的 Pod labels
  * **template**
    定義 Pod 資訊，包含 labels 及要運行在 Pod 中的 container 資訊

## Tutorials

### create nodejs application

1. simple http server
2. dockerfile to build nodejs env and run simple server
3. create deployment

## Reference

* 30天學習筆記鐵人賽
  * https://ithelp.ithome.com.tw/users/20103753/ironman/1590?page=1
* 安裝
  * https://github.com/kubernetes/minikube
  * https://kubernetes.io/docs/tasks/tools/install-kubectl/
* docker to kubectl
  * https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/
  * ​
* kubectl commands
  * https://kubernetes-v1-4.github.io/docs/user-guide/kubectl-overview/



```shell
kubectl run hello-node --image=hello-node:v1 --port=8080
kubectl expose deployment hello-node --type=LoadBalancer
```

