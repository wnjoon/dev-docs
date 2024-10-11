# Method

## Consensus/Block Execution

새로운 블록체인이 시작되면, cometBFT는 InitChain 메소드를 호출한다. 이후부터는 블록의 합의 과정마다 PrepareProposal, ProcessProposal, ExtendVote, VerifyVoteExtension 메소드가 호출되고, 이후 FinalizeBlock이 호출된다. 그리고 메소드 호출 결과에 따라 어플리케이션의 상태가 업데이트(Commit)된다. 

> ABCI 2.0부터 BeginBlock, DeliverTx(optional), EndBlock 메소드가 FinalizeBlock로 합쳐졌다.

### [InitChain](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#initchain)

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

제안자(proposer)로 선정된 노드에서 제안된 블록 정보를 어플리케이션으로 전달한다. 어플리케이션은 필요에 따라 트랜잭션 목록을 수정(추가, 삭제 또는 재정렬)할 수 있으며, 결과를 응답으로 반환한다.

예로 vaidator p가 r번째 합의 라운드 시점에 h번째 블록에 대하여 제안하는 경우 (validValue = nil)

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

제안자가 제안한 블록을 모든 검증자(validator) 노드가 검증한다. 제안된 내용은 어플리케이션에서 실제로 트랜잭션을 실행하는 FinalizeBlock 단계와 동일하나, 실행 단계를 거치지 않은 후보 단계로 유지된다. 검증에 실패하였다고 해서 바로 라운드가 종료되는 것은 아니며, 합의 과정에서 블록이 거부될 수 있다. 합의 결과로 다른 블록이 선정되면, 해당 블록은 폐기된다.

> 라운드(round): 블록 합의과정에서의 시도 횟수를 의미하며, 특정 높이(h)에서 라운드는 0부터 시작하여 검증에 실패하거나 합의에 도달하지 못할때마다 1씩 추가된다. 

해당 라운드의 제안자(proposer)에서도 ProcessProposal은 호출된다. 즉 *제안자는 PrepareProposal -> ProcessProposal 순으로 호출하며, 그 외의 검증자는 ProcessProposal만 호출한다*.

### [ExtendVote](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#extendvote)

검증자 노드가 투표 과정에 필요한 추가 정보가 있을 경우 이를 추가한다.

### [VerifyVoteExtension](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#verifyvoteextension)

검증자 노드 중 추가 정보를 설정했을 경우, 다른 검증자 노드들이 이를 검증한다.

### [FinalizeBlock](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#finalizeblock)

생성된 블록이 투표를 통해 승인되면, 각 노드의 cometBFT는 어플리케이션으로 블록을 전달한다. 어플리케이션은 해당 트랜잭션을 실행한다. 하지만 실제 상태 변화가 확정되지는 않는다.

> ABCI 1.0에서의 BeginBlock -> [DeliverTx] -> EndBlock 과정과 동일하다. 

- FinalizeBlockRequest.decided_last_commit에서 블록 생성 검증자 별 reward, FinalizedBlockRequest.misbehavior에서 잘못된 노드에 대한 punishment를 결정한다.
- FinalizeBlockRequest.txs에 있는 트랜잭션들을 실행하거나, PrepareProposal 또는 ProcessProposal을 통해 해당 블록에 대해 이전에 실행했었던 상태를 적용한다. 트랜잭션들 중 i번째 트랜잭션이 유효하게 처리 완료된 경우 FinalizeBlockResponse.tx_results[i].code를 0으로 설정한다.
- 어플리케이션에서 아래 값들을 채워서 응답한다.
    - FinalizeBlockResponse.app_hash: 어플리케이션의 상태에 대한 머클루트 해시값으로, 다음 블록 헤더의 AppHash 값으로 설정
    - FinalizeBlockResponse.tx_results: 어플리케이션에서 실행된 해당 블록 내 트랜잭션 목록
    - FinalizeBlockResponse.validator_updates: 이전 블록과 동일한 validator set이 참여하였으면 empty, 변경되었다면 업데이트 
    - FinalizeBlockResponse.consensus_param_updates: 이전 블록과 동일한 합의 파라미터가 사용되었으면 empty, 변경되었다면 업데이트
    - FinalizeBlockResponse.next_block_delay: 어플리케이션과 cometBFT가 커밋된 블록을 처리하는데 걸리는 시간으로, 0으로 셋팅할 경우 즉시 처리 (값이 셋팅될 경우 2/3 이상의 voting power를 받았음에도 계속 precommit을 받을 수 있음)

