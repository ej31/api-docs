# 주문하기

## 개요
주문 요청을 합니다. 다양한 주문 타입을 지원하며, 지정가, 시장가, 최유리 주문 등을 사용할 수 있습니다.

## HTTP 메서드와 엔드포인트
- **메서드**: POST
- **엔드포인트**: `https://api.upbit.com/v1/orders`

## 요청 파라미터

### 필수 파라미터
| 파라미터 | 타입 | 설명 |
|---------|-----|------|
| `market` | String | 마켓 ID (예: KRW-BTC) |
| `side` | String | 주문 종류 (`bid`: 매수, `ask`: 매도) |
| `ord_type` | String | 주문 타입 |
| `volume` | NumberString | 주문량 |
| `price` | NumberString | 주문 가격 |

### 주문 타입 (ord_type)
| 값 | 설명 | 필수 파라미터 |
|----|------|-------------|
| `limit` | 지정가 주문 | `market`, `side`, `volume`, `price` |
| `price` | 시장가 주문(매수) | `market`, `side`, `price` |
| `market` | 시장가 주문(매도) | `market`, `side`, `volume` |
| `best` | 최유리 주문 | `market`, `side`, `volume`, `price` |

### 선택적 파라미터
| 파라미터 | 타입 | 설명 |
|---------|-----|------|
| `identifier` | String | 사용자 정의 고유값 (각 주문마다 고유해야 함) |
| `time_in_force` | String | IOC/FOK 주문 설정 (`ioc`, `fok`) |

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
| `locked` | NumberString | 거래에 사용중인 비용 |
| `executed_volume` | NumberString | 체결된 양 |
| `trades_count` | Integer | 해당 주문에 걸린 체결 수 |
| `identifier` | String | 사용자 정의 식별자 |

## 응답 예시

### 지정가 매수 주문
```json
{
  "uuid": "cdd92199-2897-4e14-9448-f923320408ad",
  "side": "bid",
  "ord_type": "limit",
  "price": "100.0",
  "state": "wait",
  "market": "KRW-BTC",
  "created_at": "2018-04-10T15:42:23+09:00",
  "volume": "0.01000000",
  "remaining_volume": "0.01000000",
  "reserved_fee": "0.0015",
  "remaining_fee": "0.0015",
  "paid_fee": "0.0",
  "locked": "1.0015",
  "executed_volume": "0.0",
  "trades_count": 0,
  "identifier": "my-order-001"
}
```

### 시장가 매수 주문
```json
{
  "uuid": "d2c4b56a-1c3e-4c4e-9f8b-1234567890ab",
  "side": "bid",
  "ord_type": "price",
  "price": "50000.0",
  "state": "wait",
  "market": "KRW-BTC",
  "created_at": "2018-04-10T15:45:00+09:00",
  "volume": null,
  "remaining_volume": null,
  "reserved_fee": "37.5",
  "remaining_fee": "37.5",
  "paid_fee": "0.0",
  "locked": "50037.5",
  "executed_volume": "0.0",
  "trades_count": 0,
  "identifier": "my-market-buy-001"
}
```

## 주의사항

### KRW 마켓 가격 단위 규칙
- **2백만원 이상**: 1,000원 단위
- **200만원 ~ 100만원**: 500원 단위
- **100만원 ~ 50만원**: 100원 단위
- **50만원 ~ 10만원**: 50원 단위
- **10만원 ~ 5만원**: 10원 단위
- **5만원 ~ 1만원**: 5원 단위
- **1만원 ~ 1천원**: 1원 단위
- **1천원 미만**: 0.1원 단위

### 주문 타입별 특징
1. **지정가 주문 (`limit`)**:
   - 지정한 가격에서만 체결
   - `volume`과 `price` 모두 필요

2. **시장가 매수 (`price`)**:
   - 현재 시장 가격으로 즉시 매수
   - `price`만 지정 (매수할 금액)

3. **시장가 매도 (`market`)**:
   - 현재 시장 가격으로 즉시 매도
   - `volume`만 지정 (매도할 수량)

4. **최유리 주문 (`best`)**:
   - 호가창 최상단 가격으로 주문
   - `volume`과 `price` 모두 필요

### 일반 주의사항
1. **식별자 고유성**: `identifier`는 각 주문마다 고유해야 합니다.
2. **Content-Type**: 요청 시 `application/json` 헤더 필요.
3. **주문 한도**: 계정별, 마켓별 주문 한도가 적용됩니다.
4. **시장 상황**: 마켓이 일시 중지된 경우 주문이 실패할 수 있습니다.
5. **잔고 확인**: 주문 전 충분한 잔고가 있는지 확인해야 합니다.
6. **IOC/FOK 주문**: `time_in_force` 설정 시 즉시 체결되지 않으면 취소됩니다.