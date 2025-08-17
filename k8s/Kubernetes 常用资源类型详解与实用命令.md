---
source: "https://www.cnblogs.com/leojazz/p/18831432"
tags:
  - "k8s"
---
## 一、什么是 Kubernetes 资源类型

Kubernetes 的一切都是“资源（Resource）”。资源即API对象，用来描述K8S中的各种实体，比如 Pod、Service、Deployment、Node、Namespace、ConfigMap 等。 **理解资源类型，是学习K8S的基础。**

---

## 二、查看所有资源类型

在K8S集群主节点或拥有kubectl权限的机器上，执行：

```bash
kubectl api-resources
```

你将得到如下类似输出（仅截取部分作为举例）：

| NAME | SHORTNAMES | APIVERSION | NAMESPACED | KIND |
| --- | --- | --- | --- | --- |
| pods | po | v1 | true | Pod |
| services | svc | v1 | true | Service |
| deployments | deploy | apps/v1 | true | Deployment |
| nodes | no | v1 | false | Node |
| namespaces | ns | v1 | false | Namespace |
| ... | ... | ... | ... | ... |

其中， **NAMESPACED** 表示该资源是否属于某个命名空间。

---

## 三、K8S核心资源类型详解

### 1\. 工作负载类

此类资源决定了你的应用程序如何运行和扩展：

- **Pod（po）**  
	K8S最小的可调度单元，是容器的载体。实际生产中很少直接运维裸Pod，通常用在Job等场景。
- **ReplicaSet（rs）**  
	保证指定副本数的Pod持续运行。多由Deployment控制间接管理，不直接编写。
- **Deployment（deploy）**  
	最常用的部署资源，支持回滚、滚动升级。日常生产用的最多，如Web应用、API服务等。
- **StatefulSet（sts）**  
	有状态服务（如MySQL、Redis集群）专用，Pod有固定标识符和持久存储。
- **DaemonSet（ds）**  
	用于每个（或指定）Node上自动运行一个Pod，例如日志收集、监控agent。
- **Job / CronJob**  
	Job用于一次性任务，CronJob实现定时调度（可类比Linux的crontab）。

### 2\. 服务发现与负载均衡

- **Service（svc）**  
	集群内持久服务入口，实现Pod间或外部服务访问负载均衡。
- **Endpoints / EndpointSlice**  
	Service关联的实际Pod IP存储对象，自动管理，无需手动变更。
- **Ingress**  
	七层路由，支持HTTP/S流量的准入、反向代理、SSL。

### 3\. 存储管理

- **PersistentVolume（pv）、PersistentVolumeClaim（pvc）**  
	PV是系统管理员提供的物理存储，PVC是用户申请的逻辑存储，两者绑定让Pod能持久存储。
- **StorageClass（sc）**  
	提供动态存储卷选项，简化PVC自动分配。

### 4\. 配置与密钥

- **ConfigMap（cm）**  
	存储配置数据，诸如环境变量、配置文件。明文存储，非敏感信息。
- **Secret**  
	存储敏感数据，例如密码、密钥、token，默认base64编码（需注意不是加密，生产需结合KMS等方案）。

### 5\. 权限与安全

- **ServiceAccount（sa）、Role/ClusterRole、RoleBinding/ClusterRoleBinding**  
	实现K8S的RBAC权限控制，精细划分用户和服务的集群操作范围。
- **PodSecurityPolicy（psp）**  
	现已废弃（K8S v1.21开始，v1.25完全移除），建议用Pod Security标准（如baseline、restricted）替代。
- **NetworkPolicy（netpol）**  
	网络层面的访问控制，是生产环境强烈建议启用的安全措施。

### 6\. 集群级管理

- **Namespace（ns）**  
	多租户/隔离环境的实现基础，推荐每个业务系统独占一个namespace。
- **Node（no）**  
	集群中的物理机/虚拟机。不可随意操作，部分命令需集群管理员权限。

### 7\. 其他高阶与扩展

- **CRD（CustomResourceDefinition）**  
	用户自定义资源类型。实现Operator、Service Mesh、云原生数据库等扩展生态的基石。
- **AdmissionWebhook**  
	动态准入控制（如MutatingWebhook/ValidatingWebhook），提供更灵活的安全、合规管理。

---

## 四、生产环境常用命令汇总

- **查看所有资源类型及缩写**
	```bash
	kubectl api-resources
	```
- **查看单个namespace下所有资源**
	```bash
	kubectl get all -n <namespace>
	```
- **按Kind查看资源**
	```bash
	kubectl get deployment,svc,pod -n <namespace>
	```
- **查单一资源（如Pod日志、YAML）**
	```bash
	kubectl logs pod-name -n <namespace>
	kubectl get pod pod-name -o yaml -n <namespace>
	```

---

## 五、生产环境实践建议

1. **使用Deployment/StatefulSet来管理Pod副本，严禁直接管理裸Pod** 。
2. **所有部署都建议隔离到独立namespace，便于权限、配额、安全收敛** 。
3. **严格启用RBAC、NetworkPolicy到最小权限原则** 。
4. **尽量用ConfigMap、Secret优雅解耦配置与代码，提高安全** 。
5. **存储用PVC和StorageClass动态方案，避免存储漂移** 。
6. **熟悉自定义扩展（CRD），为开源生态打下基础** 。
7. **及时关注k8s版本变化，如PSP已废弃，需切换其他安全方案** 。

---