# ID로 주문 리스트 조회

## 개요
UUID 또는 identifier로 주문 리스트를 조회하는 API입니다. 최대 100개의 주문을 한 번에 조회할 수 있습니다.

## API 정보

### HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **URL**: `https://api.upbit.com/v1/orders/uuids`

### 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| market | String | 선택 | 마켓 ID |
| uuids[] | Array[String] | 선택 | 주문 UUID 목록 (최대 100개) |
| identifiers[] | Array[String] | 선택 | 주문 identifier 목록 (최대 100개) |
| order_by | String | 선택 | 정렬 방식 (asc: 오름차순, desc: 내림차순, 기본값: desc) |

> **중요**: `uuids[]` 또는 `identifiers[]` 중 한 가지 필드는 필수이며, 두 가지 필드를 함께 사용할 수 없습니다.

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
| wait | 체결 대기 |
| watch | 예약 주문 대기 |
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
| desc | 내림차순 (생성 시간 기준, 기본값) |

## 주의사항

1. **필수 파라미터**: `uuids[]` 또는 `identifiers[]` 중 하나는 반드시 포함되어야 합니다.

2. **동시 사용 불가**: `uuids[]`와 `identifiers[]`를 함께 사용할 수 없습니다.

3. **최대 조회 개수**: 한 번에 최대 100개의 주문만 조회할 수 있습니다.

4. **배열 응답**: 응답은 주문 객체들의 배열 형태입니다.

## 요청 예시

### UUID 배열로 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/uuids?uuids[]=9ca023a5-851b-4fec-9f0a-48cd83c2eaae&uuids[]=ebe6c91d-48a5-4c5e-9d9a-9b5c8a5c1c5e&order_by=desc" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Identifier 배열로 조회
```bash
curl -X GET "https://api.upbit.com/v1/orders/uuids?identifiers[]=my_order_001&identifiers[]=my_order_002&market=KRW-BTC" \
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
    "identifier": "my_order_001"
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
    "identifier": "my_order_002"
  }
]
```