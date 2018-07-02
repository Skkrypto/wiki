# 시작하기

시작하기 전에, 개발용 컴퓨터에 [준비사항](#Prerequisites.md) <!--주소 가짜 주소임. 우리가 사용할 위키 사이트에 맞게 튜닝할 필요 있음-->을 모두 설치했는지 확인해보고, 아니라면 전부 설치하도록 하자.

모든 준비가 끝났다면 페브릭을 설치하자. 페브릭의 바이너리 설치는 아직 개발 중이지만, 샘플과 바이너리 및 Docker 이미지는 이 [스크립트](https://hyperledger-fabric.readthedocs.io/en/master/install.html)를 통해 받을 수 있으며, 스크립트는 Docker 이미지를 로컬 레지스트리에도 저장한다.

## 페브릭 SDKs

페브릭은 다양한 언어를 지원하기 위해 몇가지 SDK를 지원한다. 두가지 공식 배포 SDK는 Node.js와 자바(Java)다.

- [페브릭 Node SDK](https://github.com/hyperledger/fabric-sdk-node) 와 [Node SDK 문서](https://fabric-sdk-node.github.io/).
- [페브릭 자바 SDK](https://github.com/hyperledger/fabric-sdk-java).

추가적으로 아직 공식 배포는 안되었지만 다운로드해 사용해 볼 수 있는 SDK가 세개 있다.

- [페브릭 파이썬 SDK](https://github.com/hyperledger/fabric-sdk-py).
- [페브릭 고 SDK](https://github.com/hyperledger/fabric-sdk-go).
- [페브릭 REST SDK](https://github.com/hyperledger/fabric-sdk-rest).

## 페브릭 CA

페브릭은 블록체인 네트워크에서 식별정보를 설정하고 관리할 열쇠와 인증서를 생성하는 부가적인 인증기관 서비스를 제공한다. 물론 타원곡선 기반 인증서(ECDSA certificates)를 제공하는 다른 인증기관도 사용할 수 있다.
