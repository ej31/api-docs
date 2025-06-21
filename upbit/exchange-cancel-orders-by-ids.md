# ID로 주문리스트 취소 접수

## 개요
UUID 또는 사용자 정의 식별자(identifiers)로 다수의 주문을 취소합니다.

## HTTP 메서드와 엔드포인트
- **메서드**: DELETE
- **엔드포인트**: `https://api.upbit.com/v1/orders/uuids`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|-----|-----|------|
| `uuids[]` | Array[String] | 선택* | 취소할 주문의 UUID 목록 (최대 20개) |
| `identifiers[]` | Array[String] | 선택* | 취소할 주문의 식별자 목록 (최대 20개) |

*`uuids[]` 또는 `identifiers[]` 중 하나만 사용해야 하며, 둘을 동시에 사용할 수 없습니다.

## 응답 필드

### success (성공한 취소)
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `count` | Number | 성공적으로 취소된 주문 수 |
| `orders` | Array | 취소된 주문들의 정보 |

#### orders 내 필드
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `uuid` | String | 취소된 주문의 UUID |
| `market` | String | 취소된 주문의 마켓 ID |
| `identifier` | String | 사용자 정의 식별자 |

### failed (실패한 취소)
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `count` | Number | 취소에 실패한 주문 수 |
| `orders` | Array | 취소 실패한 주문들의 정보 |

#### orders 내 필드
| 필드명 | 타입 | 설명 |
|--------|-----|------|
| `uuid` | String | 취소 실패한 주문의 UUID |
| `market` | String | 취소 실패한 주문의 마켓 ID |
| `identifier` | String | 사용자 정의 식별자 |

## 응답 예시

### UUID를 사용한 경우
```json
{
  "success": {
    "count": 2,
    "orders": [
      {
        "uuid": "12345678-1234-1234-1234-123456789012",
        "market": "KRW-BTC",
        "identifier": null
      },
      {
        "uuid": "87654321-4321-4321-4321-210987654321",
        "market": "KRW-ETH",
        "identifier": null
      }
    ]
  },
  "failed": {
    "count": 1,
    "orders": [
      {
        "uuid": "abcdefgh-abcd-abcd-abcd-abcdefghijkl",
        "market": "KRW-ADA",
        "identifier": null
      }
    ]
  }
}
```

### Identifier를 사용한 경우
```json
{
  "success": {
    "count": 1,
    "orders": [
      {
        "uuid": "12345678-1234-1234-1234-123456789012",
        "market": "KRW-BTC",
        "identifier": "my-order-001"
      }
    ]
  },
  "failed": {
    "count": 1,
    "orders": [
      {
        "uuid": "87654321-4321-4321-4321-210987654321",
        "market": "KRW-ETH",
        "identifier": "my-order-002"
      }
    ]
  }
}
```

## 주의사항

1. **매개변수 제한**: `uuids[]`와 `identifiers[]`는 동시에 사용할 수 없습니다.
2. **최대 개수**: 한 번에 최대 20개의 주문만 취소할 수 있습니다.
3. **취소 실패 사유**:
   - 주문이 이미 체결된 경우
   - 주문이 이미 취소된 경우
   - 마켓이 일시 중지된 경우
   - 존재하지 않는 UUID 또는 식별자인 경우
4. **부분 성공**: 일부 주문만 성공적으로 취소될 수 있으며, 성공한 주문과 실패한 주문이 각각 분리되어 응답됩니다.
5. **식별자 사용**: 사용자 정의 식별자는 2024-10-18 이후에 생성된 주문에만 사용할 수 있습니다.
6. **원자성 없음**: 이 API는 원자적(atomic) 연산이 아니므로, 일부 주문만 취소될 수 있습니다.