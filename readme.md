# CloudAuditTool

## 免责声明
本程序应仅用于授权的安全测试与研究目的。 由于传播、利用本工具而造成的任何直接或者间接的后果及损失，均由使用者负责，工具作者不为此承担任何责任。


## 总览
该工具用于测试和验证云平台是否存在一定的安全隐患，自动化识别容器和节点以及云平台归属,协助甲方安全人员对云环境做安全评估测试，可以评估容器，k8s，云主机安全等新基础设施安全

```
1.自动化识别容器环境和主机环境，识别云归属厂商
2.自动化根据云厂商从Metadata读取敏感信息
3.自动化枚举主机和容器凭证权限，以及枚举各个凭证所具备的权限，避免主机存在高风险凭证，例如etcd证书，k8s管理员凭证等，当攻击者获取会直接可以攻击整个集群
4.自动化检测容器是否隔离（网络，进程）
5.自动化信息收集，包括敏感挂载，环境变量等各类在容器化环境，或者渗透环境中常见的各类高危项,Docker登录凭证自动解密,协助在云环境安全评估的工作
6.自动化集成Kubectl,etcdctl等工具，方便实用
7.具备扫描关键组件的未授权漏洞(Apiserver,Etcd,kubelet,Dockersock)
```

## 使用方法
```
root@ubuntu-linux-22-04-desktop:/home/parallels/code/CloudPentestSuite# ./CloudPentestSuite
Usage:
  Test [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  info        信息收集模块
  scan        扫描模块

Flags:
  -h, --help   help for Test

```
info模块
```
root@ubuntu-linux-22-04-desktop:/home/parallels/code/CloudPentestSuite# ./CloudAuditTool info
信息收集模块

Usage:
   info [flags]
   info [command]

Available Commands:
  all             所有信息收集（尽量覆盖攻击面，凭据，文件，环境变量，所有Secret，网络，挂载等）
  apiserverunauth 查询是否存在Apiserver未授权
  cloudaccount    查询当前是否存在云账号以及权限
  critical        关键信息收集
  dockerconfig    Docker登录凭证
  env             查询当前环境变量
  etcdconfig      检索Etcd凭证
  history         查询bash history记录
  kubeconfig      检索Kubeconfig凭证
  mount           查询当前具备的容器权限
  namespace       检测命名空间隔离
  privilege       查询当前具备的容器权限
  process         查询当前所有进程
  senstivefile    查询当前环境变量
  serviceaccount  检索serviceaccount凭证以及权限
```

