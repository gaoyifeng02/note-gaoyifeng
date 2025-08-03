~~~
docker pull zookeeper:3.4.13

docker run -d --name zookeeper-alone --privileged=true -p 2181:2181  -v /docker/zookeeper/data:/data -v /docker/zookeeper/conf:/conf -v /docker/zookeeper/logs:/datalog zookeeper:3.4.13
~~~



















