# 취소 후 재주문

## 개요
기존 주문을 취소하고 같은 마켓과 주문 종류로 새로운 주문을 하나의 요청으로 처리하는 기능을 제공합니다.

## HTTP 메서드와 엔드포인트
- **메서드**: POST
- **엔드포인트**: `https://api.upbit.com/v1/orders/cancel_and_new`
- **Content-Type**: `application/json`

## 요청 파라미터

### 필수 파라미터
| 파라미터 | 타입 | 설명 |
|---------|-----|------|
| `prev_order_uuid` | String | 취소할 주문의 UUID |
| `prev_order_identifier` | String | 취소할 주문의 사용자 정의 식별자 |
| `new_ord_type` | String | 새 주문의 주문 타입 |
| `new_volume` | NumberString 또는 `remain_only` | 새 주문의 수량 |
| `new_price` | NumberString | 새 주문의 가격 |

*`prev_order_uuid` 또는 `prev_order_identifier` 중 하나는 필수입니다.

### 새 주문 타입 (new_ord_type)
| 값 | 설명 | 필수 추가 파라미터 |
|----|------|-------------------|
| `limit` | 지정가 주문 | `new_price` |
| `price` | 시장가 주문(매수) | `new_price` |
| `market` | 시장가 주문(매도) | - |
| `best` | 최유리 주문 | `new_price` |

### 선택적 파라미터
| 파라미터 | 타입 | 설명 |
|---------|-----|------|
| `new_identifier` | String | 새 주문의 사용자 정의 식별자 |
| `new_time_in_force` | String | IOC/FOK 주문 설정 (`ioc`, `fok`) |

### 특별 파라미터
- `new_volume`에 `"remain_only"` 문자열을 사용하면 취소된 주문의 남은 수량으로 새 주문을 생성합니다.

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `uuid` | String | 취소된 주문의 UUID |
| `side` | String | 주문 종류 |
| `market` | String | 마켓의 유일키 |
| `new_order_uuid` | String | 새 주문의 UUID |
| `cancelled_order` | Object | 취소된 주문의 상세 정보 |
| `new_order` | Object | 새 주문의 상세 정보 |

### cancelled_order 객체
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `uuid` | String | 취소된 주문의 UUID |
| `side` | String | 주문 종류 |
| `ord_type` | String | 주문 방식 |
| `price` | NumberString | 주문 가격 |
| `state` | String | 주문 상태 (cancel) |
| `market` | String | 마켓 ID |
| `created_at` | String | 주문 생성 시간 |
| `volume` | NumberString | 원래 주문 수량 |
| `remaining_volume` | NumberString | 취소 시점 남은 수량 |
| `executed_volume` | NumberString | 체결된 수량 |

### new_order 객체
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `uuid` | String | 새 주문의 UUID |
| `side` | String | 주문 종류 |
| `ord_type` | String | 주문 방식 |
| `price` | NumberString | 주문 가격 |
| `state` | String | 주문 상태 |
| `market` | String | 마켓 ID |
| `created_at` | String | 주문 생성 시간 |
| `volume` | NumberString | 새 주문 수량 |
| `remaining_volume` | NumberString | 남은 주문 수량 |

## 응답 예시

```json
{
  "uuid": "12345678-1234-1234-1234-123456789012",
  "side": "bid",
  "market": "KRW-BTC",
  "new_order_uuid": "87654321-4321-4321-4321-210987654321",
  "cancelled_order": {
    "uuid": "12345678-1234-1234-1234-123456789012",
    "side": "bid",
    "ord_type": "limit",
    "price": "50000.0",
    "state": "cancel",
    "market": "KRW-BTC",
    "created_at": "2023-01-01T10:00:00+09:00",
    "volume": "0.1",
    "remaining_volume": "0.08",
    "executed_volume": "0.02",
    "reserved_fee": "3.75",
    "remaining_fee": "3.0",
    "paid_fee": "0.75",
    "locked": "4000.0",
    "trades_count": 1
  },
  "new_order": {
    "uuid": "87654321-4321-4321-4321-210987654321",
    "side": "bid",
    "ord_type": "limit",
    "price": "52000.0",
    "state": "wait",
    "market": "KRW-BTC",
    "created_at": "2023-01-01T10:00:05+09:00",
    "volume": "0.08",
    "remaining_volume": "0.08",
    "reserved_fee": "3.12",
    "remaining_fee": "3.12",
    "paid_fee": "0.0",
    "locked": "4163.12",
    "executed_volume": "0.0",
    "trades_count": 0,
    "identifier": "new-order-001"
  }
}
```

## 주의사항

### 기본 규칙
1. **Content-Type**: 요청 시 반드시 `application/json` 헤더가 필요합니다.
2. **같은 마켓**: 새 주문은 취소된 주문과 같은 마켓에서 생성됩니다.
3. **같은 주문 종류**: 새 주문은 취소된 주문과 같은 side(매수/매도)로 생성됩니다.
4. **식별자 고유성**: `new_identifier`는 `prev_order_identifier`와 달라야 합니다.

### 취소 조건
- 취소할 주문이 이미 체결되었거나 취소된 경우 실패합니다.
- 마켓이 일시 중지된 경우 실패할 수 있습니다.

### 새 주문 생성 규칙
1. **지정가/최유리 주문**: `new_price` 필수
2. **시장가 매수**: `new_price` 필수 (매수 금액)
3. **시장가 매도**: `new_volume` 필수 (매도 수량)
4. **remain_only**: 취소된 주문의 남은 수량(`remaining_volume`)으로 새 주문 생성

### 원자성 (Atomicity)
- 이 API는 원자적 연산입니다.
- 취소와 새 주문 생성 중 하나라도 실패하면 전체 작업이 실패합니다.
- 성공 시에는 기존 주문이 취소되고 새 주문이 생성됩니다.

### 수수료 및 잔고
- 새 주문 생성 시 충분한 잔고가 있어야 합니다.
- 기존 주문의 수수료와 새 주문의 수수료가 각각 계산됩니다.
- 체결되지 않은 기존 주문의 reserved fee는 해제됩니다.