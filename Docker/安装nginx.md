~~~
docker pull nginx:1.12.2
~~~

~~~
# 生成容器<sub></sub>
docker run --name nginx -p 9001:80 -d nginx:1.12.2
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /docker/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /docker/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /docker/nginx/
~~~

```
# 直接执行docker rm nginx或者以容器id方式关闭容器
# 找到nginx对应的容器id
docker ps -a
# 关闭该容器
docker stop nginx
# 删除该容器
docker rm nginx
 
# 删除正在运行的nginx容器
docker rm -f nginx**
```

```
docker run \
-p 9002:80 \
--name nginx-alone \
-v /docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /docker/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /docker/nginx/log:/var/log/nginx \
-v /docker/nginx/html:/usr/share/nginx/html \
-d nginx:1.12.2
```

