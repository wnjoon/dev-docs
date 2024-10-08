# A Blockchain App Architecture

## [Consensus with CometBFT and the interchain](https://tutorials.cosmos.network/academy/2-cosmos-concepts/1-architecture.html#consensus-with-cometbft-and-the-interchain)

> Network participants are incentivized to stake their ATOM with nodes which are the most likely to provide dependable service, and to withdraw their support should those conditions change. A Cosmos blockchain is expected to adjust the validator configuration and continue even in adverse conditions.

Cosmos에서 ATOM을 스테이킹하는 것은 네트워크 단위가 아니라 특정 노드(검증자) 단위로 이루어진다.

**검증자와 위임자의 차이**

- 검증자(validator) 
    - Cosmos 블록체인 네트워크에서 블록을 생성하고 트랜잭션을 검증하는 노드 
    - Cosmos에서 모든 검증자는 고유하며, 각 검증자는 네트워크 참여자로부터 ATOM을 위임받아 운영된다.
    - 스테이킹 된 ATOM 기준으로 상위 175개의 노드만 검증자로서 참여할 수 있다.
    - 블록을 생성할 수 있는 권한은 검증자가 보유한 투표권(voting power)에 비례하여 부여되는데, 투표권은 검증자와 위임자가 스테이킹한 모든 토큰(ATOM)의 합으로 계산된다.

- 위임자(delegator)
    - ATOM을 보유한 네트워크 참여자는 직접 검증자로 활동하지 않아도 자신이 믿을 수 있는 검증자에게 ATOM을 위임할 수 있다.
    - ATOM을 위임한 위임자는 검증 과정에 참옇알 수 있고, 검증자가 얻는 보상의 일부를 나눠 가질 수 있다.

    > 예로 10번 블록을 생성할 시점에 검증자A의 투표권이 전체 검증자들(175개) 투표권의 15%인 경우
    > - 검증자A가 10번 블록을 생성할 수 있는 확률은 15%이다.
    > - 만약 검증자A가 15%의 확률을 뚫고 블록을 생성하면, 블록을 생성함으로써 얻는 보상으로 인하여 보유한 토큰(ATOM)수가 증가한다.
    > - 11번 블록을 생성할 시점에, 검증자 A는 15%보다 높은 투표권을 가지게 된다. (위임자로부터 유입되는 추가 토큰이 있을 경우 투표권은 더욱 높아질 수 있음)

**스테이킹 방식**

- ATOM을 스테이킹 할 때에는 네트워크 전체에 걸쳐 분산하는 것이 아닌, 특정 검증자(노드)에 ATOM을 위임하는 구조이다.
- 위임자는 검증자의 성능과 신뢰도를 기준으로 스테이킹할 검증자를 선택하며, 검증자의 행동에 따라 위임자 또한 인센티브 또는 페널티(slashing)를 부여받는다.

## [Application Blockchain Interface (ABCI)](https://tutorials.cosmos.network/academy/2-cosmos-concepts/1-architecture.html#application-blockchain-interface-abci)

코스모스에서의 어플리케이션과 블록체인

- 코스모스의 기본 철학은 <u>각 어플리케이션이 독립적인 블록체인(어플리케이션 별 블록체인)을 구축</u>하는 것이다. 이를 통해 블록체인은 이와 연결된 어플리케이션에 맞춤화 되어 있기 때문에 성능과 효율성을 극대화할 수 있다. 또는 여러 어플리케이션이 하나의 블록체인 위에서 구동될 수도 있다. 이는 해당 블록체인이 여러 어플리케이션의 요구를 충족할 수 있도록 설계되었을 경우(공통된 기능을 포함하는 등) 가능하다.
- 어플리케이션마다 독립적인 블록체인을 구축하게 되면, 어플리케이션마다 필요에 맞게 합의알고리즘 또는 블록체인 내부를 설정함으로써 트랜잭션 처리 속도와 같은 요구사항을 만족할 수 있다.

> 모듈식 구조를 갖는 cosmos sdk의 특성상, 블록체인에 모듈을 추가함으로써 새로운 어플리케이션에서 요구되는 사항을 쉽게 추가할 수 있다. 이를 통해 하나의 블록체인에서 동적으로 또다른 어플리케이션을 구동할 수 있다.

ABCI는 cometBFT와 어플리케이션이 서로 통신하기 위해 사용하는 인터페이스로, 어플리케이션에서 트랜잭션이 발생하였을 때 진행되는 과정은 아래와 같다.

1. 어플리케이션 계층에서 트랜잭션이 생성된다. 트랜잭션은 바이트배열([]byte) 형태로 되어있다.
2. 어플리케이션 계층에서 ABCI를 통해 cometBFT로 트랜잭션을 전달한다. cometBFT는 합의알고리즘을 통해 해당 트랜잭션을 블록에 포함시킨다. 이 과정에서 cometBFT는 트랜잭션의 실제 내용에 대해서 해석하지 않는다.
3. 블록에 포함된 트랜잭션이 확정되면, cometBFT는 ABCI를 통해 해당 트랜잭션을 어플리케이션 계층으로 전달한다. 어플리케이션 계층에서 해당 트랜잭션을 해석하고 그에 따른 상태 변화를 블록체인에 반영한다.

