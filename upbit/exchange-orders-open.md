# 대기 주문 (Open Order) 조회

## 개요
체결 대기 상태인 주문 리스트를 조회하는 API입니다. 대기 중인 주문들의 현재 상태를 확인할 수 있습니다.

## API 정보

### HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **URL**: `https://api.upbit.com/v1/orders/open`

### 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| market | String | 선택 | 마켓 ID |
| state | String | 선택 | 주문 상태 (기본값: "wait") |
| states[] | Array[String] | 선택 | 주문 상태 목록 |
| page | Number | 선택 | 페이지 수 (기본값: 1) |
| limit | Number | 선택 | 요청 개수 (기본값: 100, 최대: 100) |
| order_by | String | 선택 | 정렬 방식 (기본값: "desc") |

### 응답 필드 (배열)

각 주문 객체는 다음 필드들을 포함합니다:

| 필드명 | 타입 | 설명 |
|--------|------|------|
| uuid | String | 주문의 고유 아이디 |
| side | String | 주문 종류 (ask: 매도, bid: 매수) |
| ord_type | String | 주문 방식 |
| price | NumberString | 주문 당시 화폐 가격 |
| state | String | 주문 상태 |
| market | String | 마켓 ID |
| created_at | String | 주문 생성 시간 |
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
| wait | 체결 대기 (기본값) |
| watch | 예약 주문 대기 |

### 주문 종류 (side)

| 종류 | 설명 |
|------|------|
| ask | 매도 |
| bid | 매수 |

### 정렬 방식 (order_by)

| 방식 | 설명 |
|------|------|
| asc | 오름차순 (생성 시간 기준) |
| desc | 내림차순 (생성 시간 기준, 기본값) |

## 주의사항

1. **대기 상태 주문**: 해당 API를 통해 대기상태(open)인 주문을 조회할 수 있습니다.

2. **페이징**: 많은 대기 주문이 있는 경우 페이징을 통해 조회해야 합니다.

3. **최대 조회 개수**: 한 번에 최대 100개의 주문만 조회할 수 있습니다.

4. **상태 필터링**: `state` 또는 `states[]` 파라미터를 통해 특정 상태의 주문만 필터링할 수 있습니다.

## 요청 예시

### 기본 조회 (모든 대기 주문)
```bash
curl -X GET "https://api.upbit.com/v1/orders/open" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 특정 마켓의 대기 주문 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/open?market=KRW-BTC&limit=50" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 여러 상태의 주문 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/open?states[]=wait&states[]=watch&page=1&limit=20" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### 페이징 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/open?page=2&limit=50&order_by=asc" \
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
    "state": "wait",
    "market": "KRW-BTC",
    "created_at": "2018-04-10T15:42:23+09:00",
    "volume": "0.01000000",
    "remaining_volume": "0.01000000",
    "reserved_fee": "2.14",
    "remaining_fee": "2.14",
    "paid_fee": "0.0",
    "locked": "0.01000000",
    "executed_volume": "0.00000000",
    "trades_count": 0,
    "identifier": null
  },
  {
    "uuid": "ebe6c91d-48a5-4c5e-9d9a-9b5c8a5c1c5e",
    "side": "bid",
    "ord_type": "limit",
    "price": "4100000.0",
    "state": "wait",
    "market": "KRW-BTC",
    "created_at": "2018-04-10T14:30:15+09:00",
    "volume": "0.02000000",
    "remaining_volume": "0.02000000",
    "reserved_fee": "41.0",
    "remaining_fee": "41.0",
    "paid_fee": "0.0",
    "locked": "82041.0",
    "executed_volume": "0.00000000",
    "trades_count": 0,
    "identifier": "my_order_001"
  }
]
```