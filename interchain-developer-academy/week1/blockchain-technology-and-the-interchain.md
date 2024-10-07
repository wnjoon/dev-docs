# Blockchain Technology and the Interchain

## [How does the interchain solve the scalability issue?](https://ida.interchain.io/academy/1-what-is-cosmos/1-blockchain-and-cosmos.html#how-does-the-interchain-solve-the-scalability-issue)

블록체인 성능에는 크게 수직적/수평적 확장 방법이 있다.

- 수직적 확장(vertical scalability)
    - 하나의 노드(체인)의 성능을 개선해서 해당 노드가 모든 트랜잭션을 더 빨리 처리하도록 한다.
    - 예) 노드의 CPU, 메모리 등의 성능 개선
- 수평적 확장(horizontal scalability)
    - 여러 노드(체인)을 추가해서 여러 트랜잭션을 각 노드가 분담하여 동시에 처리하도록 한다.
    - 예) 10개의 트랜잭션을 동시에 처리하기 위해 노드의 개수를 5개에서 10개로 증가

그렇다면 IBC와 수평적 확장은 어떤 관계가 있을까?

- IBC는 여러 블록체인을 연결하여 어플리케이션이나 트랜잭션의 처리가 여러 개의 체인으로 분산되도록 한다.
- IBC는 서로 다른 체인간에 데이터를 교환할 수 있도록 상호운용성을 제공함으로써, 각 체인마다 동시에 독립적으로 처리된 정보를 다른 체인으로 전달하여 하나의 큰 네트워크 처럼 동작하도록 한다.
- IBC에서 체인 간 데이터 전송 시 암호화된 증명과 검증을 통해 데이터의 신뢰성을 보장하여 데이터 불일치를 해결한다.

## [How does the interchain solve the scalability issue?](https://ida.interchain.io/academy/1-what-is-cosmos/1-blockchain-and-cosmos.html#how-does-the-interchain-solve-the-scalability-issue)

인터체인과 일반적인 블록체인(이더리움, 솔라나 등) 모두 블록체인 위에 어플리케이션이 존재한다는 공통점이 있지만, 인터체인의 경우 어플리케이션 마다 독립적으로 블록체인을 운영하기 때문에 제한 범위가 줄어든다. 하지만 인터체인도 동일하게 블록체인 위에서 어플리케이션이 동작하는 구조이기 때문에, 거버넌스 문제가 발생할 가능성은 존재한다.

특히 IBC의 경우 체인간 상호작용을 할 때에는 IBC 또는 기타 프로토콜에 대한 영향을 받기 때문에 각 체인마다 서로 다른 합의 매커니즘에 대한 조율이 필요하며, 특정 블록체인이 중앙 허브와 같은 역할을 담당할 경우 해당 체인의 거버넌스가 다른 체인에 영향을 주지 않도록 관리해야한다.