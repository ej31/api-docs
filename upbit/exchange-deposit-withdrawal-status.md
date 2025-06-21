# 입출금 현황 조회

## API 개요
업비트에서 지원하는 모든 디지털 자산의 입출금 현황을 조회하는 API입니다. 각 코인별 지갑 상태, 블록 상태 등의 정보를 실시간으로 제공합니다.

## 요청 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/status/wallet`
- **인증**: 불필요 (Public API)

## 요청 파라미터
이 API는 별도의 요청 파라미터가 필요하지 않습니다.

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `currency` | String | 화폐 코드 (대문자) |
| `wallet_state` | String | 입출금 상태 |
| `block_state` | String | 블록 상태 |
| `block_height` | Integer | 블록 높이 |
| `block_updated_at` | DateString | 블록 업데이트 시각 |
| `block_elapsed_minutes` | Integer | 마지막 블록 업데이트 이후 경과 시간(분) |
| `net_type` | String | 입출금 네트워크 타입 |
| `network_name` | String | 네트워크명 |

## 상태 값 설명

### wallet_state (입출금 상태)
- `working`: 정상 (입금 및 출금 가능)
- `withdraw_only`: 출금만 가능
- `deposit_only`: 입금만 가능
- `paused`: 일시 중단 (입금 및 출금 불가)
- `unsupported`: 지원하지 않음

### block_state (블록 상태)
- `normal`: 정상
- `delayed`: 지연
- `inactive`: 비활성

## 응답 예시

```json
[
  {
    "currency": "BTC",
    "wallet_state": "working",
    "block_state": "normal",
    "block_height": 750000,
    "block_updated_at": "2023-07-01T12:00:00+09:00",
    "block_elapsed_minutes": 5,
    "net_type": "BTC",
    "network_name": "Bitcoin"
  },
  {
    "currency": "ETH",
    "wallet_state": "paused",
    "block_state": "delayed",
    "block_height": 17500000,
    "block_updated_at": "2023-07-01T11:45:00+09:00",
    "block_elapsed_minutes": 20,
    "net_type": "ETH",
    "network_name": "Ethereum"
  }
]
```

## 주의사항

### ⚠️ 데이터 정확성 경고
- **실시간 정보 아님**: 입출금 현황 데이터는 실제 서비스 상태와 다를 수 있습니다.
- **지연 가능성**: 데이터는 수 분간 지연될 수 있습니다.
- **트레이딩 전략 비권장**: 이 데이터를 트레이딩 전략에 사용하는 것은 권장하지 않습니다.

### 블록 상태 확인
- `block_elapsed_minutes`가 클 경우 블록 동기화 지연을 의미할 수 있습니다.
- `block_state`가 `delayed` 또는 `inactive`인 경우 입출금에 영향을 줄 수 있습니다.

### 네트워크별 확인
- 동일한 코인이라도 네트워크(`net_type`)별로 상태가 다를 수 있습니다.
- 예: USDT-ERC20, USDT-TRC20 등

## 사용 사례

### 입출금 가능 상태 확인
```javascript
// 특정 코인의 입출금 가능 여부 확인
const walletStatus = await fetch('/v1/status/wallet');
const btcStatus = walletStatus.find(coin => coin.currency === 'BTC');

switch(btcStatus.wallet_state) {
    case 'working':
        console.log('BTC 입출금 모두 가능');
        break;
    case 'withdraw_only':
        console.log('BTC 출금만 가능');
        break;
    case 'deposit_only':
        console.log('BTC 입금만 가능');
        break;
    case 'paused':
        console.log('BTC 입출금 일시 중단');
        break;
    case 'unsupported':
        console.log('BTC 지원하지 않음');
        break;
}
```

### 블록 동기화 상태 모니터링
```javascript
// 블록 지연 상태 확인
const delayedCoins = walletStatus.filter(coin => 
    coin.block_elapsed_minutes > 30 || coin.block_state !== 'normal'
);

delayedCoins.forEach(coin => {
    console.log(`${coin.currency}: 블록 지연 ${coin.block_elapsed_minutes}분`);
});
```

## 관련 API
- [디지털 자산 입금 정보 조회](./exchange-digital-asset-deposit-info.md)
- [입금 리스트 조회](./exchange-deposit-list.md)
- [출금 리스트 조회](./exchange-withdrawal-list.md)