# 종료된 주문 (Closed Order) 조회

## 개요
종료된 주문 리스트를 조회하는 API입니다. 체결 완료되거나 취소된 주문들의 내역을 확인할 수 있습니다.

## API 정보

### HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **URL**: `https://api.upbit.com/v1/orders/closed`

### 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| market | String | 선택 | 마켓 ID |
| state | String | 선택 | 주문 상태 ('done' 또는 'cancel') |
| states[] | Array[String] | 선택 | 주문 상태 목록 (기본값: ['done', 'cancel']) |
| start_time | String | 선택 | 조회 시작 시각 (ISO 8601 형식) |
| end_time | String | 선택 | 조회 종료 시각 (ISO 8601 형식) |
| limit | Number | 선택 | 요청 개수 (기본값: 100, 최대: 1,000) |
| order_by | String | 선택 | 정렬 방식 ('asc' 또는 'desc') |

### 응답 필드 (배열)

각 주문 객체는 다음 필드들을 포함합니다:

| 필드명 | 타입 | 설명 |
|--------|------|------|
| uuid | String | 주문의 고유 아이디 |
| side | String | 주문 종류 (ask: 매도, bid: 매수) |
| ord_type | String | 주문 방식 |
| price | NumberString | 주문 당시 화폐 가격 |
| state | String | 주문 상태 (done: 체결 완료, cancel: 취소) |
| market | String | 마켓 ID |
| created_at | String | 주문 생성 시각 |
| volume | NumberString | 사용자가 입력한 주문 양 |
| remaining_volume | NumberString | 체결 후 남은 주문 양 |
| reserved_fee | NumberString | 수수료로 예약된 비용 |
| remaining_fee | NumberString | 남은 수수료 |
| paid_fee | NumberString | 사용된 수수료 |
| locked | NumberString | 거래에 사용중인 비용 |
| executed_volume | NumberString | 체결된 양 |
| trades_count | Number | 해당 주문에 걸린 체결 수 |
| identifier | String | 조회용 사용자 지정 값 |

### 주문 상태 (state)

| 상태 | 설명 |
|------|------|
| done | 전체 체결 완료 |
| cancel | 주문 취소 |

### 주문 종류 (side)

| 종류 | 설명 |
|------|------|
| ask | 매도 |
| bid | 매수 |

### 정렬 방식 (order_by)

| 방식 | 설명 |
|------|------|
| asc | 오름차순 (생성 시간 기준) |
| desc | 내림차순 (생성 시간 기준) |

## 주의사항

1. **기본 조회 범위**: 기본값으로 최근 7일 이내의 종료된 주문 100개를 응답합니다.

2. **최대 조회 범위**: 최대 7일 범위 내의 주문만 조회할 수 있습니다.

3. **시장가 주문**: 시장가 주문은 체결 후 상태가 'cancel' 또는 'done'일 수 있습니다.

4. **최대 조회 개수**: 한 번에 최대 1,000개의 주문을 조회할 수 있습니다.

5. **시간 범위**: `start_time`과 `end_time`을 지정할 때는 7일을 초과하지 않도록 주의해야 합니다.

## 요청 예시

### 기본 조회 (최근 7일간 종료된 주문)
```bash
curl -X GET "https://api.upbit.com/v1/orders/closed" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 특정 마켓의 체결 완료 주문만 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/closed?market=KRW-BTC&state=done&limit=50" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 특정 시간 범위 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/closed?start_time=2023-01-01T00:00:00%2B09:00&end_time=2023-01-07T23:59:59%2B09:00&limit=200" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 여러 상태의 주문 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/closed?states[]=done&states[]=cancel&order_by=asc" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

## 응답 예시

```json
[
  {
    "uuid": "9ca023a5-851b-4fec-9f0a-48cd83c2eaae",
    "side": "ask",
    "ord_type": "limit",
    "price": "4280000.0",
    "state": "done",
    "market": "KRW-BTC",
    "created_at": "2018-04-10T15:42:23+09:00",
    "volume": "0.01000000",
    "remaining_volume": "0.00000000",
    "reserved_fee": "0.0",
    "remaining_fee": "0.0",
    "paid_fee": "2.14",
    "locked": "0.0",
    "executed_volume": "0.01000000",
    "trades_count": 1,
    "identifier": null
  },
  {
    "uuid": "ebe6c91d-48a5-4c5e-9d9a-9b5c8a5c1c5e",
    "side": "bid",
    "ord_type": "limit",
    "price": "4100000.0",
    "state": "cancel",
    "market": "KRW-BTC",
    "created_at": "2018-04-10T14:30:15+09:00",
    "volume": "0.02000000",
    "remaining_volume": "0.02000000",
    "reserved_fee": "0.0",
    "remaining_fee": "0.0",
    "paid_fee": "0.0",
    "locked": "0.0",
    "executed_volume": "0.00000000",
    "trades_count": 0,
    "identifier": "my_order_001"
  }
]
```

## 시간 형식 안내

시간은 ISO 8601 형식을 사용하며, 한국 시간(KST) 기준으로 다음과 같이 표현합니다:

```
2023-01-01T00:00:00+09:00
```

URL 인코딩 시:
```
2023-01-01T00%3A00%3A00%2B09%3A00
```