이렇게 합의 과정과 어플리케이션 계층을 분리함으로써 얻을 수 있는 장점은 아래와 같다.
- 네트워크와 어플리케이션 로직 사이의 복잡성을 낮춤으로써, cometBFT는 빠르게 블록을 생성하고 합의하는 일에 집중하고 어플리케이션은 트랜잭션의 처리와 해석에만 집중하여 성능과 확장성을 높일 수 있다.
- 어플리케이션 계층에서의 로직이 독립적으로 동작하기 때문에, 각 블록체인마다 요구에 맞는 상태 변화 규칙을 유연하게 설정할 수 있다.

#### 이더리움과 cometBFT의 트랜잭션 처리 방식 차이

이더리움
- 이더리움에서 트랜잭션이 제출되면, 각 노드는 EVM에서 해당 트랜잭션을 실행하고 상태 변화를 미리 계산한다.
- 트랜잭션이 정상적으로 처리될 경우에는 해당 결과를 바탕으로 블록이 생성되나, 트랜잭션이 실패(revert)하는 경우 해당 내용은 블록에 포함되지 않고 트랜잭션은 처리되지 않은 상태가 된다.
- 즉, 트랜잭션의 실행 결과는 블록 생성 전에 결정되고 해당 결과에 대해 노드들이 합의를 이룬다.

cometBFT
1. 트랜잭션 수신 및 합의
    - 사용자가 제출한 트랜잭션이 네트워크로 전달되면, cometBFT는 해당 트랜잭션을 블록에 포함시키기 위한 합의 과정을 거친다.
    - cometBFT는 트랜잭션의 세부적인 실행결과나 상태 변화에 대해서는 신경쓰지 않으며, cometBFT 입장에서 모든 트랜잭션은 단순히 바이트 배열([]byte) 형태의 데이터로만 취급된다.
    - 합의가 이루어지면 블록이 네트워크에 전파된다.

2. 어플리케이션 계층에서 트랜잭션 처리
    - 블록이 확정되면, 블록에 포함된 트랜잭션들은 ABCI를 통해 어플리케이션 계층으로 전달된다.
    - 어플리케이션 계층에서 해당 트랜잭션을 해석하고, 트랜잭션의 유효성 및 상태 변화를 확인한다.
    - 트랜잭션이 유효하지 않을 경우 해당 트랜잭션을 무효화하는 처리과정이 진행된다. 이 경우 상태 변화는 일어나지 않지만 해당 트랜잭션은 블록에 그대로 기록된다.

요약하면,
- 이더리움과 다르게 코스모스(cometBFT)에서는 합의를 통한 블록이 먼저 생성되고, 트랜잭션에 대한 검증은 추후 어플리케이션 계층을 통해 진행된다. 트랜잭션이 유효하지 않아도 블록에는 그대로 기록되며, 유효한 트랜잭션의 결과에 대한 상태 변화는 이후에 진행된다. 이 때 상태 변화에 대한 세부 내용은 블록에 기록되지 않으며 어플리케이션 계층 안에서반 반영된다.

## [Additional details](https://tutorials.cosmos.network/academy/2-cosmos-concepts/1-architecture.html#additional-details)

트랜잭션의 검증부터 블록의 생성 및 상태변화 과정에 사용되는 메시지를 순서대로 나열해보면 아래와 같다.

1. [beginblock](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#beginblock)
    - 새로운 블록을 생성하기 전에 cometBFT로부터 호출된다.
    - 블록 생성과 관련된 초기화 작업(블록 컨텍스트 설정 등)을 수행할 수 있다.

2. [checkTx](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#checktx-1)
    - 블록에 포함될 예정인 트랜잭션 묶음을 개별로 유효성 검증하며, 검증에 통과한 트랜잭션만 mempool에 추가된다.
    - 예) 필수 필드(송신자 또는 수신자 주소)의 누락, 서명 오류(트랜잭션 생성자의 공개키와 서명이 일치하지 않는 경우 등), 부정확한 수수료 설정(너무 낮거나 부정확한 수수료) 등

3. [deliverTx](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#delivertx-1)
    - 트랜잭션이 블록에 포함된 이후, 어플리케이션 계층에서 해당 트랜잭션을 실행하고 상태 변화를 일으키는 단계에서 발생하는 논리적 오류 또는 조건의 불충족 여부를 검증한다. 
    - 이 경우 트랜잭션은 실패할 수 있지만, 이미 블록에 기록된 상태이므로 이력으로는 남게 된다.
    - 예) 잔액 부족, 트랜잭션 실행 권한 오류 등

4. [endblock](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#endblock)
    - 블록 내 모든 트랜잭션이 실행되고 상태 변화가 완료되었을 때 cometBFT로부터 호출된다.
    - 검증자 집합 업데이트 또는 최종 상태 확인 등 블록 처리의 마무리 단계에서 사용된다.