H번째 블록의 FinalizeBlock으로 인하여 영향을 받는 블록들은 아래와 같다.
- 블록 H의 FinalizeBlockResponse.validator_updates == 블록 H의 헤더의 NextValidatorsHash
- 블록 H+1의 헤더의 ValidatorsHash == 블록 H의 헤더의 NextValidatorsHash
- 블록 H+2의 헤더의 ValidatorsHash == 기본적으로 블록 H+1의 ValidatorsHash가 되나, 블록 H+1의 FinalizeBlockResponse.validator_updates 값에 따라(empty가 아닌 경우) 변경될 수 있음
- 블록 H+3의 PrepareProposal, ProcessProposal, FinalizeBlock의 _last_commit 필드 = 블록 H+2에서의 최종 validator set으로 작성

h번째 블록에 대한 합의에 참여한 노드 P는 아래 정보를 수신한다.
- 해당 라운드(r)에서 생성할 블록(v)에 대한 제안 정보
- 해당 라운드(r)의 h번째 블록(v)에 대해 검증자들이 각자의 투표권을 통해 precommit한 메시지

노드 p는 아래의 단계를 거쳐 높이 h에 대한 합의를 마무리하고 블록 v를 결정한다.
1. 노드 p는 이전 단계로부터 생성된, 높이 h에서의 합의 결과로 생성될 블록 v를 준비한다.
2. 노드 p의 cometBFT가 블록 v에 대한 데이터와 함께 FinalizeBlock을 호출한다.
3. 노드 p의 어플리케이션이 블록 v를 실행한다.
4. 노드 p의 어플리케이션에서 실행된 각 트랜잭션의 결과 목록과 트랜잭션의 실행 결과로 인한 어플리케이션의 상태를 나타내는 AppHash를 계산하고 반환한다.
5. 노드 p의 cometBFT에서 각 트랜잭션들의 실행 결과들을 해시한 결과를 ResultHash에 저장한다.
6. 노드 p의 cometBFT에서 이전 단계에서 생성된 트랜잭션 결과, AppHash, ResultHash를 준비한다.
7. 노드 p의 cometBFT는 mempool을 잠그고 새로운 트랜잭션에서의 CheckTx 호출을 막는다.
8. 노드 p의 cometBFT는 Commit을 호출하여 어플리케이션에 상태를 유지하도록 지시한다.
9. 노드 p의 cometBFT는 해당 어플리케이션 상태에 해당하는 mempool 내 트랜잭션을 다시 확인한다(optional).
10. 노드 p의 cometBFT는 mempool 잠금을 해제하여 새로운 트랜잭션을 수신할 수 있도록 한다.
11. 노드 p는 높이 h+1에 대한 합의를 0번쨰 라운드부터 시작한다.

### [Commit](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#commit)

어플리케이션의 상태가 실제로 변경되며, cometBFT는 어플리케이션에 트랜잭션의 결과를 블록체인 상태로 확정할 것을 요청한다.

> Commit의 결과로 CommitResponse.retain_height를 반환하는데, 해당 값을 기준으로 네트워크 내 과거 블록이 제거될 수 있으므로 주의해야 한다. 기본 값은 0으로 되어있다.

<br>

## Mempool 

### [CheckTx](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#checktx)

cometBFT가 어플리케이션으로 호출하는 메소드로, 트랜잭션이 mempool에 추가되기 전에 유효성을 검증하기 위해 사용된다. 상태를 저장하지 않는 서명 확인이나 상태를 저장하는 잔액 확인 모두 가능하며, 검증 유형은 어플리케이션에 따라 달라질 수 있다.