### 信息收集
#### 主机节点
```
[节点信息]
+----------------------------+--------------------------+----------+----------------------------+--------+
| 当前环境（容器内/主机节点) |      APISERVER地址       | K8S版本  |           节点名           | 云平台 |
+----------------------------+--------------------------+----------+----------------------------+--------+
| 主机节点                   | https://10.211.55.6:6443 | v1.23.17 | ubuntu-linux-22-04-desktop | 未知   |
+----------------------------+--------------------------+----------+----------------------------+--------+
[Docker历史凭据]
+----------------------------+
| DOCKER镜像仓库的账号与密码 |
+----------------------------+
| testtest:asdddds        |
+----------------------------+
[凭证类收集]
+------------------------------+--------------------------+----------------------------------------+
|      KUBECONFIG凭证路径      | 凭证指向的APISERVER地址  |                凭证用户                |
+------------------------------+--------------------------+----------------------------------------+
| /etc/kubernetes/admin.conf   | https://10.211.55.6:6443 | kubernetes-admin                       |
| /etc/kubernetes/kubelet.conf | https://10.211.55.6:6443 | system:node:ubuntu-linux-22-04-desktop |
+------------------------------+--------------------------+----------------------------------------+
[Etcd凭证类收集]
+-------------------------------------------------+
|                  ETCD凭证路径                   |
+-------------------------------------------------+
| /etc/kubernetes/pki/etcd/ca.crt                 |
| /etc/kubernetes/pki/etcd/ca.key                 |
| /etc/kubernetes/pki/etcd/healthcheck-client.crt |
| /etc/kubernetes/pki/etcd/healthcheck-client.key |
| /etc/kubernetes/pki/etcd/peer.crt               |
| /etc/kubernetes/pki/etcd/peer.key               |
| /etc/kubernetes/pki/etcd/server.crt             |
| /etc/kubernetes/pki/etcd/server.key             |
+-------------------------------------------------+
凭证权限: /etc/kubernetes/admin.conf
Resources                                       Non-Resource URLs   Resource Names   Verbs
*.*                                             []                  []               [*]
get,watch,list.*                                []                  []               [*]
                                                [*]                 []               [*]
selfsubjectaccessreviews.authorization.k8s.io   []                  []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []               [create]
                                                [*]                 []               [get,watch,list]
                                                [/api/*]            []               [get]
                                                [/api]              []               [get]
                                                [/apis/*]           []               [get]
                                                [/apis]             []               [get]
                                                [/healthz]          []               [get]
                                                [/healthz]          []               [get]
                                                [/livez]            []               [get]
                                                [/livez]            []               [get]
                                                [/openapi/*]        []               [get]
                                                [/openapi]          []               [get]
                                                [/readyz]           []               [get]
                                                [/readyz]           []               [get]
                                                [/version/]         []               [get]
                                                [/version/]         []               [get]
                                                [/version]          []               [get]
                                                [/version]          []               [get]
[ServiceAccount收集]
+----------------------------+------------------------+
|           POD名            |   SERIVICEACCOUNT名    |
+----------------------------+------------------------+
| detector-xr57d             | sectest:lisi           |
| coredns-6d8c4cb4d-v2v6s    | kube-system:coredns    |
| ubuntu-linux-22-04-desktop | kube-flannel:flannel   |
| ubuntu-linux-22-04-desktop | kube-system:kube-proxy |
+----------------------------+------------------------+
[sectest:lisi]的路径和权限如下
/var/lib/kubelet/pods/3dee0948-8fcc-4e1f-8109-2fbbadd5fdd4/volumes/kubernetes.io~projected/kube-api-access-q6mlm/token


Resources                                       Non-Resource URLs                     Resource Names   Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                                    []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                    []               [create]
secrets                                         []                                    []               [get watch list]
                                                [/.well-known/openid-configuration]   []               [get]
                                                [/api/*]                              []               [get]
                                                [/api]                                []               [get]
                                                [/apis/*]                             []               [get]
                                                [/apis]                               []               [get]
                                                [/healthz]                            []               [get]
                                                [/healthz]                            []               [get]
                                                [/livez]                              []               [get]
                                                [/livez]                              []               [get]
                                                [/openapi/*]                          []               [get]
                                                [/openapi]                            []               [get]
                                                [/openid/v1/jwks]                     []               [get]
                                                [/readyz]                             []               [get]
                                                [/readyz]                             []               [get]
                                                [/version/]                           []               [get]
                                                [/version/]                           []               [get]
                                                [/version]                            []               [get]
                                                [/version]                            []               [get]


```
#### 容器
```
[节点信息]

+----------------------------+-----------------------+----------+---------------------------+--------+
| 当前环境（容器内/主机节点) |     APISERVER地址     | K8S版本  |          节点名           | 云平台 |
+----------------------------+-----------------------+----------+---------------------------+--------+
| 主机节点                   | https://10.96.0.1:443 | v1.23.17 | tomcat01-868cc7d965-8bcpc | 未知   |
+----------------------------+-----------------------+----------+---------------------------+--------+
[Apiserver未授权访问]
+--------------------------+--------------+
|         请求地址         | 可以匿名请求 |
+--------------------------+--------------+
| https://10.211.55.6:6443 | 否           |
+--------------------------+--------------+
[逃逸特权]
+---------------------+-----------------------------------------------------------------------------------------+
|      高危权限       |                                       可逃逸方式                                        |
+---------------------+-----------------------------------------------------------------------------------------+
| 特权容器            | 可以挂载cgroup，主机磁盘(需要禁用apparmor)等方式逃逸                                    |
| CAP_DAC_READ_SEARCH | 可以读取主机文件                                                                        |
| CAP_SYS_MODULE      | 可以通过挂载内核模块逃逸                                                                |
| CAP_SYS_PTRACE      | 如果和主机处于同一个PID命名空间，可以通过进程注入，在目标进行指定任意代码，完成容易逃逸 |
| CAP_SYS_ADMIN       | 可以通过mount命令挂载Cgroup逃逸（仅限于cgroup                                 |
|                     | v1版本）                                                                                |
+---------------------+-----------------------------------------------------------------------------------------+
[敏感挂载]
+---------------------------------------------------------------------------------+---------------------+----------+-----------+----------+
|                                      目录                                       |    容器中挂载点     | 文件类型 | 设备类型  | 读写方式 |
+---------------------------------------------------------------------------------+---------------------+----------+-----------+----------+
| /                                                                               | /                   | overlay  | overlay   | rw       |
| /                                                                               | /proc               | proc     | proc      | rw       |
| /                                                                               | /dev                | tmpfs    | tmpfs     | rw       |
| /                                                                               | /dev/pts            | devpts   | devpts    | rw       |
| /                                                                               | /sys                | sysfs    | sysfs     | ro       |
| /                                                                               | /sys/fs/cgroup      | cgroup2  | cgroup    | rw       |
| /                                                                               | /dev/mqueue         | mqueue   | mqueue    | rw       |
| /                                                                               | /dev/shm            | tmpfs    | shm       | rw       |
| /./01de2538ae94a894ef6c94a4f2768e37ec1da568834a7d8dd03f1347adf417bb/resolv.conf | /etc/resolv.conf    | ext4     | /dev/sda2 | rw       |
| /./01de2538ae94a894ef6c94a4f2768e37ec1da568834a7d8dd03f1347adf417bb/hostname    | /etc/hostname       | ext4     | /dev/sda2 | rw       |
| /./01de2538ae94a894ef6c94a4f2768e37ec1da568834a7d8dd03f1347adf417bb/hosts       | /etc/hosts          | ext4     | /dev/sda2 | rw       |
| /0                                                                              | /dev/console        | devpts   | devpts    | rw       |
| /bus                                                                            | /proc/bus           | proc     | proc      | rw       |
| /fs                                                                             | /proc/fs            | proc     | proc      | rw       |
| /irq                                                                            | /proc/irq           | proc     | proc      | rw       |
| /sys                                                                            | /proc/sys           | proc     | proc      | rw       |
| /sysrq-trigger                                                                  | /proc/sysrq-trigger | proc     | proc      | rw       |
| /                                                                               | /proc/asound        | tmpfs    | tmpfs     | ro       |
| /null                                                                           | /proc/kcore         | tmpfs    | tmpfs     | rw       |
| /null                                                                           | /proc/keys          | tmpfs    | tmpfs     | rw       |
| /null                                                                           | /proc/timer_list    | tmpfs    | tmpfs     | rw       |
| /                                                                               | /proc/scsi          | tmpfs    | tmpfs     | ro       |
| /                                                                               | /sys/firmware       | tmpfs    | tmpfs     | ro       |
+---------------------------------------------------------------------------------+---------------------+----------+-----------+----------+

[环境变量]
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
LANGUAGE=en_US:en
EXPLOIT_PORT_8080_TCP_PORT=8080
EXPLOIT_PORT_8080_TCP_PROTO=tcp
HOSTNAME=tomcat01-868cc7d965-8bcpc
TOMCAT_SHA512=b3177fb594e909364abc8074338de24f0441514ee81fa13bcc0b23126a5e3980cc5a6a96aab3b49798ba58d42087bf2c5db7cee3e494cc6653a6c70d872117e5
MYWEB_SERVICE_HOST=10.107.188.225
SHLVL=1
LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-lib
MYWEB_PORT_8080_TCP_ADDR=10.107.188.225
TOMCAT01_647F68FDC7_8GS6J_PORT=tcp://10.103.78.86:15672
EXPLOIT_PORT_8080_TCP_ADDR=10.104.57.67

[磁盘上可疑文件]
[命名空间隔离]
+--------------+----------+
|   命名空间   | 是否隔离 |
+--------------+----------+
| 进程命名空间 | 是       |
| 网络命名空间 | 是       |
+--------------+----------+
[dns探测]
10-211-55-6.kubernetes.default.svc.cluster.local. 6443
10-211-55-7.ingress-nginx-controller-admission.ingress-nginx.svc.cluster.local. 8443
10-211-55-7.ingress-nginx-controller.ingress-nginx.svc.cluster.local. 80
10-211-55-7.ingress-nginx-controller.ingress-nginx.svc.cluster.local. 443
10-244-0-238.kube-dns.kube-system.svc.cluster.local. 53
10-244-0-238.kube-dns.kube-system.svc.cluster.local. 9153
10-244-0-239.kube-dns.kube-system.svc.cluster.local. 53
10-244-0-239.kube-dns.kube-system.svc.cluster.local. 9153
10-244-1-66.exploit.default.svc.cluster.local. 8080
10-244-1-66.myweb.default.svc.cluster.local. 8080
exploit.default.svc.cluster.local. 8080
ingress-nginx-controller-admission.ingress-nginx.svc.cluster.local. 443
ingress-nginx-controller.ingress-nginx.svc.cluster.local. 80
ingress-nginx-controller.ingress-nginx.svc.cluster.local. 443
kube-dns.kube-system.svc.cluster.local. 53
kube-dns.kube-system.svc.cluster.local. 9153
kubernetes.default.svc.cluster.local. 443
myweb.default.svc.cluster.local. 8080
tomcat01-647f68fdc7-8gs6j.default.svc.cluster.local. 15672
[进程]
root    2355859 1
root    2355881 2355859 /usr/bin/dash
root    2355893 2355881 /usr/bin/sleep

[网络]
{16455868 172.17.0.6:55534 169.254.169.254:80 SYN_SENT 0 2366289/CloudAuditTool}

[云上账号权限]

[History列表]]
2024/09/10 03:12:37 err found while open /root/.bash_history
```
### 未授权扫描
##### 扫描所有关键组件的未授权
扫描所有包括Etcd,Apiserver,dockersock,kubelet组件的未授权
```
root@ubuntu-linux-22-04-desktop:/home/parallels/code/CloudPentestSuite# ./CloudPentestSuite scan all --cidr=10.211.55.6/24
┌───────────────────────────────┐
│         Kubelet未授权         │
├───┬───────────────────────────┤
│   │ 未授权地址                │
├───┼───────────────────────────┤
│ 1 │ https://10.211.55.6:10250 │
└───┴───────────────────────────┘
┌──────────────────────────────┐
│        Apiserver未授权       │
├───┬──────────────────────────┤
│   │ 未授权地址               │
├───┼──────────────────────────┤
│ 1 │ https://10.211.55.6:6443 │
└───┴──────────────────────────┘
┌─────────────────────────────┐
│       Dockersock未授权      │
├───┬─────────────────────────┤
│   │ 未授权地址              │
├───┼─────────────────────────┤
│ 1 │ http://10.211.55.6:2375 │
└───┴─────────────────────────┘
```

##### Apiserver
```
root@ubuntu-linux-22-04-desktop:/home/parallels/code/CloudPentestSuite# ./CloudPentestSuite scan apiserver --cidr=8.130.97.27
┌──────────────────────────────┐
│        Apiserver未授权       │
├───┬──────────────────────────┤
│   │ 未授权地址               │
├───┼──────────────────────────┤
│ 1 │ https://10.211.55.6:6443 │
└───┴──────────────────────────┘

```
