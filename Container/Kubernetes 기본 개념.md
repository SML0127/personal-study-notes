# 쿠버네티스?
  - Container들을 관리 및 조율(corchestraion) 해주는 tool  

## 관련 background
  a) Virtual machine: 호스트 서버와 별개의 하드웨어 스택을 가지는 가상 공간으로 자체 커널을 포함하며 hypervisor를 통해 호스트 서버와 조율된다.  
  b) Container: OS 레벨에서의 가상 공간이다. 호스트 서버의 리소스, 라이브러리등을 공유하는 가상 공간 application이다.    
  c) Docker: conatiner 이미지에 대한 생성, 관리, 배포 그리고 이미지를 이용하여 container를 띄어 주는 통합 platform으로 containerd, runc등 독립적인 프로세스들을 가진다.   
  d) Kubernetes: container들을 관리 및 조율하는 tool. docker와 호환은 되는데, 1.24 버전부터 성능상의 이유로 docker 지원 중단.    
  e) Container runtime: Container 실행을 담당하는 프로세스.(e.g. containerd, CRI-O, Docker)   



## Setup on Linux (k8s, k3s 기준)
  - Master / Worker 를 설치할 머신들에 공통적으로 설치해야할 것  

1. kubeadm   
  쿠버네티스 클러스터 생성 및 관리(부트스트랩)를 위한 커맨드 라인 툴   
  https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
  
2. kubectl   
  쿠버네티스 클러스터와의 통신 및 제어를 위한 커맨드 라인 툴   
  https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-linux/
  
3. kubelet (Master에도 필요한지 확인 필요)      
  Pod들 실행 및 관리하는 컴포넌트, 워커 노드에서는 API Server의 요청을 받아 수행    
  
4. kube-proxy (Master에도 필요한지 확인 필요)   
  클러스터 내/외부의 네트워크 설정 및 통신 관리   

## Worker에 설치

1. Containerd (Container runtime의 한 종류)   
  https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/   

 
## 그 외   

1. Master의 동작 과정   
  Client에서 API Server로의 요청이 요청, API Server는 Scheduler에게 요청. Scheduler는 Contoller Manager를 통해 etcd에 파드 실행에 필요한 정보들이 생성되었는지 확인 후 Worker의 kubelet에게 요청   
  = Client -> API Server -> Scheduler (<-> Controller Manager <-> etcd) -> kubelet   


2. Pod, Replicaset, Deployment   
  a) Job: pod들을 실행하여 어떠한 작업을 수행하고 종료시키는 것을 의미. 반복 실행은 CronJob. Job을 띄운다 = Pod를 띄우고 어떠한 작업을 수행하고 종료한다.   
  b) Pod: 클러스터에서 실행되는 컨테이너들의 집합. 가장 작은 배포 단위    
    b.1) Affinity / Anti-affinity: 파드들간의 관계를 지정하는 옵션으로 같은 노드위에 파드를 띄우고 싶다거나, 파드를 개별적인 노드 또는 다른 노드 띄워야 할 경우 설정.   
  c) Replicaset: 파드들을 생성, 관리하는 프로세스 (Replication Controller는 selector로 pod들 하나씩 관리하고, replica는 mactch label로 여러개 관리)   
  d) Deployment: 레플리카셋들과 파드들 관리하는 프로세스. 변경사항이 있을 경우 롤링 업데이트(command: roll-out) 로 새로운 파드들 생성하고 기존 파드들을 죽이며 순차적으로 업데이트 (서비스 상의 중단은 없음)   
  e) DaemonSet: 전반적인 레플리카셋, 파드들의 관리가 delopyment, 클러스터 전체에 띄우는 특정 파드들에 대한 관리는 DaemonSet   
  


3. 서비스 (https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)   
  a) cluter ip: 파드들에 할당하는 고정 ip. 파드가 업데이트되면 새로운 ip가 할당되고 이 ip를 이용할 경우 파드들 간의 네트워크 연결이 끊기니 고정 ip를 할당. (클러스터 내부)   
  b) node port: 클러스터 외부로 서비스를 노출. 클러스터 외부에서 Node ip, Node port를 이용하여 접근 가능   
  c) load balancer: 클러스터 외부에 서비스를 노출. 이걸 설정하면 node port 자동으로 설정됨.   
 
4. ingress (multiple service)   
  => 클러스터 외부로부터 서비스'들'에 대한 접근을 관리 일종의 라우터로 서비스 별로 로드밸런서 자동으로 설정해줌.    
