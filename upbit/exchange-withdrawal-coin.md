# 디지털 자산 출금하기 (Digital Asset Withdrawal)

## 개요
디지털 자산 출금을 요청하는 API입니다. 미리 등록된 허용 지갑 주소로만 출금이 가능합니다.

## API 정보
- **HTTP Method**: POST
- **Endpoint**: `https://api.upbit.com/v1/withdraws/coin`
- **인증**: API Key 필요

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `currency` | String | 필수 | 화폐 코드 |
| `net_type` | String | 필수 | 출금 네트워크 |
| `amount` | NumberString | 필수 | 출금 수량 |
| `address` | String | 필수 | 등록된 출금 주소 |
| `secondary_address` | String | 선택 | 2차 출금 주소 (메모, 태그 등) |
| `transaction_type` | String | 선택 | 출금 유형 |

## 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `type` | String | 입출금 구분 |
| `uuid` | String | 출금 고유 ID |
| `currency` | String | 화폐 코드 |
| `net_type` | String | 출금 네트워크 |
| `txid` | String | 트랜잭션 ID |
| `state` | String | 출금 상태 |
| `created_at` | DateString | 출금 생성 시간 |
| `done_at` | DateString | 출금 완료 시간 |
| `amount` | NumberString | 출금 수량 |
| `fee` | NumberString | 출금 수수료 |
| `krw_amount` | NumberString | 원화 환산 가격 |
| `transaction_type` | String | 출금 유형 |
| `is_cancelable` | Boolean | 출금 취소 가능 여부 |

## 응답 예시

```json
{
  "type": "withdraw",
  "uuid": "35a4f1dc-1db5-4d6b-89b5-7ec137875956",
  "currency": "BTC",
  "net_type": "BTC",
  "txid": null,
  "state": "WAITING",
  "created_at": "2019-02-28T15:17:51+09:00",
  "done_at": null,
  "amount": "0.01",
  "fee": "0.0005",
  "krw_amount": "365000.0",
  "transaction_type": "default",
  "is_cancelable": true
}
```

## 중요 주의사항

### 🔒 보안 요구사항
- **허용 주소만 사용 가능**: 미리 등록된 출금 허용 주소로만 출금할 수 있습니다.
- **업비트 회원 지갑 주소 사용**: 업비트 회원의 지갑 주소를 사용해야 합니다.
- **출금 주소 사전 등록 필요**: 출금하기 전에 반드시 허용 주소를 등록해야 합니다.

### 📋 사전 요구사항
1. 업비트 웹사이트에서 출금 주소 등록
2. 출금 가능 정보 API로 출금 가능 여부 확인
3. 최소 출금 수량 및 수수료 확인

### ⚠️ 주의사항
- 네트워크가 일치하지 않으면 자산 손실이 발생할 수 있습니다.
- 출금 처리 시간은 네트워크 상황에 따라 달라질 수 있습니다.
- 일부 자산의 경우 `secondary_address`(메모, 태그)가 필요합니다.
- 출금 수수료는 별도로 차감됩니다.