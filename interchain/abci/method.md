# Method

## Consensus/Block Execution

새로운 블록체인이 시작되면, cometBFT는 InitChain 메소드를 호출한다. 이후부터는 블록의 합의 과정마다 PrepareProposal, ProcessProposal, ExtendVote, VerifyVoteExtension 메소드가 호출되고, 이후 FinalizeBlock이 호출된다. 그리고 메소드 호출 결과에 따라 어플리케이션의 상태가 업데이트(Commit)된다. 

> ABCI 2.0부터 BeginBlock, DeliverTx(optional), EndBlock 메소드가 FinalizeBlock로 합쳐졌다.

### InitChain

새로운 블록체인의 초기 validator set을 설정할 수 있다.

```go
// Psuedo code
if InitChainResponse.Validators == empty {
    initial validator set = InitChainRequest.Validators
} else {
    initial validator set = InitChainResponse.Validators
}
```

### [PrepareProposal](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#prepareproposal)

r번째 합의 라운드 시점에 h번째 블록에 대하여 validator p가 제안자인 경우,

1. 제안자(p)의 cometBFT는 자신(p)의 mempool로부터 우선순위에 따른 트랜잭션들을 수집한 raw proposal을 생성하고, 이를 통해 블록 헤더를 생성한다.
2. 제안자(p)의 cometBFT는 위에서 만들어진 블록의 정보를 포함한 데이터를 가지고 PrepareProposal을 호출한다. PrepareProposal은 Sync 방식(Response가 올때까지 block)으로 동작한다.
3. 필요에 따라 어플리케이션은 수신된 정보를 사용하여 제안(proposal)을 수정할 수 있다. 
    - 트랜잭션 추가
    - 트랜잭션 삭제(mempool에는 트랜잭션이 그대로 유지된다)
    - 트랜잭션 순서 재정렬
    - 만약 트랜잭션 목록이 변경되는경우, 트랜잭션의 총 크기는 PrepareProposalRequest.max_tx_bytes보다 크면 안된다.
4. 제안자(p)는 PrepareProposal 결과로 반환된 트랜잭션 목록을 제안에 사용한다.

> 만약 클라이언트(사용자)가 순서대로 제출한 트랜잭션 t1, t2가 어플리케이션에서 재정렬되어 t2만 블록에 커밋되었을 경우, cometBFT에서는 이러한 변경 사항을 인지할 수 없다. 그러므로 트랜잭션 변경이나 순서에 대한 추적성을 유지하고싶다면 어플리케이션에서 이를 관리해야한다.

### [ProcessProposal](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#processproposal)


