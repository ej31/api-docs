# 주문 일괄 취소 접수

## 개요
다수의 주문에 대해 일괄 취소 요청을 합니다.

## HTTP 메서드와 엔드포인트
- **메서드**: DELETE
- **엔드포인트**: `https://api.upbit.com/v1/orders/open`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 기본값 | 설명 |
|---------|-----|-----|--------|------|
| `cancel_side` | String | 선택 | `all` | 취소할 주문 종류 (`all`, `ask`, `bid`) |
| `pairs` | String | 선택 | - | 취소할 마켓 목록 (쉼표로 구분, 최대 20개) |
| `excluded_pairs` | String | 선택 | - | 제외할 마켓 목록 (쉼표로 구분, 최대 20개) |
| `quote_currencies` | String | 선택 | - | 거래 통화 목록 (쉼표로 구분) |
| `count` | Number | 선택 | 20 | 취소할 주문 개수 (최대 300개) |
| `order_by` | String | 선택 | `desc` | 정렬 순서 (`asc`: 오래된 순, `desc`: 최신 순) |

### 파라미터 옵션 상세

#### cancel_side
- `all`: 모든 주문 (기본값)
- `ask`: 매도 주문만
- `bid`: 매수 주문만

#### order_by
- `asc`: 오래된 주문부터 취소
- `desc`: 최신 주문부터 취소 (기본값)

## 응답 필드

### success (성공한 취소)
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `count` | Number | 성공적으로 취소된 주문 수 |
| `orders` | Array | 취소된 주문들의 상세 정보 |

### failed (실패한 취소)
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `count` | Number | 취소에 실패한 주문 수 |
| `orders` | Array | 취소 실패한 주문들의 상세 정보 |

## 응답 예시

```json
{
  "success": {
    "count": 2,
    "orders": [
      {
        "uuid": "order-uuid-1",
        "side": "bid",
        "ord_type": "limit",
        "price": "1000.0",
        "state": "cancel",
        "market": "KRW-BTC",
        "created_at": "2023-01-01T12:00:00+09:00",
        "volume": "0.1",
        "remaining_volume": "0.1"
      },
      {
        "uuid": "order-uuid-2",
        "side": "ask",
        "ord_type": "limit",
        "price": "2000.0",
        "state": "cancel",
        "market": "KRW-ETH",
        "created_at": "2023-01-01T12:30:00+09:00",
        "volume": "0.5",
        "remaining_volume": "0.5"
      }
    ]
  },
  "failed": {
    "count": 1,
    "orders": [
      {
        "uuid": "order-uuid-3",
        "side": "bid",
        "ord_type": "limit",
        "price": "1500.0",
        "state": "done",
        "market": "KRW-ADA",
        "created_at": "2023-01-01T11:00:00+09:00",
        "volume": "100.0",
        "remaining_volume": "0.0"
      }
    ]
  }
}
```

## 주의사항

1. **요청 방식**: 쿼리 파라미터만 지원됩니다.
2. **취소 가능 주문**: 미체결 상태(WAIT)인 주문만 취소할 수 있습니다.
3. **속도 제한**: 2초에 1회 요청으로 제한됩니다.
4. **동시 체결 위험**: 주문 취소 중에 체결이 발생할 수 있습니다.
5. **마켓 제한**: 
   - `pairs`와 `excluded_pairs`는 각각 최대 20개 마켓까지 지정 가능
   - 두 파라미터를 동시에 사용할 수 있습니다
6. **주문 개수 제한**: 한 번에 최대 300개 주문까지 취소 가능합니다.
7. **실패 사유**: 이미 체결된 주문, 이미 취소된 주문, 마켓 일시 중지 등의 경우 취소가 실패할 수 있습니다.