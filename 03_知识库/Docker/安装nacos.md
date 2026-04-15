
~~~
docker pull nacos/nacos-server:v2.4.3
~~~


~~~
docker run --name nacos-alone -e MODE=standalone -p 8848:8848 -p 9848:9848 -p 9849:9849 -d nacos/nacos-server:v2.4.3
~~~