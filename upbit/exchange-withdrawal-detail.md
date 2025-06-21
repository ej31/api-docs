# 개별 출금 조회 (Individual Withdrawal Inquiry)

## 개요
출금 UUID를 통해 개별 출금 정보를 조회하는 API입니다.

## API 정보
- **HTTP Method**: GET
- **Endpoint**: `https://api.upbit.com/v1/withdraw`
- **인증**: API Key 필요

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `uuid` | String | 선택 | 출금 UUID |
| `txid` | String | 선택 | 출금 TXID |
| `currency` | String | 선택 | 화폐 코드 |

> **참고**: `uuid`, `txid`, `currency` 중 하나는 반드시 포함되어야 합니다.

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
| `transaction_type` | String | 출금 유형 (default/internal) |
| `is_cancelable` | Boolean | 출금 취소 가능 여부 |

### 출금 상태 (state) 값
- `WAITING`: 대기 중
- `PROCESSING`: 처리 중
- `DONE`: 완료
- `FAILED`: 실패
- `CANCELLED`: 취소됨
- `REJECTED`: 거부됨

## 응답 예시

```json
{
  "type": "withdraw",
  "uuid": "35a4f1dc-1db5-4d6b-89b5-7ec137875956",
  "currency": "BTC",
  "net_type": "BTC",
  "txid": "98c15999f0bdc4ae0e7215206a26543560932272",
  "state": "DONE",
  "created_at": "2019-02-28T15:17:51+09:00",
  "done_at": "2019-02-28T15:22:12+09:00",
  "amount": "0.01",
  "fee": "0.0005",
  "transaction_type": "default",
  "is_cancelable": false
}
```

## 주의사항
- 상세한 출금 정보를 제공합니다.
- 출금 상태에 따른 포괄적인 상태 추적이 가능합니다.
- 출금이 완료되지 않은 경우 `done_at` 필드는 null입니다.