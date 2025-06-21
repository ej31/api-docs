# 전체 출금 조회 (Withdrawal List Inquiry)

## 개요
사용자의 출금 내역을 조회하는 API입니다. 다양한 조건으로 출금 내역을 필터링하여 조회할 수 있습니다.

## API 정보
- **HTTP Method**: GET
- **Endpoint**: `https://api.upbit.com/v1/withdraws`
- **인증**: API Key 필요

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `currency` | String | 선택 | 화폐 코드 |
| `state` | String | 선택 | 출금 상태 |
| `uuids[]` | Array | 선택 | 출금 UUID 목록 |
| `txids[]` | Array | 선택 | 출금 트랜잭션 ID 목록 |
| `limit` | Number | 선택 | 개수 제한 (기본값: 100, 최대: 100) |
| `page` | Number | 선택 | 페이지 번호 (기본값: 1) |
| `order_by` | String | 선택 | 정렬 순서 (asc/desc, 기본값: desc) |

### 출금 상태 (state) 값
- `WAITING`: 대기 중
- `PROCESSING`: 처리 중
- `DONE`: 완료
- `FAILED`: 실패
- `CANCELLED`: 취소됨
- `REJECTED`: 거부됨

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

## 응답 예시

```json
[
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
]
```

## 주의사항
- 최대 100건까지 조회 가능합니다.
- 페이지네이션을 사용하여 더 많은 데이터를 조회할 수 있습니다.
- 출금 상태에 따라 `done_at` 필드가 null일 수 있습니다.
- 취소 가능한 출금은 `is_cancelable` 필드가 true입니다.