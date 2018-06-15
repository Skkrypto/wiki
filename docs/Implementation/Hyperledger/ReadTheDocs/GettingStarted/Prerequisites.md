# 준비사항

시작하기 전에, 개발용 컴퓨터에 준비사항을 모두 설치했는지 확인해보고, 아니라면 전부 설치하자.


## cURL 설치
아직 설치를 안했거나, 혹은 cURL 명령어를 입력할 때 오류가 발생한다면 최신 [cURL](https://curl.haxx.se/download.html) 도구를 설치하자. 

```Note
만약 윈도우 환경이면 아래의 윈도우 추가 노트를 확인하라.
```

## 도커와 구성 

> Docker and Docker Compose

페브릭을 실행할, 혹은 개발할 플랫폼에 다음 사항들을 설치한다.

- 맥 OS X나 *nix, 혹은 윈10: [Docker](https://www.docker.com/get-docker)  
    - 17.06.2-ce 이상의 버전
- 윈도 구버전: [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/)
    - 마찬가지로 17.06.2-ce 이상의 버전

도커의 버전은 다음 명령어를 터미널에 입력해 확인할 수 있다.

`docker --version`

```Note
Docker나 Docker Toolbox를 설치하면 Docker Compose도 설치된다. 
이미 Docker를 설치했다면, 1.14.0 이상의 Docker Compose가 설치되어 있는지 확인해야 한다.
없다면, 최신 Docker 설치를 권한다.
```

설치한 Docker Compose의 버전은 다음 명령어로 확인할 수 있다.

`docker-compose --version`

## 고 언어

> Go Programming Language

페브릭은 여러 구성 요소에 고 언어를 사용한다.
- 1.10.x 버전 [Go](https://golang.org/dl/)가 필요하다.

고 언어로 체인 코드 프로그램을 작성하려면 두가지 환경 변수를 설정해야한다. 변경사항을 영구적으로 저장하려면 `~/.bashrc` 파일(만약 리눅스 `bash` 셸을 쓰고 있다면)과 같은 startup 파일에 두면 된다.

먼저 `GOPATH` 라는 환경 변수가 페브릭 코드가 포함된 Go workspace를 가리키도록 다음과 같이 설정해야한다.

`export GOPATH=$HOME/go`

```Note
반드시 GOPATH 변수를 설정해야 한다.

리눅스에서 GOPATH 변수는 콜론 구분 디렉토리 리스트일 수 있고 
미설정시 기본값으로 $HOME/go를 사용하지만, 
현재의 페브릭에서는 환경 변수를 설정하고 보내는 것을 필요로 한다. 
또한 Go Workspace의 이름을 반드시 하나만 갖고 있어야 한다.(추후 배포에서 제약이 삭제될 수도 있음)
```

또한 명령어 검색 경로가 Go `bin` 디렉토리를 포함하도록 해야 한다. 리눅스 `bash`에선 다음과 같다.

`export PATH=$PATH:$GOPATH/bin`

새로운 Go workspace에 해당 디렉토리가 없을 수도 있지만, 추후 페브릭 빌드 시스템에 의해 디렉토리와 여러 Go 실행파일이 추가된다. 따라서 현재 해당 디렉토리가 없더라도 셸 검색 경로를 위와 같이 확장한다.

## Node.js Runtime and NPM

페브릭 SDK for Node.js 를 사용해 페브릭 어플리케이션을 개발하고 싶다면 8.9.x 이상의 [Node.js](https://nodejs.org/en/download/)가 설치되어 있어야 한다.

```Note
현재 Node.js 9.x 는 지원되지 않는다.
```

```Note
Node.js를 설치하면 NPM도 설치되지만, 버전을 확인하고 업그레이드 하는 것을 권장한다.
다음 명령어를 입력해 업그레이드한다.
```
`npm install npm@5.6.0 -g`

## Python

```Note
다음 절은 우분투 16.04 사용자에게만 해당된다.
```

우분투 16.04는 기본적으로 파이썬 3.5.1을 파이썬3 바이너리로 갖고 있다. 그러나 페브릭 Node.js SDK는 성공적인 `npm install`을 위해 파이썬 2.7을 필요로 한다. 다음 명령어로 파이썬 2.7을 불러온다.

`sudo apt-get install python`

버전을 확인하라.

`python --version`

## 윈도우 추가 노트

만약 윈7에서 개발중이라면, [Git Bash](https://git-scm.com/downloads)가 내장된 Docker Quickstart 터미널을 사용하자.

하지만 경험적으로 이 터미널은 기능의 한계를 가진 부족한 개발환경이다. '시작하기'와 같은 Docker 기반 시나리오엔 적합하지만, `make`와 `docker` 명령어를 포함한 명령을 실행할 때 어려움을 겪을 수 있다. 

윈10에서는 기본 Docker 배포나 윈도우 파워셸을 사용할 수 있다. 그러나 `binaries` 명령어를 사용하기 위해선 `uname` 명령어를 사용할 수 있어야 한다. Git의 지원을 사용할 수도 있지만 64비트만 지원하는 것을 유념하자.

`git clone` 명령어를 실행하기 전에 다음 명령어를 실행하자.

`git config --global core.autocrlf false`
`git config --global core.longpaths true`

설정이 제대로 되었는지는 다음 명령어로 확인할 수 있다.

`git config --get core.autocrlf`
`git config --get core.longpaths`

각각 `false` 와 `true` 값이 나와야 한다.

Git과 Docker Toolbox에 내장된 `curl` 명령어는 낡아 '시작하기'에 사용된 redirect를 제대로 조작하지 못한다. [cURL 설치 페이지](https://curl.haxx.se/download.html) 에서 새로운 판을 받아 사용하자.

Node.js 사용을 위해서 필수적인 Visual Studio C++ 빌드 도구를 설치해야 한다.

`npm install --global windows-build-tools`

자세한 정보가 궁금하다면 [NPM 윈도우 빌드 도구](https://www.npmjs.com/package/windows-build-tools)를 확인하자.

여기까지 마쳤으면 NPM GRPC 모듈을 설치한다.

`npm install --global grpc`

이제 예제와 튜토리얼을 진행할 환경이 되었다.

```Note
만약 이 문서로 해결되지 않는 질문이 생기거나, 
튜토리얼 진행중에 문제가 생긴다면 아직 질문이 있습니까?를 방문해 팁과 추가적인 도움을 찾아보라.
```