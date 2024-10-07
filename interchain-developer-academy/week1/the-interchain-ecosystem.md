# The Interchain Ecosystem 

## [The Inter-Blockchain Communication Protocol](https://ida.interchain.io/academy/1-what-is-cosmos/2-cosmos-ecosystem.html#the-inter-blockchain-communication-protocol)

IBC 모듈은 허브(hub)와 존(zone)으로 구성된다. 중요한 것은 허브와 존 모두 블록체인이다.

존(zones)
- 특정 어플리케이션이나 기능을 중심으로 구축된 블록체인

허브(hubs)
- IBC를 통해 여러 존들을 연결하는 블록체인
- 데이터와 자산이 안전하게 이동할 수 있도록 보장한다. 
    - 예로 토큰의 이중 치줄(double spending) 문제와 같은 리스크를 최소화할 수 있다.
- 체인(존)간의 직접 연결 수를 줄임으로써 네트워크의 복잡도를 줄이면서 각 존들이 서로 연결되지 않고도 상호작용할 수 있도록한다.

## [The Cosmos Hub](https://ida.interchain.io/academy/1-what-is-cosmos/2-cosmos-ecosystem.html#the-cosmos-hub)

코스모스허브(cosmos hub)

- 인터체인 스택을 기반으로 만들어진 PoS 합의알고리즘 기반 최초의 블록체인이다. 
- ATOM이라는 기본 네이티브 토큰을 거래 수수료 지불 및 스테이킹에 사용한다.  
    - 코스모스허브에서 거래수수료는 반드시 ATOM으로만 지불되지는 않는데, 만약 특정 존과 코스모스허브가 서로 신뢰관계를 형성하고 있고 해당 존에서 발행된 토큰도 수수료 지불 수단으로 허용된다면 해당 토큰으로 거래 수수료 지불이 가능하다.

페그존(peg zone)
- 꼭 cometBFT(Tendermint) 기반의 체인만 IBC 네트워크에 연결될 수 있는 것은 아니다. 
    - fast-finality를 만족하는 체인이라면, 모두 존으로 연결 가능하다.
- probabilistic-finality 기반의 체인(비트코인, PoW 기반 이더리움 등)은 시간이 지남에 따라 점점 블록이 변경되지 않는다는 확률이 증가하는 구조로, IBC와 바로 연결할 수 없다.
    - 이러한 경우 페그존이라는 fast-finality 기반의 블록체인을 probabilistic-finality 기반의 체인과 브릿지처럼 연결함으로써 IBC 내 다른 체인들과 상호작용할 수 있다.
    - 예) 이더리움에는 [Gravity Bridge](https://github.com/cosmos/gravity-bridge)라는 페그존이 구현되어 있다.

## [Ignite CLI - building application-specific blockchains with one command](https://ida.interchain.io/academy/1-what-is-cosmos/2-cosmos-ecosystem.html#ignite-cli-building-application-specific-blockchains-with-one-command)

Ignite CLI는 Cosmos SDK 기반으로 만들어진 command-line interface를 의미한다.

- 공식 홈페이지 : [https://ignite.com/](https://ignite.com/)

## [Cosmwasm - multi-chain smart contracts](https://ida.interchain.io/academy/1-what-is-cosmos/2-cosmos-ecosystem.html#cosmwasm-multi-chain-smart-contracts)

cosmos 생태계에서 스마트 컨트랙트를 실행하기 위한 플랫폼으로, cosmos sdk를 사용하는 모든 체인에서 작동될 수 있도록 설계되어 있다. Cosmwasm은 Rust 언어 기반으로 되어있다.

Wasm(WebAssembly)은 플랫폼과 언어에 구애받지 않는 범용적인 실행환경을 제공하기 때문에, 코드가 체인으로부터 독립적이 된다. 
- Wasm은 바이트코드 형식의 코드로 컴파일되기 때문에, 특정 프로그래밍 언어나 하드웨어에 종속되지 않고 다양한 플랫폼(체인)에서 동일하게 실행된다.

