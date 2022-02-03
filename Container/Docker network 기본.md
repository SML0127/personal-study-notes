## Docker network?
도커는 컨테이너마다 네트워크(network driver) 설정을 할 수 있고 또 host 서버의 네트워크를 공유할 수도 있다.       
별도의 설정을 하지 않고 컨테이너를 생성하면 default bridge가 network driver로써 할당 된다.    
아래 그림의 구조 덕분에 컨테이너 내부에서는 컨테이너 내부 네트워크라는 인식 없이 설정 및 사용할 수 있으며, 호스트서버와 연결은 docker가 담당한다. 
![bridge1](https://user-images.githubusercontent.com/13589283/151919401-0c6ada8f-6c78-45e6-978a-2030b7a871a5.png)


## Docker network 종류
 - bridge : 가장 기본이 되는 network driver로, 별도의 네트워크 설정 없시 컨테이너 생성시 할당된다.(The default network driver. If you don’t specify a driver, this is the type of network you are creating. Bridge networks are usually used when your applications run in standalone containers that need to communicate)
 - host : host 서버의 네트워크를 공유하여 사용한다.(For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly)
 - none : 네트워크 설정을 하지 않는 것으로, 내부적으로 loopback ip만 있고 별도의 ip 할당은 하지 않아 외부 네트워크에서 접근을 할 수 없다. (For this container, disable all networking. Usually used in conjunction with a custom network driver. none is not available for swarm services)
 - overlay : 도커들 간의 컨테이너들 끼리 네트워크를 공유하고자 할 때 사용한다. docker ochestration tool인 swarm을 사용 할 수 있다. (The overlay network driver creates a distributed network among multiple Docker daemon hosts. Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other)


## Docker network bridge 생성
network 생성시 gateway, subnet이 다른 기존 도커 네트워크랑 겹치지 않도록 해야함(옵션 안주면 자동으로 안겹치게 생성해줌)          
~~~
sudo docker network create --gateway=172.20.0.1 --subnet=172.20.0.0/16 -o "com.docker.network.bridge.host_binding_ipv4"="0.0.0.0" -o "com.docker.network.bridge.enable_icc"="true" -o "com.docker.network.driver.mtu"="1500" -o "com.docker.network.bridge.name"="network_name" -o "com.docker.network.bridge.enable_ip_masquerade"="true" network_name
~~~


## Docker container 외부로 통신이 안될 경우 확인해야할 것들  
  1. sysctl net.ipv4.conf.all.forwarding=1       
  2. sudo iptables -P FORWARD ACCEPT       
  3. docker network에 enable_ip_masquerade true로 설정   
  4. /etc/systemd/system/docker.service.d 경로에 프록시 있는지 확인   
  5. /etc/systemd/system/docker.service.d/docker-options.conf 파일   
  6. --iptables= true로 설정   
  ![image](https://user-images.githubusercontent.com/13589283/150479643-51f7655d-464a-40c0-9865-12b24b520486.png)
