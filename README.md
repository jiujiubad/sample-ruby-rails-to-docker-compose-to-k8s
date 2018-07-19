# kubernetes即k8s集群

## 1. 示例项目

> 功能: 实时发送及显示消息, 消息会延迟两秒显示

### 1.1 聊天室actioncable 和 sidekiq

Aaron教程: dmxs_websocket 之 actioncable 入门 (七)_聊天室

### 1.2 redis 和 sidetiq



## 2. docker-compose: 验证dockerfile是否跑通

参考日本教程第二章: https://chopschips.net/blog/2018/05/30/docker-compose-with-rails/

### 2.1 修改的文档总共有:

```
Dockerfile
docker-compose.yml
	./bin/setup-db-start-puma
	./bin/start-sidekiq
 	./bin/wait-for
yarn.lock
lib/tasks/db.rake
.dockerenv/railsw
.dockerenv/mysql
config/database.yml
config/cable.yml
config/credentials.yml.enc
```

### 2.2 用到的命令:

```
# 修改配置后，重新开启容器
docker-compose up -d --build --force-recreate

# 查看容器日志
docker-compose logs -f puma
docker-compose -f docker-compose-preview.yml logs -f

# 获取容器列表
docker-compose ps

# 开启web页面
open http://localhost:3000/
```

### 2.3 报错解决思路:

mysql报错: name@host.com问题, 删除 /app/tmp/mysql, 然后重新开启容器

mysql、sidekiq(redis)、puma 环境变量问题: 修改 .env 文件内的环境变量, 然后重新开启容器



## 3. k8s 两种api对象入手(deployment 和 service): 用已验证的 docker-compose 改写

参考日本教程第三章: https://chopschips.net/blog/2018/05/30/kubernetes-tutorial/

### 3.1 修改的文档总共有:

```
k8s/manifests-step0/mysql-deploy.yaml
k8s/manifests-step0/mysql-svc.yaml
```

### 3.2 用到的命令:

1) kubectl的使用

```
# 安装操作k8s集群的CLI工具
brew install kubernetes-cli

# 创建和更新一个api
kubectl apply -f kube/deployments/mysql_deploy.yaml
kubectl apply -f kube/services/mysql_svc.yaml

# 检查对象的状态
kubectl get deployments,replicaset,pods,services

# 检查容器日志
kubectl logs deployments/mysql
kubectl logs pods/k8s-ansible-mysql-69b46f9cdc-sv2b4
```

2) kubernetes web ui 安装

```
# 创建仪表板UI
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

# 运行仪表板server
kubectl proxy

# 访问仪表板UI,打开左侧Overview(看到 Deployment、Pods、Relica Sets、Services 都新增了刚刚配置的api对象)
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

3) 效果如下图: 每个模块各新增了两个api对象

![](https://ws4.sinaimg.cn/large/006tNc79gy1ftfgia9aa5j31g40qo771.jpg)



## 4.k8s 四种api对象(deployment、service、configmap、secret)

参考日本教程第四章: https://chopschips.net/blog/2018/05/30/kubernetes-with-rails/#mysql-env-secret-yaml-mysql%E3%81%AE%E7%92%B0%E5%A2%83%E5%A4%89%E6%95%B0%E7%94%A8Secret

### 4.1 修改的文档总共有:

```
k8s/manifests-step1/mysql-deploy.yaml
k8s/manifests-step1/redis-deploy.yaml
k8s/manifests-step1/puma-deploy.yaml
k8s/manifests-step1/sidekiq-deploy.yaml

k8s/manifests-step1/mysql-svc.yaml
k8s/manifests-step1/redis-svc.yaml
k8s/manifests-step1/puma-svc.yaml

k8s/manifests-step1/mysql-env-cm.yaml
k8s/manifests-step1/rails-env-cm.yaml

k8s/manifests-step1/mysql-env-secret.yaml
k8s/manifests-step1/rails-env-secret.yaml

k8s/manifests-step1/Makefile
lib/tasks/sidekiq.rake
```

### 4.2 用到的命令:

1) 逐个命令测试

```
# 创建 api
cd kube
cat configmaps/*.yaml | kubectl apply -f -
cat secrets/*.yaml | kubectl apply -f -
cat deployments/*.yaml | kubectl apply -f -
cat services/*.yaml | kubectl apply -f -

# 按3.2的步骤，查看k8s web ui，打开左侧 Overview 看到 Deployment、Pods、Relica Sets、Services 各新增了四个api对象

# 查看状态
kubectl rollout status deploy demoapp-puma
```

2) 制作 Makefile

```
# 先进入 `kube` 目录
$ cd kube

# 创建命令空间
$ make kubectl-namaspace

# 应用配置好的k8s的yaml部署文件
$ make kubectl-apply

# 查看部署状态
$ make kubectl-rollout-status

# 删除所有部署配置
$ make kubectl-delete

# 本地环境一键部署
$ make test

# 一键部署
$ make all
```

3) 效果如下图: 每个模块各新增四个api对象

![](https://ws3.sinaimg.cn/large/006tNc79gy1ftfgerpbjnj31kw0t5abh.jpg)

## 4. ansible 一键部署