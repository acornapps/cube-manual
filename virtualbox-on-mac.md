# Cocktail Installation to Virtualbox on Mac OS X

MacOS에서 VirtualBox를 이용하여 Cocktail이 제공하는 기능을 간단하게 확인하고자 할 경우에 아래의 절차를 따라 설치한다.

### **사전준비**

사전 준비는 [Cocktail Installation](/cocktail-c124-ce58-mac-c5d0-c11c-virtualbox-d65c-c6a9.md)을 참조한다.

설치 전 아래와 같은 프로그램들이 미리 설치 되어 있어야 하며 설치 되어 있지 않을 경우 에러 메시지가 발생한다.

1\) Docker 다운로드 후 설치

[https://store.docker.com/editions/community/docker-ce-desktop-mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)

2\) Vagrant 다운로드 후 설치

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)

3\) Virtualbox 다운로드 \(5.1.x version\)

[https://www.virtualbox.org/wiki/Download\_Old\_Builds\_5\_1](https://www.virtualbox.org/wiki/Download_Old_Builds_5_1)

4\) SSH private key & public key 생성 및 유저 등록

키 생성의 경우 아래의 항목에서 상세하게 확인할 수 있다.

```
# ssh-keygen -t rsa
# ssh-add id_rsa (유저등록)
```

#### ㅤ

**1.먼저 설치를 위해 빈 디렉토리를 만든 후 해당 디렉토리로 이동한다**

```
# mkdir /Desktop/virtualbox
# cd /Desktop/virtualbox
```

**2.cube 명령을 이용하여 virtualbox용 설치 script를 download 받고 초기화 한다.**

```
# cube init -p virtualbox
```

**3.cube.yam 파일을 편집기로 열어서 설치하고자 하는 VM 정보를 기입한다. 아래는 예시임.**

```
# vi cube.yaml
---
cloud_provider: "virtualbox"


# (required) Master node ips(comma separated).
master_ip: ["192.168.50.10"]

# (required) Worker node ips(comma separated).
worker_ip: ["192.168.50.20", "192.168.50.100"]

# (required) Master node vcpu number
master_cpus: 4

# (required) Master node memory size (MB)
master_memory: 4096

# (required) Worker node vcpu number
worker_cpus: 2

# (required) Worker node memory size (MB)
worker_memory: 2048


# (required) Set true if haproxy is required.
haproxy: false

# (conditional) Set load-balancer ip.
lb_ip:

# (required) Absolute Path to an SSH private key file to access server.
private_key_path: "/path/to/ssh_private_key"

# (required) Absolute Path to an SSH public key file to be provisioned as the SSH key.
key_path: "/path/to/ssh_public_key"

# Kubernetes
k8s_version: "1.9.10"
cluster_name: "cube"
domain_name: "acornsoft.io"
addons:
  cm: true
  km: false

# (optional) if nfs server available
nfs_ip: "192.168.50.10"  -> nfs서버의 ip 기입
nfs_mountdir: "/nfs"      -> nfs서버의 공유 디렉토리 경로 기입

# (required) Prometheus Rule Language - en, ko, jp
alertmsg_lang: ko

# (required) The unique cluster id.
cluster_id: "test-cluster"
```

상기 항목에서 private\_key\_path  와 key\_path 는 각각 VM에 ssh key로 접속하기 위한 **private key와 public key로 반드시 절대 경로로 기입**한다. 이미 존재하는 경우에는 해당 경로를 기입하면 되고, 신규로 생성할 경우에는 아래 절차대로 실행하면 된다.

**&lt; ssh key 신규 발급 방법 &gt;**

```
# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/cloud/.ssh/id_rsa): /Desktop/cubetest/id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/cloud/git/cubedeploy/virtualbox/5/id_rsa.
Your public key has been saved in /Users/cloud/git/cubedeploy/virtualbox/5/id_rsa.pub.
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

**4.cube create 명령을 이용하여 실제 VM에 cocktail을 설치한다.**

```
# cube create
```

**5.오류없이 설치가 완료되면 master 장비에 ssh로 접속하여 cocktail-system을  구성하는 컨테이너가 정상적으로 기동하는지 확인한다.**

```
# ssh -i ~/cube/pki/id_rsa root@192.168.50.11
# watch -n1 "kubectl get pods -n cocktail-system"
NAME                                     READY     STATUS    RESTARTS   AGE
apollomq-3231363346-77ltx                1/1       Running   0          2m
cocktail-api-67592375-63k43              1/1       Running   1          2m
cocktail-client-4046445963-rrwhh         2/2       Running   1          2m
cocktail-cmdb-520687312-rhm8s            1/1       Running   0          2m
cocktail-metering-aws-3487533297-tw1x7   1/1       Running   0          2m
redis-3766055555-1lzmh                   1/1       Running   0          2m
```

**6.브라우저로 **[http://master\_IP:30000으로](http://master_IP:30000으로)** 접속하면 cocktail login 화면으로 접속할 수 있다. \(User Id, Password는 별도 문의\)**![](/assets/cocktail_login.jpeg)

### **삭제**

**1.삭제는  옵션에 따라 k8s cluster만 삭제하는 경우와 생성된  VM을 삭제할 수 있다.**  
디폴트로 옵션을 주지 않는 경우에는 생성한 k8s cluster만 삭제하고 설치 스크립트는 그대로 유지하며, -f 옵션을 추가하면 생성된 VM을 삭제하고 cube.yaml 파일을 cube.yaml.org로 백업파일을 생성한 후 설치스크립트도 모두 삭제하게 된다.

따라서, 이 명령어는 주의해서 실행해야 한다.

```
# cd /Desktop/cubetest

// k8s만 삭제하고 설치 스크립트는 그대로 유지 
# cube destroy

// k8s와 설치 스크립트를 삭제하고 cube.yaml을 cube.yaml.org파일로 백업함. 
# cube destroy -f
```

-f 옵션으로 삭제한 후에 다시 설치하기 위해서 cube init 명령을 통해 설치 스크립트를 다시 다운로드 받은 후 설치하면 된다.

