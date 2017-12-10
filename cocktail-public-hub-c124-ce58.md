# Cocktail Public Hub 서버 설치

웹서비스에 https 를 적용할 경우 SSL 인증서를 VeriSign 이나 Thawte, GeoTrust 등에서 인증서를 발급받아야 하지만 비용이 발생하므로 실제 운영 서버가 아니면 발급 받는데 부담이 된다.

이런 경우에, OpenSSL 을 이용하여 인증기관을 만들고 Self signed certificate 를 생성하고 SSL 인증서를 발급하는 사용한다.

> 참고. Self Signed Certificate\(SSC\)란 ?  
> 인증서\(digital certificate\)는 개인키 소유자의 공개키\(public key\)에 인증기관의 개인키로 전자서명한 데이타다. 모든 인증서는 발급기관\(CA\) 이 있어야 하나 최상위에 있는 인증기관\(root ca\)은 서명해줄 상위 인증기관이 없으므로 root ca의 개인키로 스스로의 인증서에 서명하여 최상위 인증기관 인증서를 만든다. 이렇게 스스로 서명한 ROOT CA 인증서를 Self Signed Certificate 라고 부른다.
>
> IE, FireFox, Chrome 등의 Web Browser 제작사는 VeriSign 이나 comodo 같은 유명 ROOT CA 들의 인증서를 신뢰하는 CA로 미리 등록해 놓으므로 저런 기관에서 발급된 SSL 인증서를 사용해야 browser 에서는 해당 SSL 인증서를 신뢰할수 있는데 OpenSSL 로 만든 ROOT CA와 SSL 인증서는 Browser가 모르는 기관이 발급한 인증서이므로 보안 경고를 발생시킬 것이나 테스트 사용에는 지장이 없다.
>
> ROOT CA 인증서를 Browser에 추가하여 보안 경고를 발생시키지 않으려면 [Browser 에 SSL 인증서 발급기관 추가하기](https://www.lesstif.com/pages/viewpage.action?pageId=6979614#) 를 참고하자.
>
> * CSR\(Certificate Signing Request\)은?
>
> 공개키 기반\(PKI\)은 private key\(개인키\)와 public key\(공개키\)로 이루어져 있다. 인증서라고 하는 것은 내 공개키가 맞다고 인증기관\(CA\)이 전자서명하여 주는 것이며 나와 보안 통신을 하려는 당사자는 내 인증서를 구해서 그 안에 있는 공개키를 이용하여 보안 통신을 할 수 있다.
>
> CSR 은 인증기관에 인증서 발급 요청을 하는 특별한 ASN.1 형식의 파일이며\([PKCS\#10 - RFC2986](http://tools.ietf.org/html/rfc2986)\)  그 안에는 내 공개키 정보와 사용하는 알고리즘 정보등이 들어 있다. 개인키는 외부에 유출되면 안 되므로 저런 특별한 형식의 파일을 만들어서 인증기관에 전달하여 인증서를 발급 받는다.
>
> SSL 인증서 발급시 CSR 생성은 Web Server 에서 이루어지는데 Web Server 마다 방식이 상이하여 사용자들이 CSR 생성등을 어려워하니 인증서 발급 대행 기관에서 개인키까지 생성해서 보내주고는 한다.

Cocktail Private Hub는 Harbor Registry와 Cocktail build Server를 위한 인증서 Self signed certificate를 생성하여 내부에서

Harbor와 Docker 인증서를 생성한다.

---

* Docker 설치

Cocktail Private Hub를 설치하기 위한 VM또는 machine에서 먼저 Docker를 설치한다.

```
# sudo su - root
# mkdir cocktail        // 작업 디렉토리를 하나 생성한다.
# yum install –y yum-utils
# yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum makecache fast
# yum install -y docker-ce
# systemctl enable docker
# systemctl start docker
```

* Docker-Compose 설치

Docker-Compose 설치를 위한 방법은 아래와 같다.

    # mkdir -p cocktail
    # cd cocktail
    # curl -L https://github.com/docker/compose/releases/download/1.12.0/docker-compose-`uname -s`-`uname -m` > docker-compose
    # chmod +x docker-compose
    # mv docker-compose /usr/local/bin/
    # docker-compose --version

* Harbor 설치

Harbor 압축파일 다운로드 및 압축 해제

```
# cd cocktail
# wget https://github.com/vmware/harbor/releases/download/v1.1.1-rc4/harbor-online-installer-v1.1.1-rc4.tgz
# tar -zxvf harbor-online-installer-v1.1.1-rc4.tgz
```

Harbor Self Signing Certificate 생성하기

참고 사이트 \([https://github.com/vmware/harbor/blob/master/docs/configure\_https.md](https://github.com/vmware/harbor/blob/master/docs/configure_https.md%29%29\)

```
# cd cocktail
# mkdir cert
# openssl req -newkey rsa:4096 -nodes -sha256 -keyout ca.key -x509 -days 3650 -out ca.crt
# openssl req -newkey rsa:4096 -nodes -sha256 -keyout harbor.key -out harbor.csr
# echo subjectAltName = IP:xxx.xxx.xxx.xxx > extfile.cnf
# openssl x509 -req -days 3650 -in harbor.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out harbor.crt
# cp harbor.crt ~/cocktail/cert
# cp harbor.key ~/cocktail/cert
```

Harbor 설정파일에서 아래  속성 확인 및 변경하기.

```
# vi harbor/harbor.cfg
...
hostname = 서버 ip
ui_url_protocol = https
db_password = root123
ssl_cert = /root/cocktail/cert/harbor.crt     (harbor crt 파일 경로)
ssl_cert_key = /root/cocktail/cert/harbor.key (harbor 인증서 key 파일 경로)
harbor_admin_password = C0ckt@1lAdmin
...
```

* Harbor 설치 및 기동

./prepare 스크립트를 실행해서 harbor 설정 파일들을 generation한다.

Harbor가 이미 떠 있는 경우에는 stop한 후 start 한다.

```
# ./prepare
# docker-compose down
# docker-compose up -d
```

* Docker client에서 Harbor 로그인이 정상적으로 되는지 확인하기.

Harbor 서버에서 생성한 ca.crt파일을 docker client가 인증서를 확인하는 디렉토리인 /etc/docker/certs.d/serverip/ 밑에 복사한다.

```
# mkdir -p /etc/docker/certs.d/xxx.xxx.xxx.xxx
# cp ca.crt /etc/docker/certs.d/xxx.xxx.xxx.xxx
# docker login xxx.xxx.xxx.xxx
```

* 참고 - Harbor 실행/중단/설정변경시

Harbor는 부팅시에 자동 실행되며 수동으로 기동, 종료 및 설정 변경시에 아래 명령을 실행하면 된다.

```
# cd cocktail/harbor
# docker-compose start
# docker-compose stop

설정을 변경한 경우에는 install.sh 파일을 다시 실행한다.
```

* ---

  Docker Build Server 설치.

Docker build Server는 docker상에 container로 실행되며 Cocktail Builder-api component와 통신하게 된다.

또한,  Docker Server와 Docker Client간 로컬이 아닌 원격지에서는 반드시 TLS로 통신해야 함으로 사설인증서를 생성하여 설정해 준다.

이를 편하게 설정하기 위해 "create-docker-tls.sh" 파일을 설치하고자 하는 VM 또는 machine에 upload하여 실행한다.

```
# ./create-docker-tls.sh "도메인 또는 IP"
# cd /root/.docker
# ls -al
# vi /etc/profile.d/docker.sh
  export DOCKER_CERT_PATH=/root/home/.docker -> export DOCKER_CERT_PATH=/root/.docker 로 변경

# ./docker.sh
# vi /lib/systemd/system/docker.service  

ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock --tlsverify --tlscacert=/root/.docker/ca.pem --tlscert=/root/.docker/server-cert.pem --tlskey=/root/.docker/server-key.pem

# daemin-reload
# systemctl restart docker
```

* k8s node에서 harbor로 부터 image를 pull하기 위해 harbor 설치시에 생성한 ca.crt파일을 각  node의 /etc/docker/certs.d/serverip/ 밑에 복사한다.

```
mkdir -p /etc/docker/certs.d/xxx.xxx.xxx.xxx
이후 ca.crt를 복사한다.
```

* Cocktail에서 builder-api, client-api deployment update

k8s dashboard에서 builder-api의 환경변수 REGISTRY_URL, DOCKER\_URL, SERVER\_TYPE, CA\_PEM, CERT\_PEM, KEY\_PEM 를 수정하고, cocktail-api의 환경변수 REGISTRY\_URL을 수정한다._

| 환경변수 | 값 예시 |
| :--- | :--- |
| REGISTRY\_URL | xxx.xxx.xxx.xxx |
| DOCKER\_URL | [https://xxx.xxx.xxx.xxx1](https://172.10.1.1) |
| SERVER\_TYPE | 임의의 string |
| CA\_PEM | build server 인증키 ca.pem파일을 base64 encoding해서  입력 |
| CERT\_PEM | build server cert.pem파일을 base64 encoding해서 입력 |
| KEY\_PEM | build server key.pem파일을 base64 encoding해서 입력 |

| 환경변수 | 값 예시 |
| :--- | :--- |
| REGISTRY\_URL | [https://](https://172.10.1.1)xxx.xxx.xxx.xxx |
| REGISTRY\_USER | harbor login id \(admin\) |
| REGISTRY\_PASSWORD | harbor login passwd \(C0ckt@1!Admin\) |


