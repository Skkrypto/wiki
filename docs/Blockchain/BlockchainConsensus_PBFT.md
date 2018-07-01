<이 글은 작성자가 '케블리' 활동으로 작성한 글입니다.>

# [케블리] #48. PBFT 합의 알고리즘 이해하기

![consensus](https://user-images.githubusercontent.com/26548454/42135259-83122f5a-7d83-11e8-8499-f5df2b7848d4.jpg)
\[그림1 - Consensus]

케블리 #44 글에서 ‘합의 알고리즘 마스터하기 - PoW, PoS’를 다루었습니다. (<https://steemit.com/kr/@kblock/44-1-pow-pos>) PoW와 PoS 모두 블록체인이라는 분산 원장 시스템에서 노드 간 합의를 이루는 새로운 방법입니다. 특히 비트코인이 제시한 합의 알고리즘은 ‘Nakamoto Consensus’라고도 불릴 만큼, 종전 컴퓨터과학의 합의 알고리즘에서는 시도한 적 없던 방법이었습니다.


하지만 블록체인 기술이 나타나기 전에도 분산컴퓨팅 기술은 있었고, 분산 환경에서 쓰이는 합의 알고리즘이 있습니다. 비잔틴 장군 문제(Byzantine General Problem) - 네트워크 내에 배신자가 있더라도 합의 내용에 문제가 없으려면 어떻게 해야 하는가 - 를 해결하는 방안으로 제시된 비잔틴 장애 허용(Byzantine Fault Tolerance)과, 비잔틴 장군 문제가 발생할 수 있는 상황에서도 네트워크의 합의를 보장하는 **PBFT(Practical Byzantine Fault Tolerance)** 알고리즘을 소개하고자 합니다.
<bR>
<br>

## PBFT 이해를 위한 사전지식 - FLP Impossibility

어떤 합의 알고리즘이 네트워크에서 통용되기 위해선 **Safety**와 **Liveness**라는 특성을 가지고 있어야 합니다. Safety의 의미는 “문제 없는 노드들 사이에서는 잘못된 합의가 이루어지지 않는다”입니다. 실용적인 의미로는 ‘노드 간 합의가 발생했다면, 어느 노드가 접근하든 그 값은 동일해야 한다’로 이해해도 좋습니다. Liveness는 “Transaction과 같은 합의 대상에 문제가 없다면 반드시 합의가 이루어진다”라는 의미입니다. 
<Br>


> 그동안 컴퓨터과학의 분산환경에서 합의 알고리즘을 설계할 때엔, Safety를 Liveness보다 우선하도록 알고리즘을 설계해오곤 했습니다. 설령 네트워크 내에 있는 모든 거래를 전부 합의하지는 못하더라도, 일단 합의된 거래는 네트워크 내에 있는 모두에게 동일한 값으로 제공되는 것이 더 중요하다고 보았기 때문이죠. 그래서 과거의 합의 알고리즘의 관심사는 ‘Safety’를 충족하는 상황에서 어떻게 하면 ‘Liveness’의 손실을 최소화할지 고민했습니다.
>
> 비트코인의 합의 알고리즘이 주목받은 이유는, Safety와 Liveness를 대하는 컴퓨터과학의 기본 전제를 뒤틀었기 때문이었습니다. 보통 컴퓨터과학에서의 합의 알고리즘은 Safety를 확보한 상태에서 Liveness의 손실을 최소화하는 방식으로 알고리즘을 설계해 왔습니다. 그런데 비트코인은 모든 거래를 가감없이 수용하는 구조를 채택함으로써 Liveness를 극대화하되, 하드포크가 발생할 경우 “최장 길이 체인 선택 알고리즘”을 활용해 Safety 문제를 보완했습니다. 그리고 블록이 쌓일수록 변경이 사실상 불가능해지는 구조를 활용해 ‘확률적으로’ Safety를, 블록체인의 용어로는 Finality를 확보했습니다. 
>
> 하지만, 어디까지나 Safety를 ‘확률적으로’ 확보하는 것이기 때문에 51% 공격과 같은 네트워크 공격이 성공할 경우 Safety를 보장할 수 없다는 단점을 안고 있습니다. 비트코인은 블록의 Finality를 확률적으로 보장한다는 의미와 동일한 맥락입니다. 
<br>


그런데, 비동기 네트워크 내에서는 Safety와 Liveness를 모두 완벽히 만족하는 합의 알고리즘을 설계하는 것이 불가능하다는 것이 증명되었습니다. 이 증명을 **“FLP Impossibility”**라고 합니다. 정확히 말하면, 비동기 네트워크에서는 합의 문제를 완벽히 해결할 수 있는 분산 알고리즘이 없다는 것을 증명했습니다. 1985년 4월 발표된 논문 ‘Impossibility of Distributed Consensus with One Faulty Process’에서 언급된 내용입니다. 비동기 네트워크에서는 어떤 한 노드에서 문제가 발생했을 경우 그 노드에서 합의가 됐는데 단순히 응답에 오랜 시간이 걸리는 건지, 아니면 합의 과정에서 충돌이 발생해서 응답하지 않는 건지 알 수 없기 때문입니다.
<br>
<Br>

> ### 비동기 네트워크란??
>
> 네트워크는, 네트워크 내에서 주어진 프로토콜의 실행 라운드에서 메시지를 교환할 때 발생하는 대기 시간의 여부에 따라 크게 세 가지로 나뉩니다.
>
> - 동기 네트워크(Synchronous Network) : 노드가 메시지를 보내는 시간과 수신 노드가 메시지를 수신하는 시간 사이에 상한선이 존재하고, 그 시간 안에 반드시 메시지가 수신 노드에게 도착한다는 것을 보증하는 네트워크. 따라서 네트워크 사이에 데이터 통신이 ‘반드시’ 이루어져야 하는 경우에 사용합니다. 단순하게 이해하자면, 어떤 메시지를 보내면, 정해진 시간 안에 반드시 응답이 도착하는 네트워크라고 생각하면 됩니다. 비행기나 우주선의 통신과 같이, 노드 간 통신이 반드시 필요한 네트워크에서 주로 사용됩니다.
>
> - 비동기 네트워크(Asynchronous Network) : 노드가 메시지를 보내는 시간과 수신 노드가 메시지를 수신하는 시간 사이에 상한선이 없습니다. 즉 메시지가 수신 노드에게 제대로 도착했는지 여부를, 수신 노드가 응답하는 데 얼마나 시간이 걸리는지를 알 수 없습니다. 따라서 특정 노드가 임의로 메시지를 전송하지 않거나 거짓 데이터를 전송하는 배신행위가 가능한 네트워크이기도 합니다. 인터넷이나 블록체인 네트워크가 여기에 해당합니다.
> - 부분 동기 네트워크(Partial synchronous Network) : 노드가 메시지를 보내는 시간과 수신 노드가 메시지를 수신하는 시간 사이에 상한선이 존재하지만, 그 상한선이 얼마인지는 알 수 없는 네트워크입니다. 단순히 표현하자면 ‘노드와 노드 사이에 메시지가 도달하는 것은 보증하지만, 언제 도달할지는 알 수 없다’고 말할 수 있습니다. 네트워크에서 메시지가 도달하는 시간의 상한선이 유한(finite)한 것이 부분 동기 네트워크이고, 무한(infinite)한 것이 비동기 네트워크라는 점에서 비동기 네트워크와 차이가 있습니다.



블록체인이 구동되는 네트워크는 비동기 네트워크입니다. 따라서 비동기 네트워크에서 완벽한 합의 알고리즘은 존재하지 않습니다. 다시 말해, Safety와 Liveness를 동시에 완벽히 만족하는 합의 알고리즘을 설계할 수가 없습니다. 즉 **블록체인에서 어떤 합의 알고리즘을 채택한다는 것은, Safety와 Liveness 중 하나를 어느 정도 포기해야 한다**는 것을 말합니다.
<br>
<br>

## PBFT란??

Safety를 확보하고 Liveness를 일부 희생하면서, 비동기 네트워크에서도 합의를 이룰 수 있는 알고리즘이 바로 Practical Byzantine Fault Tolerance, PBFT입니다. 즉 네트워크에 배신자 노드가 어느 정도 있다고 해도 네트워크 내에서 이루어지는 합의의 신뢰를 보장하는 알고리즘입니다. 현재까지 블록체인 합의 알고리즘 중 BFT 방식을 채택했다고 하는 경우 대부분 PBFT 합의 알고리즘을 바탕으로 조금씩 변형을 가했다고 볼 수 있습니다. 대표적으로 Tendermint는 PBFT에 DPoS 합의 알고리즘을 결합했으며, 이더리움 Casper는 PoW 방식의 채굴 위에 PoS + PBFT 형태의 블록 검증 시스템을 제안했습니다. 이외에도 PBFT는 Hyperledger Fabric, R3, Ripple, EOS에 이르기까지 Public과 Private을 가리지 않고 다양한 블록체인에서 사용되고 있습니다.



PBFT를 간단히 요약하자면, 비동기 네트워크에서 배신자 노드가 f개 있을 때, 총 노드 개수가 3f+1개 이상이면 해당 네트워크에서 이루어지는 합의는 신뢰할 수 있다는 것을 수학적으로 증명한 알고리즘입니다. 네트워크의 모든 노드는 거래와 같은 합의 대상의 상태를 변화할 것인지 prepared certificate와 commit certificate라는 두 번의 절차를 거쳐 결정합니다.

<br>
<br>


## PBFT 합의 알고리즘 절차 쉽게 이해하기

PBFT 합의 알고리즘 논문이 말하는 PBFT 합의 절차를 쉽게 이해해 보겠습니다. MIGUEL CASTRO, BARBARA LISKOV 의 논문 "Practical Byzantine Fault Tolerance and Proactive Recovery" 의 내용을 바탕으로 이해했으며, 비잔틴 노드(배신자 노드)의 개수가 최대 f일 때, 전체 네트워크 노드 수가 3f+1개라는 전제로 절차가 진행됩니다.
<br>


![k-308](https://user-images.githubusercontent.com/26548454/42135090-76b1f026-7d81-11e8-8134-ec46f48c981d.jpg)
\[그림2 - PBFT 합의 알고리즘 설명. 출처: Kodebox 세미나 자료]

<br>

1. 클라이언트가 상태 변환을 요청하는 Request 메시지 m을 Primary Node에 전송합니다. 논문에서는 처음 상태변환 요청을 받은 노드를 Primary로, Primary를 제외한 네트워크 나머지 노드를 Backup이라고 칭합니다. 그림에서는 0번 노드를 의미합니다.

​     

2. Primary가 request 요청을 받으면 먼저 Pre-prepare라는 절차를 시행합니다. 
   - 해당 request에 대응하는 Sequential number "N"을 생성합니다.
   - 네트워크의 나머지 모든 노드에게 Pre-prepare 메시지를 전송합니다. 메시지의 구성은 <Pre-prepare, V, N, D(m)>입니다. V는 메시지가 전송되는 View를 의미하고(View in which the message is being sent), N은 Sequential Number, D(m)은 요청 메시지 m의 요약본입니다.

​     

3. 네트워크 내 임의의 Backup 노드 i가 Pre-prepare 메시지를 받고, D(m)과 V, N이 서로 대응되는 값인지 검증합니다. 만약 검증 결과 서로 대응되지 않는 값이라면 Pre-prepare 메시지를 수용하지 않습니다. 검증 결과가 참이라면, Prepare 메시지를 생성해 네트워크의 나머지 모든 노드에게 전송합니다.  
   - Prepare 메시지는 <Prepare, V, N, D(m), i> 형태입니다. V와 N, D(m)은 Pre-prepare에서 받은 값과 동일하며, i는 Pre-prepare 메시지를 검증한 Backup노드의 번호라고 이해하면 되겠습니다. 즉 모든 노드가 메시지를 검증하고, 검증한 결과가 참일 경우 모든 노드에게 Prepare라는 이름의 메시지를 전송합니다.

​     

4. 각각의 노드는 Pre-prepare 메시지와 Prepare 메시지를 수집합니다. 수집한 Pre-prepare 메시지 개수가 2f+1개이고 Prepare 메시지가 2f개 이상인 경우 "prepared certificate"라고 부르며, 해당 노드는 "prepared the request" 상태가 됩니다.

​     

5. "prepared certificate" 조건을 만족한 노드는 네트워크의 모든 노드에게 Commit 메시지를 전송합니다.
   - Commit 메시지의 구성은 <commit, V, N, i> 입니다.

​     

6. 각각의 노드는 Commit 메시지를 수집합니다. Commit 메시지가 2f+1개 모이면 해당 노드는 "commit certificate"상태가 됩니다. 

​     

"prepared certificate"와 "commit certificate" 두 가지가 모두 있을 경우 해당 노드는 “committed certificate”가 되며, 클라이언트가 요청한 request를 수용해 상태 변화 함수를 시행합니다. 즉 네트워크의 합의 알고리즘이 작동하여 합의를 도출합니다. Request를 수용할 경우 네트워크 어느 노드에서도 상태 변화를 수용했기 때문에 Safety를 충족합니다. 하지만 네트워크 합의 요건을 충족하지 못한 request는 기각하기 때문에 Liveness를 일부 희생했다고 이해할 수 있습니다.
<br>
<BR>

## PBFT 합의 절차 - 두 번의 절차를 걸쳐서 합의해야만 하는 이유?

비동기 네트워크에서 한 번의 절차로만 합의를 시행할 경우, 배신자 노드가 합의 알고리즘의 Safety를 파괴할 수 있습니다.

4개의 노드가 존재하는 네트워크를 가정해 보겠습니다.
<br>
![k-308 2](https://user-images.githubusercontent.com/26548454/42135099-93926112-7d81-11e8-8bfc-bbf4336e6f43.jpg)
\[그림3 - 한 번의 합의절차만을 전제로 한 합의 알고리즘의 의사결정단계_1 출처: Kodebox 세미나 ]
<br>
<br>
노드1이 블록 B1을 생성하고 모든 네트워크에게 전파했습니다. 그리고 노드1과 노드2는 해당 블록을 검증한 결과를 노드1, 2, 3, 4에게서 전부 받았습니다. 그런데 노드3과 노드4는 모종의 이유로 나머지 노드들의 검증 결과를 받지 못했다고 가정해 보겠습니다. PBFT 알고리즘이라면, 노드3과 노드4에서는 prepared certificate 상태에 도달하지 못해 commit 메시지를 보내지 못합니다. commit 메시지가 과반수에 미치지 못하므로 블록1은 commit되지 못합니다.

<br>

만약 certificate 한 번 만으로 commit을 할 수 있다고 가정해 보겠습니다. 여기서 노드2가 배신자 노드일 경우, 노드2는 다음과 같은 행동이 가능합니다.

![k-309](https://user-images.githubusercontent.com/26548454/42135112-a409c328-7d81-11e8-9dba-c7c983eb8c07.jpg)
\[그림4 - 한 번의 합의절차만을 전제로 한 합의 알고리즘의 의사결정단계_2 출처: Kodebox 세미나 자료]
<br>
<br>
- 먼저 노드1에게는 블록1을 Commit 합니다. 이제 노드1은 R2 단계에서의 블록은 블록1이라고 간주하게 됩니다.
- 합의에 도달할 투표수가 부족한 노드3과 노드4에게는 새로운 블록2를 제안합니다. 노드3과 4는 블록1이 유효성 검증을 받지 못해 탈락한 것으로 알고, 노드2과 같이 블록2의 유효성 검증을 진행합니다.
- 노드2, 3, 4 모두 블록2의 유효성 검증을 마치고 certificate를 얻었기에 블록2를 Commit합니다. 이제 노드 3, 4는 R2 단계의 블록은 블록2라고 간주하게 됩니다.



이 결과, R2 단계에서 Commit 인증을 받은 블록은 블록1과 블록2 두 개가 됩니다. PBFT 시스템의 최대 장점은 배신자가 발생할 수 있는 비동기 네트워크에서 Safety를 보장하는 것입니다. 그런데 투표 절차가 한 번에 불과할 경우, 한 명의 배신자 노드가 네트워크 합의 시스템의 Safety를 무력화하는 상황이 발생합니다. 합의 알고리즘이 무너지는 것이죠.

따라서 PBFT는 prepared certificate와 commit certificate 라는 두 번의 절차를 활용하여 배신자 노드가 존재하는 상황에서도 네트워크의 합의를 도출합니다.
<br>


## PBFT 합의 알고리즘을 블록체인에 적용한 사례

PBFT 알고리즘을 블록의 합의에 적용한 대표적인 사례로는 Tendermint를 들 수 있습니다. Tendermint 합의 알고리즘을 분석하기에는 글이 너무 길어져서, 이번 글에서는 Tendermint의 합의 알고리즘이 PBFT의 각각 어느 부분에 대응되는지 설명하겠습니다.



![consensus_logic](https://user-images.githubusercontent.com/26548454/42135120-b1dd6892-7d81-11e8-8dfc-6d45e59b8f9e.png)
\[그림5 - Tendermint의 블록 생성 과정 출처: Tendermint readthedocs]
<br>
<br>

Tendermint는 Propose, Prevote, Precommit 과정을 거쳐 블록을 생성합니다. 각 라운드마다 블록을 제안하며,  매 라운드에서 합의를 거쳐 블록을 생성합니다.

- Tendermint에서 클라이언트가 네트워크에 블록의 생성을 Request하는 과정이 Propose입니다.
- Propose된 블록을 각 노드가 검증하고, 검증한 결과 참인지 거짓인지를 투표하는 것이 Prevote입니다. 각 노드가 블록을 검증한 결과를 네트워크에 전달하는 것이므로 Prepare 과정에 비교할 수 있습니다. Prevote Block이 전체의 2/3이상일 경우 tendermint에서는 "polka"라고 부르는데, 이는 "prepared certificate"에 대응됩니다.
- Prevote 이후 Precommit 과정을 다시 한 번 진행합니다. Precommit에 동의한 노드가 전체의 2/3이상일 경우 블록을 commit합니다. "commit certificate"에 대응될 수 있으며, commit에 필요한 2/3 이상의 Precommit을 얻지 못할 경우 블록을 생성하지 않고 다음 라운드로 진행합니다.



## Conclusion

꽤 긴 지면을 할애해 PBFT 합의 알고리즘을 설명했습니다. 최대한 간명하게 풀어보았으나, 컴퓨터공학이나 분산컴퓨팅 비전공자 입장에서 논문을 읽고 이해한 내용이다 보니 설명이 부족하거나 명확하지 못한 부분이 있을 거라고 생각합니다. 학술적인 차원에서 깊게 이해하기엔 부족한 글이지만, PBFT 합의 알고리즘이 어떤 것인지 맥을 잡기에는 조금이나마 도움이 되셨으면 합니다. 감사합니다.



## Reference

- Byzantine Fault Tolerance - en.wikipedia
- A Brief tour of FLP Impossibility
- 2018.05.23 Kodebox 세미나 - BFT Consensus Algorithm
- Distributed Systems courses L6 - Byzantine Fault tolerance
  (https://www.youtube.com/watch?v=_e4wNoTV3Gw&t=36s) 
- 사진 출처: Kodebox 세미나 자료 <https://docs.google.com/presentation/d/10W7gKEvk_6XRIlSdiKwnwP9gVzo5Re5m_24QzLGaqvk/edit>
- Synchronous, partially synchronous and asynchronous consensus algorithms 
  (https://www.reddit.com/r/ethereum/comments/7e0yze/synchronous_partially_synchronous_and/)
- Practical Byzantine Fault Tolerance and Proactive Recovery - MIGUEL CASTRO,BARBARA LISKOV
  (http://www.pmg.csail.mit.edu/papers/bft-tocs.pdf)
- BFT 합의 알고리즘 - theloop
  (https://blog.theloop.co.kr/2017/06/21/bft-%EA%B8%B0%EB%B0%98-%ED%95%A9%EC%9D%98-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98/)
- What is Tendermint?
  (https://tendermint.readthedocs.io/projects/tools/en/master/introduction.html)
- https://blog.seulgi.kim/2018/05/safety-liveness-in-blockchain.html

(Kodebox 세미나 자료 사용을 허락해주신 서광열 대표님께 감사드립니다.)
