## Docker 네트워크 생성
network 생성시 gateway, subnet이 다른 기존 도커 네트워크랑 겹치지 않도록 해야함(옵션 안주면 자동으로 안겹치게 생성해줌)   
e.g. sudo docker network create --gateway=172.20.0.1 --subnet=172.20.0.0/16 -o "com.docker.network.bridge.host_binding_ipv4"="0.0.0.0" -o "com.docker.network.bridge.enable_icc"="true" -o "com.docker.network.driver.mtu"="1500" -o "com.docker.network.bridge.name"="pse-network" -o "com.docker.network.bridge.enable_ip_masquerade"="true" pse-network



## Docker container 외부로 통신이 안될 경우 확인   
  1. sysctl net.ipv4.conf.all.forwarding=1   
  2. sudo iptables -P FORWARD ACCEPT   
  3. docker network에 enable_ip_masquerade true로 설정   
  4. /etc/systemd/system/docker.service.d 경로에 프록시 있는지 확인   
  5. /etc/systemd/system/docker.service.d/docker-options.conf 파일   
  6. --iptables= true로 설정   
  ![image](https://user-images.githubusercontent.com/13589283/150479643-51f7655d-464a-40c0-9865-12b24b520486.png)
