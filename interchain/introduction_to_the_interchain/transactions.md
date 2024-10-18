# Transactions

## Transaction process

1. Decide
    - 지갑 또는 어플리케이션과 같은 사용자 인터페이스를 통해 발생하며, 트랜잭션에 포함시킬 메시지(Msg)를 결정한다.
2. Generate
    - cosmos-sdk의 TxBuilder를 사용하여 트랜잭션을 생성한다. 
3. Sign
    - 트랜잭션을 서명한다.
    - 트랜잭션은 validator가 블록에 트랜잭션을 포함시키기 전 서명되어야 한다.
4. Broadcast
    - 서명된 트랜잭션을 validator가 받을 수 있도록 브로드캐스팅한다.

<br>

## Transaction objects

트랜잭션 객체는 cosmos-sdk의 Tx 인터페이스에 구현되어있으며, 아래와 같은 메소드를 포함한다.

```go
// Tx defines an interface a transaction must fulfill.
Tx interface {
	HasMsgs

	// GetMsgsV2 gets the transaction's messages as google.golang.org/protobuf/proto.Message's.
	GetMsgsV2() ([]protov2.Message, error)
}

// HasMsgs defines an interface a transaction must fulfill.
HasMsgs interface {
	// GetMsgs gets the all the transaction's messages.
	GetMsgs() []Msg
}

// HasValidateBasic defines a type that has a ValidateBasic method.
// ValidateBasic is deprecated and now facultative.
// Prefer validating messages directly in the msg server.
HasValidateBasic interface {
	// ValidateBasic does a simple validation check that
	// doesn't require access to any other information.
	ValidateBasic() error
}
```

- 기존에는 GetMsgs와 ValidateBasic 메소드가 Tx 인터페이스에 포함되어있었으나, v0.50.0 이후부터는 ValidateBasic이 선택적으로 적용하는 방안으로 변경되어 있다.
- HasValidateBasic 인터페이스에서 말하는 msg server는 트랜잭션 내 메시지를 처리하고 검증하는 서버측 로직을 의미하는데, 최신 cosmos-sdk에서는 msg server가 트랜잭션 내에 포함될 개별 메시지에 대한 검증을 수행함으로써 유연하고 모듈화된 방식으로 처리되도록 권장하고 있다.

```
예) 사용자가 토큰 전송을 요청하는 경우,

1. MsgSend라는 메시지가 포함된 트랜잭션을 생성한다.
2. 이 트랜잭션이 메시지 서버로 전달되면, 서버는 메시지 내부에서 필수 필드(예: 보낸 사람 주소, 받는 사람 주소, 금액)가 제대로 설정되어 있는지 검증한다.
3. 검증이 통과되면 트랜잭션이 처리되고, 그렇지 않으면 거부된다.
```

cosmos-sdk에서 트랜잭션은 TxBuilder 인터페이스를 통해 관리된다.

## Messages

## Signing Transactions

트랜잭션에 포함되는 모든 메시지들은 GetSigners를 통해 반환된 주소들로 서명되어야 한다. 

```go
Msg interface {
	proto.Message

	// ValidateBasic does a simple validation check that
	// doesn't require access to any other information.
	ValidateBasic() error

	// Signers returns the addrs of signers that must sign.
	// CONTRACT: All signatures must be present to be valid.
	// CONTRACT: Returns addrs in some deterministic order.
	GetSigners() []AccAddress
}
```