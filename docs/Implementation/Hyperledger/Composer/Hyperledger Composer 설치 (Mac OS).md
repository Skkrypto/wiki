# Hyperledger Composer 설치 (Mac OS)

## 개요
Hyperledger Composer는 Hyperledger Fabric를 지원하는 블록체인 개발도구이자 프레임워크입니다.  Playground라는 GUI를 통해 쉽게 블록체인 어플리케이션 제작이 가능하고, CLI를 통해서도 제작이 가능해 Hyperledger 진영에서 빼놓을 수 없는 필수 도구입니다. 그리고 이 도구를 설치하는 이유 중 하나로 덤으로 Fabric를 정말 쉽게 설치할 수 있다는 점이 있습니다.



## 준비 사항
- OS X 10.12.6 이상
- **Python 2.7.X (2.7.14 구동 확인)** 
- Git
- X Code
- NVM & Node.js : NVM은 Node의 버젼 관리를 위한 툴이다
- Docker: Fabric Node를 컨테이너에 담아 구동시킨다.
- VS Code: IDE로 활용, 아래의 익스텐션 때문에 사용한다.
- Hyperledger Composer VS Code Extension

### 준비 사항 설치
1. 먼저 맥 앱스토어를 통해 Xcode를 설치한다. 용량이 10GB가 넘어가므로 미리 설치해두면 좋다.

2. 아래의 명령어로 NVM을 설치한다.  공식문서에선 v0.33.0을 설치하는데 2018년 6월 현재 v0.33.11이 최신이다. 혹시 모르니 공식문서 버젼대로 설치한 후 나중에 업데이트를 해보도록 하자.
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

3. 버젼을 확인한다. v0.33.0 이 나온다면 정상
```
nvm —-version
```

4. NVM을 통해 LTS 버젼의 Node를 설치한다. 
````
nvm install --lts
```

5. LTS를 사용하는 것으로 전환한다.
```
nvm use --lts
```

6. Node 버젼을 확인한다. 앞자리수로 짝수 나오면 정상
```
node --version
```

7. Docker를 다운받아 설치한다.
[Install Docker for Mac | Docker Documentation](https://docs.docker.com/docker-for-mac/install/)

8. VS Code를 다운받아 설치한다.
[Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)

9. VS Code의 Extension 탭에서 hyperledger composer를 검색하여 나오는 최상단의 Extension을 설치한다. 

참고: [Installing pre-requisites | Hyperledger Composer](https://hyperledger.github.io/composer/latest/installing/installing-prereqs.html)




## 설치
### Step 1: Composer CLI Tool 설치

절대 Sudo를 사용하지 말것! 

```
npm install -g composer-cli
npm install -g composer-rest-server
npm install -g generator-hyperledger-composer
npm install -g yo
```

> Troubleshooting: 오류가 분명 많이 나온다.   

1. Python 3.6을 사용하여 나오는 오류
Mac의 경우 Pyenv나 Conda ENV 등 평소 사용하는 Python 버젼 관리 툴을 이용해 2.7로 환경을 바꾸어주면 편하다. 아직 설치를 하지 않았다면 Pyenv 사용을 적극 권한다.
적어도 Virtualenv까진 해주면 정말 편리하다. 

[파이썬 가상 개발 환경 구성: pyenv, virtualenv, autoenv, pip](http://taewan.kim/post/python_virtual_env/)


2. Error: c++: error: unrecognized command line option '-stdlib=libc++' installing node lib
Python 2.7로 바꾸어서 해당 오류는 나지 않지만 여전히 실패하는 경우가 있다. 오류코드를 보다가 해당 오류가 나온다면 아래와 같이 CXX=clang++ 를 앞에 붙여서 3가지를 설치해보자.

```
CXX=clang++ npm install -g composer-cli
CXX=clang++ npm install -g composer-rest-server
CXX=clang++ npm install -g generator-hyperledger-composer
```



### Step 2: Playground 설치
```
npm install -g composer-playground
```



### Step 3: Hyperledger Fabric 설치

1. fabric-dev-servers 폴더를 Home 폴더 밑에 만들고 그 안에 Fabric Server 설치를 위한 파일을 다운로드 받아 압축을 푼다.
```
mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers

curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
tar -xvf fabric-dev-servers.tar.gz
```

2. fabric-dev-servers 로 이동하여 Fabric 을 다운받는다. (Docker Pull를 함)
```
cd ~/fabric-dev-servers
./downloadFabric.sh
```



## 개발환경 Control
### Hyperledger Fabric 시작 및 중단하기

1. fabric-dev-servers로 이동하여 Fabric을 구동하고 AdminCard를 추가한다.
```
cd ~/fabric-dev-servers
./startFabric.sh
./createPeerAdminCard.sh
```

2. Fabric 서버를 확인하고 싶다면 Docker 명령어를 통해 확인이 가능하다.
`docker ps`

3. Fabric  서버를 멈출 수 있다.
```
# Fabric 서버 멈추기
cd ~/fabric-dev-servers
./stopFabric.sh

# 개발을 마친 후 Teardown을 하기 위해
./teardownFabric.sh
```


### Playground 실행하기

1. 바로 실행할 수 있다. 
`composer-playground`
2. 만약 찾을 수 없다는 내용이 뜬다면 환경 변수에 composer-playgroud가 있는 폴더를 추가해야한다. 해당 폴더는  주로 ~/.nvm/versions/해당 Version/bin 폴더이다.
3. 그 폴더를 찾아 아래 빈칸을 알맞게 채워넣고 실행시켜 bash_profile에 PATH를 추가한다.
`echo 'export PATH="/Users/[유저명]/.nvm/versions/node/[v버젼명]/bin:$PATH"' >> ~/.bash_profile`


참고: [Installing the development environment | Hyperledger Composer](https://hyperledger.github.io/composer/latest/installing/development-tools.html)




#skkrypto/hyperledger