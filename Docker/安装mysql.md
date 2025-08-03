
~~~
docker pull mysql:8.4.3
~~~

~~~
docker run --name mysql8-alone \
  -v /docker/mysql8:/var/lib/mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=gaoyifeng \
  -d mysql:8.4.3
~~~

~~~
docker pull mysql:5.7
~~~

~~~
docker run --name mysql5.7-alone \
  -v /docker/mysql5.7:/var/lib/mysql \
  -p 3307:3306 \
  -e MYSQL_ROOT_PASSWORD=gaoyifeng \
  -d mysql:5.7
~~~




