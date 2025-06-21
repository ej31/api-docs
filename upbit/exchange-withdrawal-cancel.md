# 디지털 자산 출금 취소 접수 (Digital Asset Withdrawal Cancellation)

## 개요
디지털 자산 출금 취소를 요청하는 API입니다. 출금 UUID를 통해 출금을 취소할 수 있습니다.

## API 정보
- **HTTP Method**: DELETE
- **Endpoint**: `https://api.upbit.com/v1/withdraws/coin`
- **인증**: API Key 필요

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `uuid` | String | 필수 | 출금 UUID |

## 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `type` | String | 입출금 구분 |
| `uuid` | String | 출금 고유 ID |
| `currency` | String | 화폐 코드 |
| `net_type` | String | 출금 네트워크 |
| `txid` | String | 트랜잭션 ID |
| `state` | String | 출금 상태 (취소 시 "CANCELLED") |
| `created_at` | DateString | 출금 생성 시간 |
| `done_at` | DateString | 출금 완료 시간 |
| `amount` | NumberString | 출금 수량 |
| `fee` | NumberString | 출금 수수료 |
| `transaction_type` | String | 출금 유형 (default/internal) |
| `is_cancelable` | Boolean | 출금 취소 가능 여부 |

## 응답 예시

```json
{
  "type": "withdraw",
  "uuid": "35a4f1dc-1db5-4d6b-89b5-7ec137875956",
  "currency": "BTC",
  "net_type": "BTC",
  "txid": null,
  "state": "CANCELLED",
  "created_at": "2019-02-28T15:17:51+09:00",
  "done_at": "2019-02-28T15:25:33+09:00",
  "amount": "0.01",
  "fee": "0.0005",
  "transaction_type": "default",
  "is_cancelable": false
}
```

## 출금 취소 조건

### ✅ 취소 가능한 경우
- 출금 상태가 `WAITING` (대기 중)인 경우
- `is_cancelable` 필드가 `true`인 경우
- 블록체인 네트워크로 전송되기 전인 경우

### ❌ 취소 불가능한 경우
- 출금 상태가 `PROCESSING` (처리 중) 이상인 경우
- 이미 블록체인 네트워크로 전송된 경우
- `is_cancelable` 필드가 `false`인 경우

## 취소 프로세스

### 1단계: 출금 상태 확인
출금 취소 전에 개별 출금 조회 API로 현재 상태를 확인합니다.

```bash
GET /v1/withdraw?uuid={withdrawal_uuid}
```

### 2단계: 취소 요청
`is_cancelable`이 `true`인 경우에만 취소를 요청합니다.

```bash
DELETE /v1/withdraws/coin?uuid={withdrawal_uuid}
```

### 3단계: 취소 결과 확인
응답에서 `state` 필드가 `CANCELLED`로 변경되었는지 확인합니다.

## 주의사항

### ⚠️ 취소 제한사항
- **유효한 UUID 필요**: 정확한 출금 UUID가 필요합니다.
- **시간 제한**: 출금 처리가 시작되면 취소할 수 없습니다.
- **원화 출금**: 원화 출금은 취소를 지원하지 않습니다.

### 💡 권장사항
- 출금 요청 후 즉시 취소가 필요한 경우 빠르게 처리하세요.
- 출금 전에 주소와 수량을 다시 한 번 확인하세요.
- 네트워크 상황에 따라 취소 가능 시간이 달라질 수 있습니다.

### 🔄 취소 후 처리
- 취소된 출금 수량은 즉시 계좌로 복구됩니다.
- 출금 수수료는 차감되지 않습니다.
- 취소 완료 후 `done_at` 필드에 취소 시간이 기록됩니다.