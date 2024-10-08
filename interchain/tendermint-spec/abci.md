# ABCI

[github](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md)

## [CheckTx](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#checktx)

#### TBC

> The CheckTx ABCI method controls what transactions are considered for inclusion in a block. When Tendermint receives a ResponseCheckTx with a non-zero Code, the associated transaction will not be added to Tendermint's mempool or it will be removed if it is already included.

Question

- 트랜잭션이 Mempool에 들어갔다는 것은 이미 유효성 검증에 통과했기 때문인데, 다시한번 유효성을 검사했을 때 잘못되는 경우가 무엇이 있을까?
- 이미 유효성 검증이 완료된 트랜잭션을 다시한번 검증하는 시기와, 이 때 mempool에 해당 트랜잭션이 이미 들어가있는 경우 이를 제거해야한다는 것은 checkTx를 통한 트랜잭션 유효성 검증이 불완전하다는 것인가?

## [Events](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#events)

BeginBlock, CheckTx, DeliverTx, EndBlock 각각의 결과(Response)에는 이벤트가 포함된다. 이벤트 메시지 구조는 아래와 같다.

```go
// type: 특정 Response 또는 Tx에 따라 이벤트를 분류하기 위한 유형 (이벤트 내 중복되어 존재할 수 있음)
// attributes: 메서드 실행 과정에 발생한 내용에 대한 메타데이터(key/value)
message Event {
  string                  type       = 1;
  repeated EventAttribute attributes = 2;
}

message EventAttribute {
  bytes key   = 1;
  bytes value = 2;
  bool  index = 3;  // nondeterministic
}
```

> Note that the set of events returned for a block from BeginBlock and EndBlock are merged. In case both methods return the same key, only the value defined in EndBlock is used.
> - BeginBlock, EndBlock은 상태 변화와 직접적인 관련은 없으나, 각 단계별로 발생한 시스템 이벤트가 존재할 수 있다. (검증자 업데이트, 보상 계산 등)
> - 두 메시지의 내용을 병합함으로써 블록에 대한 전체적인 정보를 확인할 수 있으며, 만약 동일한 키가 존재할 경우 블록 생성을 마무리하는 EndBlock의 값이 최종적으로 적용된다.

```sh
# 이벤트 생성 예시
abci.ResponseDeliverTx{
  // ...
 Events: []abci.Event{
  {
   Type: "validator.provisions",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("balance"), Value: []byte("..."), Index: true},
   },
  },
  {
   Type: "validator.provisions",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: false},
    abci.EventAttribute{Key: []byte("balance"), Value: []byte("..."), Index: false},
   },
  },
  {
   Type: "validator.slashed",
   Attributes: []abci.EventAttribute{
    abci.EventAttribute{Key: []byte("address"), Value: []byte("..."), Index: false},
    abci.EventAttribute{Key: []byte("amount"), Value: []byte("..."), Index: true},
    abci.EventAttribute{Key: []byte("reason"), Value: []byte("..."), Index: true},
   },
  },
  // ...
 },
}
```

## [EvidenceType](https://github.com/tendermint/tendermint/blob/master/spec/abci/abci.md#evidencetype)

네트워크 참여자로부터 발생하는 악의적인(비정상적) 동작을 검증하기 위하여 사용된다. 
- 악의적인 동작이 감지되면 이에 대한 증거를 다른 노드에 전달하고, 모든 검증자가 이를 확인하면 증거가 체인에 커밋된다. 
- 커밋된 증거는 ABCI를 통해 어플리케이션으로 전달된다. 해당 증거에 대한 처벌을 행사하는 것은 어플리케이션에서 진행한다.
- 증거에는 [DuplicateVoteEvidence](https://github.com/tendermint/tendermint/blob/master/spec/core/data_structures.md#duplicatevoteevidence)와 [LightClientAttackEvidence](https://github.com/tendermint/tendermint/blob/master/spec/core/data_structures.md#lightclientattackevidence)이 있다.

## Determinism