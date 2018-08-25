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

5\) 만약 NFS 서버가 별도로 없을 경우에 특정 노드에 NFS Server를 설치한다.

나머지 노드에는 모두에 nfs-util를 yum으로 설치한다.

```
# yum install -y nfs-utils
# vi /etc/exports        // 해당 파일에 아래 내용 기술하여 저장. 
/nfs 203.236.100.0/24(rw,sync,no_root_squash,no_all_squash,no_subtree_check)

// rpcbind, nfs-server를 서비스에 등록 및 기동.
# systemctl enable rpcbind
# systemctl enable nfs-server
# systemctl start rpcbind
# systemctl start nfs-server
```

#### ㅤ

### 설치

**1.먼저 설치를 위해 아래와 같이 빈 디렉토리를 만든 후 해당 디렉토리로 이동한다**

```
# mkdir /Desktop/baremetal
# cd /Desktop/baremetal
```

**2.cube 명령을 이용하여 baremetal용 설치 script를 download 받는다.**

```
# cube init -p baremetal
```

**3.앞서 생성된 스크립트 중 cube.yaml 파일을 편집기로 열고, 설치하고자 하는 VM 정보를 기입한 후 저장한다.**

\( 아래는 master 2, worker 3대, nfs server로 구성하는 예임.\)

만약 외부 LoadBalancer 가용하여 master를 이중화 할 경우 lb\_ip에 해당 load balancer ip를 기재하면 됨.

```
---
cloud_provider: "baremetal"


# (required) Master node ips(comma separated).
master_ip: ["203.236.100.10","203.236.100.11"]  -> Master Node ip 기입 

# (optional) Master node private ips(comma separated).
master_private_ip: ["10.0.0.2","10.0.0.3"]  -> Master Node private ip 기입

# (required) Worker node ips(comma separated).
worker_ip: ["203.236.100.12", "203.236.100.13", "203.236.100.14"] -> Worker Node ip 기입 

# (optional) Worker node private ips(comma separated).
worker_private_ip: ["10.0.0.4","10.0.0.5","10.0.0.6"] -> 워커 서버 private ip 기입

# (required) Set true if high-availability is required.  -> haproxy 사용여부(true of false 기입 )
haproxy: false

# (conditional) Set load-balancer ip. -> LB사용 시 해당 아이피 기입하고 아닐 시 공란
lb_ip:

# (required) ssh username to access server.
ssh_user_id: "root" 

# (required) Absolute Path to an SSH private key file to access server.. -> ssh-private 키 경로 기입
private_key_path: "/path/to/ssh_private_key"

# (required) Absolute Path to an SSH public key file to be provisioned as the SSH key. -> ssh-public 키 경로 기입 
key_path: "/path/to/ssh_public_key"

# (required) Data directory for docker, kubelet, etcd, log.
data_dir: "/cocktail"

# Kubernetes
k8s_version: "1.9.10"
cluster_name: "cube"
domain_name: "acornsoft.io"
addons:
  cm: true
  km: false

# (optional) if nfs server available
nfs_ip: "203.236.100.15"  -> nfs서버의 ip 기입
nfs_mountdir: "/nfs"      -> nfs서버의 공유 디렉토리 경로 기입

# (required) Prometheus Rule Language - en, ko, jp
alertmsg_lang: ko

# (required) The unique cluster id.
cluster_id: "test-cluster"
```

**4.cube.yaml 파일이 있는 경로에서 cube create 명령을 이용하여 실제 VM에 cocktail을 설치한다.**

```
# cube create
```

**5.오류없이 설치가 완료되면 master 장비에 ssh로 접속하여 cocktail-system를 구성하는 컨테이너가 정상적으로 기동하는지 확인한다.**

```
# ssh -i ~/cube/pki/id_rsa root@203.236.100.10
# watch -n1 "kubectl get pods -n cocktail-system"

NAME                                 READY     STATUS    RESTARTS   AGE
apollomq-59fbfbb7c4-nzpls            1/1       Running   0          3d
cocktail-api-7c9bb84c46-dgw8c        1/1       Running   0          3d
cocktail-client-5869fb7975-tmckz     1/1       Running   0          3d
cocktail-cmdb-fd87c6c5f-tfzlg        1/1       Running   1          3d
cocktail-dashboard-fb6f8f5b9-zn6sj   1/1       Running   0          3d
redis-cb9c6859f-qq4f8                1/1       Running   0          3d
```

**6.브라우저로 **[**http://{VM의**](http://{VM의)** masterip}:30000으로 접속하면 cocktail login 화면으로 접속할 수 있다.**![](/assets/baremetal_login.jpeg)

### **삭제**

**1.삭제는  옵션에 따라 k8s cluster만 삭제하거나, script를 모두 정리하고 cube.yaml파일을 백업할 수 있다.**  
디폴트로 옵션을 주지 않는 경우에는 생성한 k8s cluster만 삭제하고 설치 스크립트는 그대로 유지하며, -f 옵션을 추가하면 cube.yaml 파일을 cube.yaml.org로 백업파일을 생성한 후 설치스크립트도 모두 삭제하게 된다.

따라서, 이 명령어는 주의해서 실행해야 한다.

```
# cd /Desktop/cubetest

// k8s만 삭제하고 설치 스크립트는 그대로 유지 
# cube destroy

// k8s와 설치 스크립트를 삭제하고 cube.yaml을 cube.yaml.org파일로 백업함. 
# cube destroy -f
```

-f 옵션으로 삭제한 후에 다시 설치하기 위해서 cube init 명령을 통해 설치 스크립트를 다시 다운로드 받은 후 설치하면 된다.

