### KubeQuick可视化快速安装K8s集群及云原生服务

##### 原文地址: https://zhuanlan.zhihu.com/p/639099492


#### KubeQuick可视化安装K8s及组件服务, 目前支持的组件: Kubernetes、Prometheus

后续将会继承更多云原生组件服务, 快速安装部署

#### 适合人群: 想要入门K8s及云计算的学习人员

相信大家在按照K8s官方文档安装K8s时, 经常会碰到各种问题, 这让想要学习K8s的小伙伴很是头大, 本人也是在安装K8s的路上一步一个坑的走了过来, 为了后续的快速安装学习, 现在通过可视化方式安装K8s集群, 此项目使用了睿云智合Breeze开源项目的界面设计, 安装K8s集群的底层使用开源项目kubespray

### 一、 KubeQuick项目节点结构图

![Alt text](https://pic3.zhimg.com/80/v2-b42b5912a9cc7ca3d488a7bd0f1fc576_1440w.webp)

#### 1. KubeQuick部署机环境要求:
* 操作系统: 目前支持CentOS 7 和 Windows 系统
* 部署机需要docker 和docker-compose环境
#### 2. Kubernetes集群节点环境要求:
* 操作系统: 兼容CentOS 以及 Ubuntu 操作系统

### 二、安装KubeQuick
需要注意： 部署机为CentOS系统时， 需要取消SELINUX设定及放开防火墙

```
# 关闭selinux防火墙
setenforce 0     
sed --follow-symlinks -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config 
# 防火墙设定为可信任模式，放开所有访问策略
firewall-cmd --set-default-zone=trusted 
# 更新防火墙规则
firewall-cmd --complete-reload
```

#### 1. docker-compose.yml文件
需要注意： 部署机为Windows系统时， 更改变量$HOME的值为自己的本地地址， 例如： C:\Users\yuner

```
vi docker-compose.yml

---
version: '2'
services:
  deploy:
    container_name: deploy-main
    image: registry.cn-beijing.aliyuncs.com/kubequick/pagoda:v1.3.0
    restart: always
    entrypoint: sh
    command:
    - -c
    - "/root/pagoda -logtostderr -v 4 -w /workspace"
    ports:
    - 88:80
    - 8088:8080
    volumes:
    - $HOME/.ssh:/root/.ssh
    - $PWD/deploy:/deploy
    volumes_from:
    - playbook
  ui:
    container_name: deploy-ui
    image: registry.cn-beijing.aliyuncs.com/kubequick/deploy-ui:v2.0.0
    restart: always
    network_mode: "service:deploy"
  playbook:
    container_name: deploy-playbook
    image: registry.cn-beijing.aliyuncs.com/kubequick/playbook:v2.0.0
    volumes:
    - playbook:/workspace
volumes:
  playbook:
    external: false
```

![Alt text](https://pic1.zhimg.com/80/v2-52364087359a27f6fa24bea22ee0bc40_1440w.webp)

#### 2. 开始部署KubeQuick服务
```
docker-compose up -d
```
![Alt text](https://pic2.zhimg.com/80/v2-f0890ac1fb69594c99ebca2eae26a241_1440w.webp)


#### 3. KubeQuick安装成功, 浏览器访问: http://{{ IP }}:88

![Alt text](https://pic4.zhimg.com/80/v2-89e2568b5f1b400d7ae09b9b102d4113_1440w.webp)


### 三、开始安装K8s集群
#### 1. 开始 》 添加集群

![Alt text](https://pic4.zhimg.com/80/v2-cd2b057a96b057a8751e504395052c27_1440w.webp)
#### 2. 集群 》 添加主机
![Alt text](https://pic2.zhimg.com/80/v2-f67260f66f998064d93e5072db75a3f9_1440w.webp)

![Alt text](https://pic4.zhimg.com/80/v2-071f19fec3ec46e900bcd61a7855236b_1440w.webp)

#### 3. 为K8s集群节点间建立互信
K8s集群节点环境准备:

* 暂不支持非root账号的环境部署，因此请确保部署机到各个服务器节点是直接root ssh免密的, 所以需要注意的是Ubuntu系统需要配置root登录
* K8s集群节点是CentOS系统时, 检查环境配置是否充分
* K8s集群节点是Ubuntu系统时, 检查环境配置是否充分

```
### Windows部署机建立互信， 参考这里

### 以CentOS部署机为例， 建立部署机与K8s集群节点的互信

## 在部署机执行, 生成密钥
ssh-keygen -t rsa
## 公钥复制到K8s集群的所有节点
ssh-copy-id 192.168.110.5
ssh-copy-id 192.168.110.160

---

## 在k8s Master节点执行, 生成密钥
ssh-keygen -t rsa
## 公钥复制到集群中所有节点(包括本机)
ssh-copy-id 192.168.110.5
ssh-copy-id 192.168.110.160
```
#### 4. 集群 》 添加组件服务
![Alt text](https://pic3.zhimg.com/80/v2-3b7194cffd166f1e777113b5ede738fa_1440w.webp)

#### 5. K8s配置:
![Alt text](https://pic3.zhimg.com/80/v2-46b0173c70ee5e9046e6fec8e11b9122_1440w.webp)

#### 6. 开始安装:
![Alt text](https://pic2.zhimg.com/80/v2-67fe24184c94ae4c708e8e8fb62c1e61_1440w.webp)
![Alt text](https://pic2.zhimg.com/80/v2-ce69049b4a8c8b3796b9c41038a9f99d_1440w.webp)

![Alt text](https://pic1.zhimg.com/80/v2-e679492299cbb9a16fb075f4c8b3bb28_1440w.webp)



#### 7. 安装成功, 大功告成, 开始你的K8s之旅吧 ! ! !


---


### 四、随笔记录:
#### 1. 查看日志:
```
docker logs -f deploy-main
```
#### 2. 注意事项
请不要把KubeQuick部署机加入部署集群主机列表