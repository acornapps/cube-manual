# Cocktail Installation on Azure\(windows\)

Azure cloud에 Cocktail를 설치하는 과정은 다음과 같다.

**사전준비**

설치 전 아래와 같은 프로그램들이 미리 설치 되어 있어야 하며 설치 되어 있지 않을 경우 에러 메시지가 발생한다.

1\) Docker 다운로드 후 설치

[https://store.docker.com/editions/community/docker-ce-desktop-windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)

2\) Hyper-v : \[제어판\] - \[windows 기능 겨기/끄기\] 에서 \[Hyper-V\] 기능을 활성화ㅤ

3\) Azure 계정 및 application 과 관련된 값들을 아래와 같이 조회해서 메모해 놓는다.

* Subscription\_Id : Azure포탈 메인페이지 상단 중간 검색창에 'subscription' 검색하여 subscriptionID값을 확인

![](/assets/섭스크립션id.PNG)

* TenantId : Home &gt; Azure Active Directory &gt; App registrations &gt; Endpoints &gt; OAUTH2.0 AUTHORIZATION에 포함된 값 중 아래 예시와 같이 x로 표시된 부분이 TenantId 임.\) 

\(EX : [https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/oauth2/authorize](https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/oauth2/authorize) \)

![](/assets/TenantId.jpeg)

* ApplicationId 등록하여  clientId 값 구하기

Home &gt; Azure Active Directory &gt; App registrations 에서  +New application registration 메뉴 선택.

Application Type을 Web app/API로 선택하고, Sign-on URL\(제공하는 서비스 URL\) [http://cocktail.acornsoft.io](http://cocktail.acornsoft.io) 로 입력

![](/assets/ClientId.jpeg)

등록된 application 상세 화면에서 Application ID가 clientId 값임.

![](/assets/ClientId2.jpeg)

* client\_secret 값 구하기

상기화면에서 우측의 Keys 메뉴를 선택하고 description\(예, cocktail\_key\), expires\(never expires\)를 입력하고 저장버튼을 클릭하면 자동으로 value값이 표시되는데 이 값이  client\_secret값임.

#### ㅤ

### **설치**

1.설치를 위해 빈 디렉토리를 만든 후 해당 디렉토리로 이동한다.

```
  # mkdir /tmp/cubetest
  # cd /tmp/cubetest
```

2.cube 명령을 이용하여 azure용 설치 script를 download 받고 초기화 한다.

```
# cube init -p azure
```

3.cube.yaml 파일을 편집기로 열어서 설치하고자 하는 Azure 정보 및 인스턴스 정보를 기입한다.

```
---
cloud_provider: "azure"

## (required) When azure is used, you need to set the following variables.
subscription_id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

client_id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

client_secret: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

tenant_id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

location: "JapanWest"  -> azure region

# (Required) It used as resource group name and prefix to all resources
# alphanumeric characters, periods, underscores, hyphens and parenthesis allowed
resource_prefix: "cube-01"

# (optional) Instance size for the master node(s). Default: Standard_F2s.  -> AZURE VM 사양(마스터) 
master_vm_size: "Standard_D2_v3"

# (optional) Instance size for the worker node(s). Default: Standard_F2s.  -> AZURE VM 사양(노드) 
worker_v m_size: "Standard_D2_v3"

# (required) The number of master nodes to be created. Example: 2  -> 마스터 노드 수
master_node_count: 1

# (required) The number of worker nodes to be created. Example: 3  -> 워커 노드 수 
worker_node_count: 2

# (required) Storage account type. Example: Standard_LRS  ->  Azure 스토리지 타입. default값 "Standard_LRS"
storage_account_type: "Standard_LRS"

# (required) Absolute path to an SSH private key file to access server.
private_key_path: "C:\\Users\\acornsoft\\.ssh\\id_rsa"

# (required) Absolute path to an SSH public key file to be provisioned as the SSH key.
key_path: "C:\\Users\\acornsoft\\.ssh\\id_rsa.pub"


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
```

상기 항목에서 private\_key\_path  와 key\_path 는 서버에 ssh key로 접속하기 위한 private key와 public key의 경로를 기입한다. 이미 존재하는 경우에는 해당 경로를 기입하면 되고, 신규로 생성할 경우에는 git을 다운로드 받아 bash shell을 실행한 후 아래 절차대로 실행하면 된다. \(git 다운로드 링크 : [https://git-for-windows.github.io/](https://git-for-windows.github.io/) \)

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

4.ssh접속을 위해 ansible.cfg파일의 remote\_user명을 cocktail로 수정한다.\(기본 값 : root\)

\(script다운로드폴더-&gt;cubescripts -&gt; ansible.cfg\)

```
[defaults]
remote_user = cocktail
sudo = yes
host_key_checking = False
fact_caching = jsonfile
fact_caching_connection = /tmp
callback_whitelist = profile_tasks
```

5.cube deploy 명령을 이용하여 cocktail을 설치한다. -v debug옵션을 주면 설치되는 세부 내용을 확인할 수 있다.

```
# cube deploy [-v debug]
```

6.오류없이 설치가 완료되면 자동으로 browser가 기동되어 k8s dashboard로 접속하게 된다.

