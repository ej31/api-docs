# 개별 주문 조회

## 개요
주문 UUID를 통해 개별 주문건을 조회하는 API입니다.

## API 정보

### HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **URL**: `https://api.upbit.com/v1/order`

### 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| uuid | String | 선택 | 주문 UUID |
| identifier | String | 선택 | 조회용 사용자 지정 값 |

> **중요**: `uuid` 혹은 `identifier` 둘 중 하나의 값이 반드시 포함되어야 합니다.

### 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| uuid | String | 주문의 고유 아이디 |
| side | String | 주문 종류 (ask: 매도, bid: 매수) |
| ord_type | String | 주문 방식 (limit: 지정가, price: 시장가 매수, market: 시장가 매도) |
| price | NumberString | 주문 당시 화폐 가격 |
| state | String | 주문 상태 (wait: 체결 대기, watch: 예약 주문 대기, done: 전체 체결 완료, cancel: 주문 취소) |
| market | String | 마켓의 유일키 |
| created_at | String | 주문 생성 시간 |
| volume | NumberString | 사용자가 입력한 주문 양 |
| remaining_volume | NumberString | 체결 후 남은 주문 양 |
| reserved_fee | NumberString | 수수료로 예약된 비용 |
| remaining_fee | NumberString | 남은 수수료 |
| paid_fee | NumberString | 사용된 수수료 |
| locked | NumberString | 거래에 사용중인 비용 |
| executed_volume | NumberString | 체결된 양 |
| trades_count | Number | 해당 주문에 걸린 체결 수 |

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

### 주문 방식 (ord_type)

| 방식 | 설명 |
|------|------|
| limit | 지정가 주문 |
| price | 시장가 주문 (매수) |
| market | 시장가 주문 (매도) |

## 주의사항

1. **필수 파라미터**: `uuid` 또는 `identifier` 중 하나는 반드시 포함되어야 합니다.

2. **주문 상태 확인**: 주문 처리 상태를 정확히 파악하기 위해 `state` 필드를 확인해야 합니다.

3. **체결 정보**: `executed_volume`과 `remaining_volume`을 통해 부분 체결 상황을 파악할 수 있습니다.

## 요청 예시

### UUID로 조회
```bash
curl -X GET "https://api.upbit.com/v1/order?uuid=9ca023a5-851b-4fec-9f0a-48cd83c2eaae" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Identifier로 조회
```bash
curl -X GET "https://api.upbit.com/v1/order?identifier=my_order_001" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

## 응답 예시

```json
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
  "trades_count": 1
}
```