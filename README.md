# Jennifer Kubernetes Proxy 설치가이드

컨테이너의 매트릭 정보를 연동하기 위한 Proxy 서버  
Prometheus 와 연결하여 매트릭 정보를 조회하고 있음  

# 지원환경
아래는 Jennifer Kubernetes Proxy 가 동작하는 환경이다.
Node.js 12 이상 버전 사용
> 다운로드 : [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

Proxy를 위한 포트 (기본 3300)가 열려있어야 함

Jennifer5 가 설치되어 있어야 함  
Jennifer5 가 https 를 사용하는 경우 Proxy 에도 키설정을 해주어야 함

# 다운로드

다음 명령어를 사용하여 설치 파일 다운로드  
설치파일은 `tar.gz` 와 `zip` 두개다 제공됨  
```sh
wget http://k8s.jennifersoft.com/jennifer-kubernetes-proxy-0.0.1.tar.gz
``` 

```sh
$ tar xvfz jennifer-kubernetes-proxy-0.0.1.tar.gz
$ ls 
jennifer-kubernetes-proxy-0.0.1.tar.gz jennfier-kubernetes-proxy
```

# 제니퍼 쿠버네티스 프록시 디렉토리 구성
| 디렉토리명 | 설명|
| --- | --- |
| config | 설정파일들을 포함하고 있는 디렉토리 |
| dist | 배포된 node.js 앱 |
| node_modeuls | 애플리케이션에서 사용할 의존성 모듈 |
| package.json | node.js 앱을 관리하는 package.json |
| logs | 최초 실행 후 생기는 로그 디렉토리 |

# Linux 시스템에서 제니퍼 뷰 서버의 기동 및 중지

## Proxy 서버 설정
### Proxy 서버 환경설정 
| 환경변수 | 예 |
| --- | --- |
| JENNIFER_KUBERNETES_PROXY_CONF | /etc/jennifer/conf/server_proxy.conf |

### Proxy 서버의 옵션 설정
Jennifer Kubernetes Proxy 서버를 관리하는데 필요한 기본을 설정하는 파일이다.  
Proxy 서버의 홈 디렉토리 `config/jennifer.conf` 에 존재하고 있으며 환경설정 파일에서 위치와 파일명을 변경할 수 있다. 
아래는 `jennifer.conf` 에 대한 기본적인 설정에 대한 설명이다.
| 설정 | 기본값 | 설명 |
| --- | --- | --- |
| jennifer_proxy_port | 3300 | 프록시서버가 사용할 포트. 제니퍼 뷰서버에서 연결하기 위한 포트이다 |
| ssl_listen_port | 3443 | https 를 이용하는 경우 사용할 포트이다. |
| ssl_keystore_path | ./config | https 를 이용하는 경우 사용할 인증서의 위치 |
| ssl_keystore_password | (null) | https 를 이용하는 경우 사용할 인증서의 비밀번호 |
| log_dir | ./logs | 로그파일이 저장될 위치 |
| prometheus | (null) | Proxy 서버가 접근할 프로메테우스 서버 주소 |
| prometheus_token | (null) | Prometheus 서버 접속시 사용할 토큰 정보 |

## Proxy 서버 기동
1. Proxy 서버의 홈 디렉토리로 이동한다.
    ```sh
    cd /home/jennifer/jennfier-kubernetes-proxy
    ```
2. 해당 위치에서 forever를 실행한다.
    ```sh
    ./node_modules/forever/bin/forever start -a -l /dev/null dist/bin/www
    ```
3. Proxy 서버가 정상적으로 기동되었는지 확인한다.
    ```sh
    2022-04-25 14:58:26 1 [JENNIFER_KUBERNETES_PROXY] debug: [Middleware Loaded]:TokenVerifyMiddleware
    2022-04-25 14:58:26 1 [JENNIFER_KUBERNETES_PROXY] debug: [Controller Loaded]:TestController,PodsController,NodesController
    2022-04-25 14:58:26 1 [JENNIFER_KUBERNETES_PROXY] info: Node running on [dev] mode
    2022-04-25 14:58:26 1 [JENNIFER_KUBERNETES_PROXY] info: Listening on port 3300
    2022-04-25 14:58:26 1 [JENNIFER_KUBERNETES_PROXY] info: Listening on port 3443
    ```

## Proxy 서버 중지
1. Proxy 서버의 홈 디렉토리로 이동한다.
    ```sh
    cd /home/jennifer/jennfier-kubernetes-proxy

2. Proxy 서버의 상태를 확인한다.
    ```sh
    ./node_modules/forever/bin/forever list
    info:    Forever processes running
    data:        uid  command                                              script          forever pid   id logfile   uptime
    data:    [0] f4FV /Users/youngsoo/.nvm/versions/node/v10.22.0/bin/node dist/bin/www.js 39671   39695    /dev/null 0:0:0:8.132
    ```

3. 해당 위치에서 forever 를 종료한다.
    종료는 forever 프로세스의 `uid` 혹은 `index` 를 입력하여 종료할 수 있다.
    ```sh
    ./node_modules/forever/bin/forever stop 0
    info:    Forever stopped process:
        uid  command                                              script          forever pid   id logfile   uptime
    [0] f4FV /Users/youngsoo/.nvm/versions/node/v10.22.0/bin/node dist/bin/www.js 39671   39695    /dev/null 0:0:2:11.355999999999995
    ```


# Jennifer5 대시보드 설정
1. Jennifer5 뷰서버에 Jennifer Kubernetes Proxy 주소를 등록한다.
    ```sh
    $ cd /home/jennifer/jennfier5-server/server.view
    $ vi ./conf/server_view.conf
    #Jennifer5 뷰서버가 https로 구동 중이라면 Jennifer Kubernetes Proxy 도 https로 구동해야 합니다.
    k8s_proxy_url = http://k8s.proxy.com


2. Jennifer5 뷰서버 재시작
    ```sh
    $ /home/jennifer/jennfier5-server/server.view/bin/shutdown_view.sh
    $ /home/jennifer/jennfier5-server/server.view/bin/startup_view.sh
3. 관리> 그룹 > 메뉴별 권한 설정

    > k8s_proxy_url가 추가 되었을때만 메뉴에 표시됩니다.
    > 
    관리 페이지에서 '시스템 관리자 (K8S)', '시스템리소스 (K8S)' 체크후 저장



