# Cocktail Installation

## 사전 준비 사항

설치 전 아래와 같은 프로그램들이 미리 설치 되어 있어야 하며 설치 되어 있지 않을 경우 에러 메시지가 발생한다.

1\) Docker 다운로드 후 설치 \(필수\)

[https://store.docker.com/editions/community/docker-ce-desktop-mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)

2\) Vagrant 다운로드 후 설치 \(virtualbox를 이용한 로컬 테스트 환경 구축시\)

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)

3\) VirtualBox 다운로드 \(5.1.x version\) \(VirtualBox를 이용한 로컬 테스트 환경 구축시\)

[https://www.virtualbox.org/wiki/Download\_Old\_Builds\_5\_1](https://www.virtualbox.org/wiki/Download_Old_Builds_5_1)

4\) SSH private key & public key 생성 및 유저 등록 \(필수\)

* SSH키는 Cube가 생성한 server instance에 접속하기 위해 사용한다.

* SSH key경로는 'cube.yaml 파일'의 키 경로 항목에 지정한다. 이에 대한 자세한 내용은 각 설치 환경에 대한 문서를 참조한다.

* 새로운 키 생성은 아래를 참조한다. 기존 키 사용과 마찬가지로 생성한 키는 'cube.yaml' 파일에 지정해야 한다.

```
# ssh-keygen -t rsa
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


# ssh-add id_rsa (유저등록)
```

5\) kubectl 다운로드 \(옵션\)

[https://kubernetes.io/docs/tasks/tools/install-kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)

6\) minikube 다운로드 \(k8s minikube를 이용한 로컬 테스트 환경 구축시\)

[https://github.com/kubernetes/minikube/releases](https://github.com/kubernetes/minikube/releases)

#### 



