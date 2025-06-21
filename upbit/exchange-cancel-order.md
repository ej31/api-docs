# 주문 취소 접수

## 개요
- UUID를 사용하여 주문을 취소합니다
- 주문 UUID 또는 사용자 정의 식별자를 통해 취소가 가능합니다

## HTTP 메서드와 엔드포인트
- **메서드**: DELETE
- **엔드포인트**: `https://api.upbit.com/v1/order`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|-----|-----|------|
| `uuid` | String | 선택* | 취소할 주문의 UUID |
| `identifier` | String | 선택* | 조회용 사용자 지정값 |

*`uuid` 혹은 `identifier` 둘 중 하나의 값이 반드시 포함되어야 합니다.

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `uuid` | String | 주문의 고유 아이디 |
| `side` | String | 주문 종류 |
| `ord_type` | String | 주문 방식 |
| `price` | NumberString | 주문 당시 화폐 가격 |
| `state` | String | 주문 상태 |
| `market` | String | 마켓의 유일키 |
| `created_at` | String | 주문 생성 시간 |
| `volume` | NumberString | 사용자가 입력한 주문 양 |
| `remaining_volume` | NumberString | 체결 후 남은 주문 양 |
| `reserved_fee` | NumberString | 수수료로 예약된 비용 |
| `remaining_fee` | NumberString | 남은 수수료 |
| `paid_fee` | NumberString | 사용된 수수료 |
| `locked` | NumberString | 거래에 사용 중인 비용 |
| `executed_volume` | NumberString | 체결된 양 |
| `trades_count` | Integer | 해당 주문에 걸린 체결 수 |

## 응답 예시

```json
{
  "uuid": "9ca023a5-851b-4fec-9f0a-48cd83c2eaae",
  "side": "ask",
  "ord_type": "limit",
  "price": "4280000.0",
  "state": "wait",
  "market": "KRW-BTC",
  "created_at": "2018-04-10T15:42:23+09:00",
  "volume": "0.01",
  "remaining_volume": "0.01",
  "reserved_fee": "0.0",
  "remaining_fee": "0.0",
  "paid_fee": "0.0",
  "locked": "0.01",
  "executed_volume": "0.0",
  "trades_count": 0
}
```

## 주의사항

1. **필수 매개변수**: `uuid` 또는 `identifier` 중 하나는 반드시 포함되어야 합니다.
2. **식별자 제공**: `identifier` 필드는 2024-10-18 이후에 생성된 주문에만 제공됩니다.
3. **주문 상태**: 이미 체결되었거나 취소된 주문은 취소할 수 없습니다.
4. **시장 상황**: 마켓이 일시 중지된 경우 취소가 실패할 수 있습니다.