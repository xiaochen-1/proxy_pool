# Proxy_Pool 代理池项目快速搭建

## 环境准备
* windows10（支持wsl2）
* docker

## 项目搭建

### Redis 安装
1.  利用docker安装redis
```bash
docker pull redis
```

2.  启动redis

```bash
docker run -d -p 6379:6379 --name redis redis
```

4.下载proxy_pool
```
docker pull jhao104/proxy_pool

docker run --env DB_CONN=redis://:123456@172.19.0.2:6379/0 -p 5010:5010 jhao104/proxy_pool:2.4.0
```

### 可能遇见的问题
#### 1.无法连接redis
查看docker容器的内网ip(name/id需要替换为指定容器)
```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}} name/id'
```
然后将输出的redis的内网ip放在proxy_pool启动时的DB_CONN中

#### 2.redis密码没设置
```bash
#启动并配置#
docker run --name redis -p 6379:6379 redis --requirepass 123456

#为现有redis配置密码#
docker exec -it redis redis-cli
查看现有的redis密码：config get requirepass
设置redis密码：config set requirepass ****（****为你要设置的密码）
若出现(error) NOAUTH Authentication required.错误，则使用 auth 密码 来认证密码
```
#### 3.使用--net=host 运行docker容器 主机无法访问容器
将proxy_pool项目重新运行，不能使用-net=host 或 127.0.0.1，需要获取redis内网地址（详见问题1）然后将ip地址更换为redis内网地址，redis正常连接项目在本地即可访问
#### 4.将容器挂在同一个网段下
* bridge：多由于独立container之间的通信
* host： 直接使用宿主机的网络，端口也使用宿主机的
* overlay：当有多个docker主机时，跨主机的container通信
* macvlan：每个container都有一个虚拟的MAC地址
* none: 禁用网络
创建默认bridge网络
```
docker network create default_network
```
查看网络
```
docker network list
```
docker run 时添加 --network default_network
