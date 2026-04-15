# es

~~~
docker pull elasticsearch:7.13.4
~~~


~~~
docker run -d \
--name elasticsearch-7-alone \
-e "ES_JAVA_OPTS=-Xms2048m -Xmx2048m" \
-e "discovery.type=single-node" \
-e "http.host=0.0.0.0" \
-v /docker/elasticsearch/data:/usr/share/elasticsearch/data \
-v /docker/elasticsearch/es-plugins:/usr/share/elasticsearch/plugins \
-v /docker/elasticsearch/es-logs:/usr/share/elasticsearch/logs \
-v /docker/elasticsearch/conf/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
--privileged \
-p 9200:9200 \
-p 9300:9300 \
elasticsearch:7.13.4
~~~

~~~
docker stop elasticsearch-7-alone
docker rm elasticsearch-7-alone
~~~

~~~
# 将配置文件从容器中复制到宿主机
docker cp elasticsearch-7-alone:/usr/share/elasticsearch/config/elasticsearch.yml /docker/elasticsearch/conf/elasticsearch.yml
~~~

//这里要稍等一下完全启动
~~~
# 进入容器内
docker exec -it elasticsearch-7-alone bash

# 设置elastic，apm_system，kibana，kibana_system，logstash_system，beats_system，remote_monitoring_user 这些用户的密码
bin/elasticsearch-setup-passwords interactive
~~~

所有密码都是gaoyifeng

# kibana

~~~
docker pull kibana:7.13.4
~~~

