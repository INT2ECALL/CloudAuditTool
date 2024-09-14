# CloudAuditTool

## 免责声明
本程序应仅用于授权的安全测试与研究目的。 由于传播、利用本工具而造成的任何直接或者间接的后果及损失，均由使用者负责，工具作者不为此承担任何责任。


## 总览
该工具用于测试和验证云平台是否存在一定的安全隐患，自动化识别容器和节点以及云平台归属,协助甲方安全人员对云环境做安全评估测试，可以评估容器，k8s，云主机安全等新基础设施安全

```
1.自动化识别容器环境和主机环境，识别云归属厂商
2.自动化根据云厂商从Metadata读取敏感信息
3.自动化枚举主机和容器凭证权限，以及枚举各个凭证所具备的权限，避免主机存在高风险凭证，例如etcd证书，k8s管理员凭证等，当攻击者获取会直接可以攻击整个集群
4.具备扫描关键组件的未授权漏洞(Apiserver,Etcd,kubelet,Dockersock)
5.自动化检测容器是否隔离（网络，进程）
6.自动化信息收集，包括敏感挂载，环境变量等各类在容器化环境，或者渗透环境中常见的各类高危项,Docker登录凭证自动解密,协助在云环境安全评估的工作
7.自动化集成Kubectl,etcdctl等工具，一键命令安装
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
overlay on / type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/NHRL2DAYKUQQLCAAXWEZJKA5SX:/var/lib/docker/overlay2/l/NE6C2PO76GTZBTCOSBNHB6SWLE:/var/lib/docker/overlay2/l/KUBQBW547V5H56IKUUG5CGJK7C:/var/lib/docker/overlay2/l/EHPY4M746XJVN25TXRI25LSLSF:/var/lib/docker/overlay2/l/TOHQ6MANDIWKXYIGZ5QOX2GV7Z:/var/lib/docker/overlay2/l/IMJVRAWRCFRQDEVVJKTVCRITXI:/var/lib/docker/overlay2/l/JALLKVQ546MOPKI7SXV2NZSWE3:/var/lib/docker/overlay2/l/UCBAPBCKVPGATKHZANLO4JTGI4:/var/lib/docker/overlay2/l/QSHHGHOPA46ZMJOHSOPNSKSTCB:/var/lib/docker/overlay2/l/VOCWYSV6QXMF7BMX54ACUSGNR2,upperdir=/var/lib/docker/overlay2/5e4711091ab70078dc54bc771ee2a7c29f3cdb5213dd30eb18a86768fd245913/diff,workdir=/var/lib/docker/overlay2/5e4711091ab70078dc54bc771ee2a7c29f3cdb5213dd30eb18a86768fd245913/work)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
/dev/sda2 on /dev/termination-log type ext4 (rw,relatime)
/dev/sda2 on /etc/resolv.conf type ext4 (rw,relatime)
/dev/sda2 on /etc/hostname type ext4 (rw,relatime)
/dev/sda2 on /etc/hosts type ext4 (rw,relatime)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k,inode64)
tmpfs on /run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime,size=1911968k,inode64)

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
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.7  4.5 3156048 91408 ?       Ssl  03:03   0:04 /opt/java/openjdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
root          54  0.0  0.2   7312  4072 pts/0    Ss   03:04   0:00 bash
root          77  4.5  1.9 5846388 40092 pts/0   Sl+  03:12   0:00 ./CloudPentestSuite info all
root          87  0.0  0.0   2380   888 pts/0    S+   03:12   0:00 /bin/sh -c ps aux
root          88  0.0  0.1  10620  3884 pts/0    R+   03:12   0:00 ps aux

[网络]


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
