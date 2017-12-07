# Cocktail Installation on baremetal\(MAC OS\)

Mac 설치 PC에서 baremetal 장비에 Cocktail를 설치하는 과정은 다음과 같다.

### **사전준비**

설치 전 아래와 같은 프로그램들이 미리 설치 되어 있어야 하며 설치 되어 있지 않을 경우 에러 메지가 발생한다.

\(에러 메시지에대한 해결책은 문서 마지막 TroubleShooting 에서 확인할 수 있다.\)

1\) Docker 다운로드 후 설치

[https://store.docker.com/editions/community/docker-ce-desktop-mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)

2\) Vagrant 다운로드 후 설치

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)

3\) SSH private key & public key 복사

키 복사의 경우 아래의 항목에서 상세하게 확인할 수 있다.

```
# ssh-copy-id -i id_rsa.pub root@ip
```

4\) NFS 설치

```
# yum install -y nfs
```

#### ㅤ

1.먼저 설치를 위해 빈 디렉토리를 만든 후 해당 디렉토리로 이동한다

```
# mkdir /Desktop/cubetest
# cd /Desktop/cubetest
```

2.cube 명령을 이용하여 baremetal용 설치 script를 download 받고 초기화 한다.

```
# cube init -p baremetal
```

3.cube.yaml 파일을 편집기로 열어서 설치하고자 하는 VM 정보를 기입한다. 아래는 master 1ea, worker 3ea, nfs server로 구성하는 예임.

만약 외부 LoadBalancer가 가용하여 master를 이중화 할 경우 ib\_ip에 해당 load balancer ip를 기재하면 됨.

```
---
cloud_provider: "baremetal"


# (required) Master node ips(comma separated). Example: ["192.168.50.11", "192.168.50.12"]
master_ip: ["203.236.100.10"]  -> 마스터 ip

# (required) Worker node ips(comma separated). Example: ["192.168.50.13", "192.168.50.14", "192.168.50.15"]
worker_ip: ["203.236.100.12", "203.236.100.13", "203.236.100.14"] -> 워커 ip 

# (required) Set true if high-availability is required.  -> haproxy 사용여부(true of false)
haproxy: false

# (conditional) Set load-balancer ip. -> LB사용 시 해당 아이피 입력
lb_ip:

# (required) Path to an SSH private key file to access server.
private_key_path: "/cubetest/id_rsa"

# (required) Path to an SSH public key file to be provisioned as the SSH key.
key_path: "/cubetest/id_rsa.pub"


# Kubernetes
k8s_version: "1.6.7"
cluster_name: "cube"
domain_name: "acornsoft.io"
addons:
  ingress: true
  monitoring: true
  logging: true

# (required) cocktail service
cocktail: true
# (optional) if nfs server available
nfs_ip: "203.236.100.15"  -> nfs서버 ip
nfs_mountdir: "/nfs"      -> nfs서버 경로
```

상기 항목에서 private\_key\_path  와 key\_path 는 Baremetal 장비에 ssh key로 접속하기 위한 private key와 public key의 경로를 기입한다. 이미 존재하는 경우에는 해당 경로를 기입하면 되고, 신규로 생성할 경우에는 아래 절차대로 실행하면 된다.

&lt; ssh key 신규 발급 방법 &gt;

```
# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/cloud/.ssh/id_rsa): /tmp/cubetest/id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /tmp/cubetest/id_rsa.
Your public key has been saved in /tmp/cubetest/id_rsa.pub.
The key fingerprint is:
SHA256:liTKyW/l3eU9+mBzyksL0AKpYXRsvsQ793nWJiUgJC0 cloud@Clouds-MacBook-Pro.local
The key's randomart image is:
+---[RSA 2048]----+
|     ....        |
|    . .E.o       |
|     o=o=        |
|   o.oo*.o..     |
|    =.. So... .  |
|     . B oo. + o |
|      o + o.o==o.|
|     .     o=+Bo.|
|            o*=. |
+----[SHA256]-----+
```

1. Baremetal 서버에 ssh private key로 자동 접속을 위해 ssh-copy-id 명령을 사용하여 ssh public key를 해당 서버에 복사한다.

```
# ssh-copy-id -i /tmp/cubetest/id_rsa.pub root@ip
```

4.cube deploy 명령을 이용하여 실제 VM에 cocktail을 설치한다. -v debug옵션을 주면 설치되는 세부 내용을 확인할 수 있다.

```
# cube deploy [-v debug]
```

5.오류없이 설치가 완료되면 자동으로 browser가 기동되어 k8s dashboard로 접속하게 된다.

