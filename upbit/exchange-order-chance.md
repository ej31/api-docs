# 주문 가능 정보 조회

## 개요
마켓별 주문 가능 정보를 확인하는 API입니다. 매수/매도 수수료, 계좌 상태, 주문 타입 등의 정보를 제공합니다.

## API 정보

### HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **URL**: `https://api.upbit.com/v1/orders/chance`

### 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| market | String | 필수 | 마켓 ID (예: KRW-BTC) |

### 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| bid_fee | Number | 매수 수수료 비율 |
| ask_fee | Number | 매도 수수료 비율 |
| market | Object | 마켓 정보 |
| market.id | String | 마켓 ID |
| market.name | String | 마켓 이름 |
| market.order_types | Array | 주문 타입 목록 (만료됨) |
| bid_account | Object | 매수 시 화폐 계좌 상태 |
| ask_account | Object | 매도 시 화폐 계좌 상태 |
| ask_types | Array | 매도 주문 타입 |
| bid_types | Array | 매수 주문 타입 |

### 계좌 정보 (bid_account, ask_account)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| currency | String | 화폐 코드 |
| balance | NumberString | 보유 잔고 |
| locked | NumberString | 주문 중 잠긴 금액 |
| avg_buy_price | NumberString | 매수 평균가 |
| avg_buy_price_modified | Boolean | 매수 평균가 수정 여부 |
| unit_currency | String | 평단가 기준 화폐 |

## 주의사항

1. **신규 필드 사용 권장**: `market.order_types` 필드는 만료되었으므로 새로운 `ask_types`, `bid_types` 필드를 사용하시기 바랍니다.

2. **주문 타입 확인**: 각 마켓별로 지원하는 주문 타입이 다를 수 있으므로 주문 전에 반드시 확인해야 합니다.

3. **잔고 확인**: 주문 전 충분한 잔고가 있는지 확인이 필요합니다.

## 요청 예시

```bash
curl -X GET "https://api.upbit.com/v1/orders/chance?market=KRW-BTC" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

## 응답 예시

```json
{
  "bid_fee": "0.0005",
  "ask_fee": "0.0005",
  "market": {
    "id": "KRW-BTC",
    "name": "BTC/KRW",
    "order_types": ["limit", "price", "market"],
    "order_sides": ["ask", "bid"],
    "bid": {
      "currency": "KRW",
      "price_unit": null,
      "min_total": 5000
    },
    "ask": {
      "currency": "BTC", 
      "price_unit": null,
      "min_total": 5000
    },
    "max_total": "100000000.00000000",
    "state": "active"
  },
  "bid_account": {
    "currency": "KRW",
    "balance": "0.0",
    "locked": "0.0",
    "avg_buy_price": "0",
    "avg_buy_price_modified": false,
    "unit_currency": "KRW"
  },
  "ask_account": {
    "currency": "BTC",
    "balance": "10.0",
    "locked": "0.0", 
    "avg_buy_price": "8042000",
    "avg_buy_price_modified": false,
    "unit_currency": "KRW"
  }
}
```