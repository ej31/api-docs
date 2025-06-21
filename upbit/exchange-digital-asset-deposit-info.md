# 디지털 자산 입금 정보 조회

## API 개요
디지털 자산의 입금 관련 정보를 조회하는 API입니다. 각 코인별 입금 가능 여부, 최소 입금량, 컨펌 수 등의 정보를 제공합니다.

## 요청 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/deposits/chance/coin`
- **인증**: JWT 토큰 필요

## 요청 파라미터
이 API는 별도의 요청 파라미터가 필요하지 않습니다.

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `currency` | String | 화폐 코드 (대문자) |
| `net_type` | String | 입금 네트워크 |
| `is_deposit_possible` | Boolean | 입금 가능 여부 |
| `deposit_impossible_reason` | String | 입금 불가 사유 |
| `minimum_deposit_amount` | Decimal | 최소 입금 수량 |
| `minimum_deposit_confirmations` | Integer | 최소 입금 컨펌 수 |
| `decimal_precision` | Integer | 입금 소수점 자릿수 |

## 응답 예시

```json
[
  {
    "currency": "BTC",
    "net_type": "BTC",
    "is_deposit_possible": true,
    "deposit_impossible_reason": null,
    "minimum_deposit_amount": "0.00050000",
    "minimum_deposit_confirmations": 1,
    "decimal_precision": 8
  },
  {
    "currency": "ETH",
    "net_type": "ETH",
    "is_deposit_possible": false,
    "deposit_impossible_reason": "네트워크 점검 중",
    "minimum_deposit_amount": "0.01000000",
    "minimum_deposit_confirmations": 12,
    "decimal_precision": 8
  }
]
```

## 주의사항

### ⚠️ 데이터 정확성 주의
- **실시간 정보 아님**: 디지털 자산 입금 정보 조회 데이터는 실제 서비스 상태와 다를 수 있습니다.
- **지연 가능성**: API 데이터는 수 분간 지연될 수 있으며, 참고용으로만 사용하시기 바랍니다.
- **공식 확인 필요**: 실제 입금 전에는 업비트 공지사항 및 지갑 상태 페이지를 반드시 확인하세요.

### 입금 시 고려사항
- **최소 입금량**: `minimum_deposit_amount` 미만으로 입금할 경우 입금이 처리되지 않을 수 있습니다.
- **컨펌 수**: `minimum_deposit_confirmations` 만큼의 블록 컨펌이 완료되어야 입금이 완료됩니다.
- **네트워크 타입**: 반드시 올바른 네트워크(`net_type`)로 입금해야 합니다.

### 입금 불가 상황
- `is_deposit_possible`이 `false`인 경우 입금이 불가능합니다.
- `deposit_impossible_reason`에서 입금 불가 사유를 확인할 수 있습니다.

## 사용 사례

### 입금 전 사전 확인
```javascript
// 입금 가능 여부 확인 예시
const depositInfo = await fetch('/v1/deposits/chance/coin');
const btcInfo = depositInfo.find(coin => coin.currency === 'BTC');

if (btcInfo.is_deposit_possible) {
    console.log(`BTC 최소 입금량: ${btcInfo.minimum_deposit_amount}`);
    console.log(`필요 컨펌 수: ${btcInfo.minimum_deposit_confirmations}`);
} else {
    console.log(`입금 불가: ${btcInfo.deposit_impossible_reason}`);
}
```

## 관련 API
- [입출금 현황 조회](./exchange-deposit-withdrawal-status.md)
- [입금 주소 조회](./exchange-deposit-address-lookup.md)
- [입금 리스트 조회](./exchange-deposit-list.md)