이때,  Loadbalancer가 활성화 되어 있으면, [https://lb\_ip:6443/ui](https://lb_ip:6443/ui) 로 접속하게 되고 아니면 [https://master1\_ip:6443/ui](https://master1_ip:6443/ui) 로 접속한다.

![](/assets/k8s_dashboard_1.jpeg)

고급 link를 클릭하고 아래 이동 버튼을 클릭한다.

![](/assets/k8s_dashboard_2.jpeg)

사용자이름과 비밀번호를 입력하면 k8s dashboard로 접속할 수 있다.

Namespace를 cocktail-system으로 선택하고 Services메뉴에서 cocktail-client-node-port를 선택한다

![](/assets/k8s_dashboard_4.jpeg)

connection의 internal endpoints에서 cocktail client의 node port를 확인한다. 아래 예에서는 31876 port임.

![](/assets/k8s_dashboard_5.jpeg)

브라우저로 [http://{lb\_ip](http://{VM의)}:31876 또는 [http://{master1\_ip](http://{VM의)}:31876으로 접속하면 cocktail login 화면으로 접속할 수 있다. \(User Id, Password는 별도 문의

6.프로바이더 등록

프로바이더는 클라우드 리소스로 사용 할 Public/Private 클라우드 계정 정보를 등록, 편집, 삭제할 수 있다.

프로바이더 계정을 등록함으로써 해당 프로바이더의 리소스, 미터링 정보를 가져 오거나 서버를 생성 할 수 있다. 단 로컬 테스트 환경에서는 클러스터를 생성하기 위한 정보로만 사용됨.

프로바이더는 Onpremise, 유형은 User로 선택하고 생성버튼을 클릭한다.

![](/assets/cocktail_conf_provider.jpeg)

| **프로바이더** | **설명** |
| :--- | :--- |
| Amazon Web Service | Amazon사의 public 클라우드 서비스 |
| Google Cloud Platform | Google사의 public클라우스 서비스 |
| Microsoft cloud Service | Microsoft사의 public 클라우드 서비스 |
| Rovius Cloud | Rovius사의 private 클라우드 서비스 |
| Onpremise | Baremetal환경의 private 클라우드 서비스 |

| **유형** | **설명** |
| :--- | :--- |
| User | 프로바이더 등록을 위한 기본 값. 미터링 아닌 경우 User 선택 |
| Metering | Public Cloud를 사용시, 리소스 사용량을 받아오고자 할 때 선택 |

7.클러스터 등록

생성된 클러스터 정보를 기입한다.

마스터 URL은 [https://{lb\_\_ip}:6443](https://{lb__ip}:6443) or [https://{master1ip\_}:6443](https://{master1ip_}:6443) 로 기재

![](/assets/cocktail_conf_cluster_baremetal.jpeg)

모니터링 호스트, ingress host는 lb\_ip or master1\_ip로 기재.![](/assets/스크린샷 2017-12-04 오후 1.12.49.png)Clustngress hoster CA Certification 값은 아래 결과값을 넣어주면 되고,

```
# cat /etc/kubernetes/pki/ca.pem
```

Certificate Certificate Data 값은 아래 명령을 실행한 결과값을 넣어주면 된다.

```
# cat /etc/kubernetes/pki/apiserver.pem
```

Certificate Authority Data 값은 아래 명령을 실행한 결과값을 넣어주면 된다.

```
# cat /etc/kubernetes/pki/apiserver-key.pem
```

8.볼륨 설정하기

서비스에서 사용되는 Volume을 등록한다. 입력값은 아래 이미지와 같은 값으로 등록하면 된다. \(주의. 스토리지클래스 이름은 반드시 "cocktail-nfs"라고 기입해야 한다.\)

![](/assets/cocktail_volume.jpeg)

9.서비스 생성하기

아래와 같이 간단한 tomcat server를 deploy 하고 완료시 해당 service port로 접속하여 정상적으로 tomcat 초기 화면이 표시되는지 확인한다.

![](/assets/deploy1.jpeg)

![](/assets/deploy2.jpeg)

![](/assets/deploy3.jpeg)

![](/assets/deploy4.jpeg)

![](/assets/deploy5.jpeg)

![](/assets/deploy6.jpeg)

![](/assets/deploy7.jpeg)

![](/assets/deploy11.jpeg)

![](/assets/deploy8.jpeg)![](/assets/deploy9.jpeg)![](/assets/deploy10.jpeg)![](/assets/deploy12.jpeg)**TroubleShooting**

**1.Docker가 설치되어 있지 않은 경우**

```
MinHoui-MacBook-Pro:cubetest minhona$ cube init -p baremetal
Current Working directory : /Users/minhona/Desktop/cubetest
Checking pre-requisition [darwin]
exec: "docker": executable file not found in $PATH
docker is not found. please install docker before proceeding
Visit https://store.docker.com/editions/community/docker-ce-desktop-mac
```

다운로드 링크로 이동하여 Docker 설치 후 cube를 재실행 한다.

**2.local-**_**hostname과 ansible-hostname이 다른 경우**_

```
TASK [addon : assign master roles]
Monday 30 October 2017  10:55:37 +0900 (0:00:00.042)     0:02:29.762
failed: [192.168.50.11] (item=master1) => ["changed": true, "cmd": "kubectl label nodes master1 
role=master1 --overwrite", "delta":"0:00:00.117665", "end": "2017-10-30 10:54:30.111133", "failed":
true, "item": "master1", "rc" 1, "start": "2017-10-30 10:54:29.993468", "stderr": Error from server
(NotFound): nodes \"master1\" not found", "stderr_lines": ["Error from server (NotFound): nodes \
"master1\" not found"]
```

서버에서 직접 hostname을 검색하고 그 값을 \[cubescripts-&gt;roles-&gt;addon-&gt;tasks-&gt;main.yml\]의 with\_items 값을 수정하고 재실행합니다.

```
name: assign master roles
  when: kube_dash and cloud_provider != "aws"
  shell: "kubectl label nodes {{ item }} role=master --overwrite"
  with_items: "{{ ansible_hostname|lower }}"
  register: assign_master_roles
```



