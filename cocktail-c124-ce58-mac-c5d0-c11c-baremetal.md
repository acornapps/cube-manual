# Cocktail Installation to baremetal on macos, linux

Mac, linux 설치 PC에서 baremetal 장비에 Cocktail를 설치하는 과정은 다음과 같다.

### **사전준비**

설치 전 아래와 같은 프로그램들이 미리 설치 되어 있어야 하며 설치 되어 있지 않을 경우 에러 메시지가 발생한다.

1\) 제공받은 cube 바이너리를 어느 디렉토리에서든 사용할 수 있도록 환경변수 path 설정을 한다.

2\) Docker 다운로드 후 설치

\( 다운로드 링크 : [https://store.docker.com/search?offering=community&q=&type=edition ](https://store.docker.com/search?offering=community&q=&type=edition)\)

3\) SSH key 생성

설치 pc에서 각 서버\(baremetal 장비\)에 ssh로 접속하여 설치하기 때문에 ssh-key 생성이 필요하다. 기존에 가지고 있는 ssh-key를 사용해도 무방하다.

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

4\) ssh-key 복사

설치 pc에서 각 서버\(baremetal 장비\)에 ssh로 접속할 수 있도록 앞서 발급한 ssh Public키를 각 서버에 복사한다.

```
# ssh-copy-id -i id_rsa.pub root@ip
```

5\) NFS 설치 \(마스터, 워커 노드로 사용될 서버 모두에 nfs를 설치한다.\)

```
# yum install -y nfs-utils
```

#### ㅤ

### 설치

**1.먼저 설치를 위해 아래와 같이 빈 디렉토리를 만든 후 해당 디렉토리로 이동한다**

```
# mkdir /Desktop/cubetest
# cd /Desktop/cubetest
```

**2.cube 명령을 이용하여 baremetal용 설치 script를 download 받는다.**

```
# cube init -p baremetal
```

**3.앞서 생성된 스크립트 중 cube.yaml 파일을 편집기로 열고, 설치하고자 하는 VM 정보를 기입한 후 저장한다.**

\( 아래는 master 1, worker 3대, nfs server로 구성하는 예임.\)

만약 외부 LoadBalancer 가용하여 master를 이중화 할 경우 lb\_ip에 해당 load balancer ip를 기재하면 됨.

```
---
cloud_provider: "baremetal"


# (required) Master node ips(comma separated). Example: ["192.168.50.11", "192.168.50.12"]
master_ip: ["203.236.100.10","203.236.100.11"]  -> 마스터 서버 ip 기입 

# (required) Worker node ips(comma separated). Example: ["192.168.50.13", "192.168.50.14", "192.168.50.15"]
worker_ip: ["203.236.100.12", "203.236.100.13", "203.236.100.14"] -> 워커 서버 ip 기입 

# (required) Set true if high-availability is required.  -> haproxy 사용여부(true of false 기입 )
haproxy: false

# (conditional) Set load-balancer ip. -> LB사용 시 해당 아이피 기입하고 아닐 시 공란
lb_ip:

# (required) ssh username to access server.
ssh_user_id: "root" 

# (required) Path to an SSH private key file to access server. -> ssh-private 키 경로 기입
private_key_path: "/cubetest/id_rsa"

# (required) Path to an SSH public key file to be provisioned as the SSH key. -> ssh-public 키 경로 기입 
key_path: "/cubetest/id_rsa.pub"

# (required) Data directory for docker, kubelet, etcd, log.
data_dir: "/cocktail"

# Kubernetes  -> 기본사항으로 변경하지 않음.
k8s_version: "1.8.6"
cluster_name: "cube"
domain_name: "acornsoft.io"
addons:
  ingress: true
  monitoring: true
  logging: true

# (required) cocktail service
cocktail: true
# (optional) if nfs server available
nfs_ip: "203.236.100.15"  -> nfs서버의 ip 기입
nfs_mountdir: "/nfs"      -> nfs서버의 공유 디렉토리 경로 기입
```

**4.cube.yaml 파일이 있는 경로에서 cube deploy 명령을 이용하여 실제 VM에 cocktail을 설치한다. -v debug 옵션을 주면 설치 되는 세부 내용을 확인할 수 있다.**

```
# cube deploy -v debug
```

**5.오류없이 설치가 완료되면 master 장비에 ssh로 접속하여 cocktail-system를 구성하는 컨테이너가 정상적으로 기동하는지 확인한다.**

```
# ssh -i ~/cube/pki/id_rsa root@203.236.100.10
# watch -n1 "kubectl get pods -n cocktail-system"

NAME                                     READY     STATUS    RESTARTS   AGE
apollomq-3231363346-77ltx                1/1       Running   0          2m
builder-api-517225361-c9j86              1/1       Running   0          2m
builder-db-2825750227-fwqlz              1/1       Running   0          2m
cocktail-api-67592375-63k43              1/1       Running   1          2m
cocktail-client-4046445963-rrwhh         2/2       Running   1          2m
cocktail-cmdb-520687312-rhm8s            1/1       Running   0          2m
cocktail-metering-aws-3487533297-tw1x7   1/1       Running   0          2m
redis-3766055555-1lzmh                   1/1       Running   0          2m
```

**6.브라우저로 **[**http://{VM의**](http://{VM의)** masterip}:30000으로 접속하면 cocktail login 화면으로 접속할 수 있다. \(User Id, Password는 별도 문의\)**![](/assets/cocktail_login.jpeg)

### **삭제**

1.테스트가 완료되어 더 이상 필요하지 않을 경우에는 아래 명령으로 삭제할 수 있다.  
디폴트로 옵션을 주지 않는 경우에는 생성한 k8s cluster만 삭제하고 설치 스크립트는 그대로 유지하며, -f 옵션을 추가하면 cube.yaml 파일을 cube.yaml.org로 백업파일을 생성한 후 설치스크립트도 모두 삭제하게 된다.

따라서, 이 명령어는 주의해서 실행해야 한다.

```
# cd /Desktop/cubetest

// k8s만 삭제하고 설치 스크립트는 그대로 유지 
# cube destroy

// k8s와 설치 스크립트를 삭제하고 cube.yaml을 cube.yaml.org파일로 백업함. 
# cube destroy -f
```

2. 삭제한 후에 다시 설치하기 위해서는 cube init 명령을 통해 설치 스크립트를 다시 다운로드 받은 후 설치하면 된다.



