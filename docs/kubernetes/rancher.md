# rancher部署

### docker

#### docker安装

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
sudo yum install docker-ce docker-ce-cli containerd.io

PS:https://docs.docker.com/install/linux/docker-ce/centos/
```

#### docker调优配置

```shell
touch /etc/docker/daemon.json
cat > /etc/docker/daemon.json <<EOF
{
    "oom-score-adjust": -1000,
    "log-driver": "json-file",
    "log-opts": {
    "max-size": "100m",
    "max-file": "3"
    },
    "max-concurrent-downloads": 10,
    "max-concurrent-uploads": 10,
    "bip": "169.254.123.1/24",
    "registry-mirrors": ["https://7bezldxe.mirror.aliyuncs.com"],
     #"registry-mirrors": ["https://mirror.ccs.tencentyun.com"],
    "insecure-registries" : ["0.0.0.0/0"],
    "storage-driver": "overlay2",
    "storage-opts": [
    "overlay2.override_kernel_check=true"
    ]
}
EOF
systemctl daemon-reload && systemctl restart docker
```



#### docker-compose安装

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

ps:https://docs.docker.com/compose/install/
```

------



### 修改IPV4转发

```shell
cat >> /etc/sysctl.conf<<EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

或者
echo  "
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf

执行sudo sysctl -p 立刻生效。


```

------



### rancher集群安装

#### 辅助工具下载

kubectl

```shell
wget https://www.rancher.cn/download/kubernetes/linux-amd64-v1.15.3-kubectl
chmod +x ./linux-amd64-v1.14.1-kubectl
sudo mv ./linux-amd64-v1.14.1-kubectl /usr/local/bin/kubectl
```

helm

```shell
wget https://www.rancher.cn/download/helm/helm-v2.14.3-linux-amd64.tar.gz
tar -zxvf helm-v2.14.3-linux-amd64.tar.gz
chmod +x ./linux-amd64/helm
sudo mv ./linux-amd64/helm /usr/local/bin/helm
```

rke

```shell
rke: wget https://www.rancher.cn/download/rke/v0.2.8-rke_linux-amd64
chmod +x ./v0.2.8-rke_linux-amd64
sudo mv ./v0.2.8-rke_linux-amd64 /usr/local/bin/rke
```



#### 免密登录

> 创建rancher用户并设置免密登录

```shell
useradd -g docker rancher
sudo passwd rancher
sudo echo 'rancher ALL=(ALL) ALL' >> /etc/sudoers

ssh-keygen -t rsa 
ssh-copy-id c47
ssh-copy-id c48
ssh-copy-id c49

配置AllowTcpForwarding和PermitTunnel 为yes
sed -i 's/#AllowTcpForwarding no/AllowTcpForwarding yes/g'  /etc/ssh/sshd_config
sed -i 's/#PermitTunnel no/PermitTunnel yes/g'  /etc/ssh/sshd_config

```

#### rancher-cluster.yml

```yaml
cluster_name: tqcloud
nodes:
  - address: 192.168.60.247
    internal_address: 192.168.60.247
    user: rancher
    role: [controlplane,worker,etcd]
  - address: 192.168.60.248
    internal_address: 192.168.60.248
    user: rancher
    role: [controlplane,worker,etcd]
  - address: 192.168.60.249
    internal_address: 192.168.60.249
    user: rancher
    role: [controlplane,worker,etcd]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

```

#### 创建k8s集群

```shell
执行：rke up --config ./rancher-cluster.yml
```

#### 安装helm

```shell
--kubeconfig=kube_configxxx.yml
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller \
--clusterrole cluster-admin --serviceaccount=kube-system:tiller

kubeconfig=xxx.yml

helm_version=`helm version |grep Client | awk -F""\" '{print $2}'`
helm init --upgrade \
--service-account tiller --skip-refresh \
--tiller-image registry.cn-shanghai.aliyuncs.com/rancher/tiller:$helm_version

查看状态
kubectl -n kube-system  rollout status deploy/tiller-deploy

```

#### 安装rancher

**HA rancher**

```shell
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.hongjinpin.com \
  --set ingress.tls.source=secret
  --set replicas=1
```

**单节点rancher**

```shell
docker run -d --restart=unless-stopped \
-p 9000:80 -p 9443:443 \
-v /root/data/rancher:/var/lib/rancher \
-v /root/var/log/auditlog:/var/log/auditlog \
-e AUDIT_LEVEL=3 \
rancher/rancher
```



### k8s一些配置

##### 自动补全命令

```shell
编辑.bashrc自动补全命令
source <(kubectl completion bash)
```

##### k8s节点添加污点

```shell
kubectl taint nodes node1 cassandra:NoSchedule
```

kubectl查询污点模板

```tex
{{printf "%-50s %-12s\n" "Node" "Taint"}}
{{- range .items}}
    {{- if $taint := (index .spec "taints") }}
        {{- .metadata.name }}{{ "\t" }}
        {{- range $taint }}
            {{- .key }}={{ .value }}:{{ .effect }}{{ "\t" }}
        {{- end }}
        {{- "\n" }}
    {{- end}}
{{- end}}
kubectl get nodes -o go-template-file="./nodes-taints.tmpl"

```