CheckTx의 결과로 CheckTxResponse.Code != 0일 경우, mempool에 추가되지 않을 뿐 아니라 다른 노드로 브로드캐스팅 되거나 다른 블록 제안에 포함되지 않는다. 블록에 대해 Commit이 호출된 이후라도 mempool에 존재하는 비정상적인 트랜잭션들에 대해 re-CheckTx가 호출될 수도 있다.

<br>

## Info

### [Info](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#info)

어플리케이션의 기동 또는 복구 과정에서 진행되는 handshake 시점에 cometBFT와 어플리케이션을 동기화 하기 위해 어플리케이션의 상태 정보를 확인할 때 사용한다. 
- InfoResponse.app_version은 블록 내 모든 헤더에 포함된다.
- InfoResponse.last_block_app_hash, InfoResponse.last_block_height는 Commit 호출 과정에서 업데이트된다.

### [Query](https://github.com/cometbft/cometbft/blob/main/spec/abci/abci++_methods.md#query)

과거 특정 높이(h)에서의 어플리케이션 상태 정보를 조회한다. QueryRequest.prove == true인 경우 QueryResponse.proof_ops 값을 통해 선택적으로 머클증명을 반환하여 해당 높이(h)에서의 app_hash를 검증할 때 사용할 수 있다.

<br>

## State-sync

새로운 노드가 상태머신(어플리케이션)의 스냅샷을 통해 빠르게 부트스트랩할 수 있도록 한다. 

새로운 노드로부터 스냅샷을 요청받은 노드의 cometBFT는 어플리케이션으로 ListSnapShots를 호출하여 사용 가능한 스냅샷의 메타데이터(높이, 검증 데이터 등) 목록을 반환한다. 새로운 노드는 OfferSnapshot을 스냅샷을 제공한 노드에 호출하여 목록에 있는 스냅샷 중 하나를 선택하고, 유효성을 확인한 후 연결된 로컬 어플리케이션에 제공한다. 

스냅샷은 작은 조각(chunk)으로 분할되며, 조각을 제공하는 노드는 LoadSnapshotChunk를 통해 로컬 어플리케이션에서 조각을 가져와 새로운 노드로 전달한다. 새로운 노드가 조각을 수신하면 ApplySnapshotChunk를 통해 로컬 어플리케이션에 순차적으로 조각을 적용한다. 모든 조각이 수신(적용)되면 Info를 통해 어플리케이션의 AppHash를 반환하고, cometBFT는 로컬 어플리케이션의 AppHash와 Light Client Verification을 통해 확인된 블록체인 내 저장되어있는 AppHash와 비교한다.

- ListSnapshots: 사용 가능한 스냅샷의 목록을 반환한다.
- OfferSnapshot: cometBFT가 스냅샷을 어플리케이션에 제공한다.
- LoadSnapshotChunk: cometBFT가 어플리케이션에서 스냅샷 조각(chunk)를 검색하고 이를 피어(새로운 노드)에 전달한다.
- ApplySnapshotChunk: 스냅샷 조각을 전달받은 피어(새로운 노드)의 cometBFT가 조각을 어플리케이션에 전달한다

<br>

## Proposal Timeout

[PrepareProposal](#prepareproposal)이 진행되는 동안 cometBFT는 일시적으로 동작하지 않는다. 그렇기 때문에 어플리케이션에서 제안을 준비하는데 시간이 오래걸리면, 충분하지 않은 timeout이 설정되었을 경우 PrepareProposal을 완료하지 못하고 새로운 라운드를 요구할 수 있다. timeout은 해당 높이에서 라운드가 증가할때마다 자동으로 증가하며, 결국에는 PrepareProposal의 실행을 완료할 수 있을만큼 충분히 길어질 수 있다. 하지만 이 방식은 성능 저하를 발생시킬 수 있기 때문에, cometBFT에서 적절한 TimeoutPropose 값을 초기 세팅해두는 것이 좋다.

