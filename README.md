# Upbit API 통합 문서 (v 1.5.7)

이 문서는 Upbit에서 제공하는 모든 API 문서를 하나로 통합한 내용입니다.
CLAUDE.md 혹은 cursor rule 등에서 아래 내용을 global rule 에 추가해서 사용합니다.

```
- Upbit API 및 WebSocket Interface 및 사용법은 https://raw.githubusercontent.com/ej31/upbit-api-docs/refs/heads/main/README.md 문서를 준수할 것
```

## 주의사항
- **시간대**: Upbit API의 TIME ZONE은 KST(한국 표준 시간) 기준으로 맞춰야 합니다.
- **인증**: 일부 API는 JWT 토큰 인증이 필요합니다.
- **Rate Limiting**: API 호출 제한이 있으니 주의하시기 바랍니다.
- **CustomException**: 예외 처리 시 반드시 Custom Exception을 사용하고, Exception을 상속받는 클래스로 작성하세요.

## 문서 구성
- **Exchange API**: 거래, 계좌, 입출금 관련 API (29개)
- **Quotation API**: 시세, 캔들, 현재가 관련 API (12개)  
- **WebSocket API**: 실시간 데이터 수신 API (13개)

## 목차 (Table of Contents)

### 1. Exchange API (거래소 API)
- [전체 계좌 조회](#exchange-accounts)
- [API 키 리스트 조회](#exchange-api-key-list)
- [주문 일괄 취소 접수](#exchange-bulk-cancel-orders)
- [취소 후 재주문](#exchange-cancel-and-reorder)
- [주문 취소 접수](#exchange-cancel-order)
- [ID로 주문리스트 취소 접수](#exchange-cancel-orders-by-ids)
- [입금 주소 생성 요청](#exchange-deposit-address-generate)
- [개별 입금 주소 조회](#exchange-deposit-address-lookup)
- [개별 입금 조회](#exchange-deposit-detail)
- [원화 입금하기](#exchange-deposit-krw)
- [입금 리스트 조회](#exchange-deposit-list)
- [입출금 현황 조회](#exchange-deposit-withdrawal-status)
- [디지털 자산 입금 정보 조회](#exchange-digital-asset-deposit-info)
- [주문 가능 정보 조회](#exchange-order-chance)
- [개별 주문 조회](#exchange-order-detail)
- [ID로 주문 조회](#exchange-orders-by-ids)
- [전체 미체결 주문 조회](#exchange-orders-open)
- [전체 체결 주문 조회](#exchange-orders-closed)
- [주문하기](#exchange-place-order)
- [Travel Rule 인증](#exchange-travel-rule-verification)
- [Travel Rule 거래소 리스트](#exchange-travel-rule-exchange-list)
- [Travel Rule TXID 검증](#exchange-travel-rule-txid-verification)
- [출금 가능 주소 리스트 조회](#exchange-withdrawal-addresses)
- [출금 취소](#exchange-withdrawal-cancel)
- [출금 가능 정보](#exchange-withdrawal-chance)
- [디지털 자산 출금하기](#exchange-withdrawal-coin)
- [개별 출금 조회](#exchange-withdrawal-detail)
- [원화 출금하기](#exchange-withdrawal-krw)
- [출금 리스트 조회](#exchange-withdrawal-list)

### 2. Quotation API (시세 API)
- [전체 마켓 조회](#quotation-market-all)
- [초봉 캔들 조회](#quotation-candles-seconds)
- [분봉 캔들 조회](#quotation-candles-minutes)
- [일봉 캔들 조회](#quotation-candles-days)
- [주봉 캔들 조회](#quotation-candles-weeks)
- [월봉 캔들 조회](#quotation-candles-months)
- [연봉 캔들 조회](#quotation-candles-years)
- [최근 체결 내역](#quotation-recent-trades)
- [현재가 정보](#quotation-ticker-current-price)
- [마켓별 현재가 정보](#quotation-market-current-price)
- [호가 정보 조회](#quotation-orderbook)
- [지원 레벨 조회](#quotation-supported-levels)

### 3. WebSocket API (웹소켓 API)
- [일반 정보](#websocket-general-info)
- [인증](#websocket-authentication)
- [테스트 샘플](#websocket-test-sample)
- [요청 형식](#websocket-request-format)
- [현재가](#websocket-ticker)
- [체결](#websocket-trade)
- [호가](#websocket-orderbook)
- [캔들](#websocket-candle)
- [내 주문](#websocket-myorder)
- [내 자산](#websocket-myasset)
- [구독](#websocket-subscriptions)
- [에러 코드](#websocket-error)
- [연결](#websocket-connection)

---

## Exchange API (거래소 API)

<a name="exchange-accounts"></a>
### 전체 계좌 조회

## 개요
내가 보유한 자산 리스트를 조회하는 API

## 엔드포인트
```
GET https://api.upbit.com/v1/accounts
```

## 요청 파라미터
- 없음

## 응답 필드
| 필드명 | 타입 | 설명 |
|--------|------|------|
| currency | String | 화폐 영문 코드 |
| balance | NumberString | 주문 가능 금액/수량 |
| locked | NumberString | 주문 중 묶인 금액/수량 |
| avg_buy_price | NumberString | 매수평균가 |
| avg_buy_price_modified | Boolean | 매수평균가 수정 여부 |
| unit_currency | String | 평단가 기준 화폐 |

## 응답 예시
```json
[
  {
    "currency": "KRW",
    "balance": "1000000.0",
    "locked": "0.0",
    "avg_buy_price": "0",
    "avg_buy_price_modified": false,
    "unit_currency": "KRW"
  },
  {
    "currency": "BTC",
    "balance": "0.1",
    "locked": "0.0",
    "avg_buy_price": "50000000.0",
    "avg_buy_price_modified": false,
    "unit_currency": "KRW"
  }
]
```

## 주의사항
- API 키 필요 (인증 헤더 포함)
- Open API 이용약관 준수 필요

---

<a name="exchange-api-key-list"></a>
### API 키 리스트 조회

## API 개요
계정에 등록된 Open API 키 목록과 만료 일자를 조회하는 API입니다. API 키 관리 및 모니터링을 위해 사용됩니다.

## 요청 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/api_keys`
- **인증**: JWT 토큰 필요

## 요청 파라미터
이 API는 별도의 요청 파라미터가 필요하지 않습니다.

## 응답 필드
구체적인 응답 필드는 문서에서 명시되지 않았으나, 일반적으로 다음과 같은 정보를 포함합니다:

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `access_key` | String | API 액세스 키 |
| `expire_at` | DateString | API 키 만료 일시 |
| `created_at` | DateString | API 키 생성 일시 |
| `permissions` | Array | API 키 권한 목록 |
| `allowed_ips` | Array | 허용된 IP 주소 목록 |

## 응답 예시

```json
[
  {
    "access_key": "aBcDeFgH1234567890",
    "expire_at": "2024-07-01T00:00:00+09:00",
    "created_at": "2023-07-01T10:30:00+09:00",
    "permissions": ["read", "trade"],
    "allowed_ips": ["192.168.1.100", "203.0.113.1"]
  }
]
```

## API 키 제한사항

### 키 개수 제한
- **최대 10개**: 계정당 최대 10개의 API 키 생성 가능
- **Secret Key 보안**: Secret Key는 생성 시에만 확인 가능하며, 이후에는 조회할 수 없습니다.

### 유효 기간
- **1년 유효**: API 키 토큰은 1년간 유효하며 연장 불가능
- **만료 전 재생성**: 만료 전에 새로운 API 키를 생성해야 합니다.

### 보안 요구사항
- **2차 인증**: API 키 관리 시 2단계 인증(2FA) 필요
- **사용 기능 지정**: 키 생성 시 사용할 기능을 명시해야 함
- **IP 주소 등록**: 등록된 IP 주소에서만 접근 가능

## IP 주소 제한

### 등록 가능 IP
- **최대 10개**: API 키당 최대 10개 IP 주소 등록 가능
- **사전 등록 필수**: 반드시 사전에 등록된 IP에서만 API 호출 가능
- **동적 IP 주의**: 동적 IP 환경에서는 주기적인 IP 업데이트가 필요할 수 있습니다.

## 권한 관리

### 기능별 권한
API 키 생성 시 다음과 같은 기능별 권한을 설정할 수 있습니다:
- **조회 (READ)**: 잔고, 주문 내역 등 조회 기능
- **거래 (TRADE)**: 주문 생성, 취소 등 거래 기능
- **출금 (WITHDRAW)**: 출금 요청 기능

## 주의사항

### 보안 관련
- **Secret Key 보관**: Secret Key는 안전한 곳에 보관하고 타인과 공유하지 마세요.
- **정기적 갱신**: 보안을 위해 주기적으로 API 키를 갱신하는 것을 권장합니다.
- **사용하지 않는 키 삭제**: 더 이상 사용하지 않는 API 키는 즉시 삭제하세요.

### 제한 정책 변경사항
- **v1.4.1 업데이트**: 요청 수 제한 정책이 '계정 단위'로 변경되었습니다.
- **Rate Limiting**: 계정 전체에 대한 API 호출 제한이 적용됩니다.

## 사용 사례

### API 키 만료 확인
```javascript
// API 키 만료 일자 확인
const apiKeys = await fetch('/v1/api_keys');
const soonToExpire = apiKeys.filter(key => {
    const expireDate = new Date(key.expire_at);
    const now = new Date();
    const daysUntilExpire = (expireDate - now) / (1000 * 60 * 60 * 24);
    return daysUntilExpire < 30; // 30일 이내 만료
});

if (soonToExpire.length > 0) {
    console.log('만료 예정 API 키:', soonToExpire);
}
```

### 권한별 키 관리
```javascript
// 권한별 API 키 분류
const readOnlyKeys = apiKeys.filter(key => 
    key.permissions.includes('read') && !key.permissions.includes('trade')
);
const tradeKeys = apiKeys.filter(key => 
    key.permissions.includes('trade')
);
```

## 관련 기능
- API 키 생성 (업비트 웹사이트에서 수행)
- API 키 삭제 (업비트 웹사이트에서 수행)
- 2단계 인증 설정
- IP 주소 관리

---

<a name="exchange-bulk-cancel-orders"></a>
### 주문 일괄 취소 접수

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

---

<a name="exchange-cancel-and-reorder"></a>
### 취소 후 재주문

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

---

<a name="exchange-cancel-order"></a>
### 주문 취소 접수

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

---

## Quotation API (시세 API)

<a name="quotation-market-all"></a>
### 전체 마켓 조회

## 개요
업비트에서 거래 가능한 종목 목록을 조회합니다.

## API 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/market/all`

## 요청 파라미터

| 파라미터 | 타입 | 필수 여부 | 설명 |
|---------|------|----------|------|
| `is_details` | Boolean | 선택 | 유의종목 필드과 같은 상세 정보 노출 여부 (기본값: false) |

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `market` | String | 업비트에서 제공하는 시장 정보 (예: KRW-BTC) |
| `korean_name` | String | 거래 대상 디지털 자산의 한글명 |
| `english_name` | String | 거래 대상 디지털 자산의 영문명 |
| `market_event.warning` | Boolean | 업비트 시장경보 > 유의종목 지정 여부 |
| `market_event.caution` | Object | 업비트 시장경보 > 주의종목 지정 여부 |

## 응답 예시

```json
[
  {
    "market": "KRW-BTC",
    "korean_name": "비트코인",
    "english_name": "Bitcoin",
    "market_event": {
      "warning": false,
      "caution": {
        "PRICE_FLUCTUATIONS": false,
        "TRADING_VOLUME_SOARING": false,
        "DEPOSIT_AMOUNT_SOARING": false,
        "PRICE_DIFFERENCES": false,
        "CONCENTRATION_OF_SMALL_ACCOUNTS": false
      }
    }
  }
]
```

## 주의사항
- 주의종목 유형에는 가격 변동성, 거래량 급등, 입금량 급등, 가격 차이, 소액 계정 집중 등이 포함됩니다.
- 시장 경보에 대한 자세한 정보는 관련 공지사항을 참고하시기 바랍니다.
- 시장 코드는 거래소 전체에서 고유한 식별자로 사용됩니다.

---

<a name="quotation-candles-minutes"></a>
### 분봉 캔들 조회

## 개요
분봉 단위의 캔들 데이터를 조회합니다.

## API 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/candles/minutes/{unit}`

## 요청 파라미터

| 파라미터 | 타입 | 필수 여부 | 설명 |
|---------|------|----------|------|
| `unit` | Integer | 필수 | 분 단위 (URL 경로에 포함) |
| `market` | String | 필수 | 마켓 코드 (예: KRW-BTC) |
| `to` | String | 선택 | 마지막 캔들 시각 (exclusive), ISO8061 형식 지원, 기본값은 가장 최근 캔들 |
| `count` | Integer | 선택 | 캔들 개수 (최대 200개까지 요청 가능) |

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `market` | String | 종목 코드 |
| `candle_date_time_utc` | String | 캔들 기준 시각 (UTC) |
| `candle_date_time_kst` | String | 캔들 기준 시각 (KST) |
| `opening_price` | Double | 시가 |
| `high_price` | Double | 고가 |
| `low_price` | Double | 저가 |
| `trade_price` | Double | 종가 |
| `timestamp` | Long | 마지막 틱 저장 시각 |
| `candle_acc_trade_price` | Double | 누적 거래 금액 |
| `candle_acc_trade_volume` | Double | 누적 거래량 |
| `unit` | Integer | 분 단위 |

## 응답 예시

```json
[
  {
    "market": "KRW-BTC",
    "candle_date_time_utc": "2023-01-01T00:00:00",
    "candle_date_time_kst": "2023-01-01T09:00:00",
    "opening_price": 21500000.0,
    "high_price": 21600000.0,
    "low_price": 21400000.0,
    "trade_price": 21550000.0,
    "timestamp": 1672531200000,
    "candle_acc_trade_price": 1500000000.0,
    "candle_acc_trade_volume": 70.5,
    "unit": 1
  }
]
```

## 지원하는 분 단위
- 1분: `/v1/candles/minutes/1`
- 3분: `/v1/candles/minutes/3`
- 5분: `/v1/candles/minutes/5`
- 15분: `/v1/candles/minutes/15`
- 30분: `/v1/candles/minutes/30`
- 60분: `/v1/candles/minutes/60`
- 240분: `/v1/candles/minutes/240`

## 주의사항
- 캔들은 해당 시간대에 체결이 발생한 경우에만 생성됩니다.
- 최대 200개의 캔들을 한 번에 요청할 수 있습니다.
- `to` 파라미터를 사용하여 특정 시점 이전의 데이터를 조회할 수 있습니다.
- 분 단위는 URL 경로에 직접 포함되어야 합니다.

---

<a name="quotation-ticker-current-price"></a>
### 현재가 정보

## 개요
요청 당시 종목의 스냅샷을 반환합니다.

## HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/ticker`

## 요청 파라미터

| 파라미터명 | 타입 | 필수 여부 | 설명 |
|-----------|------|-----------|------|
| `markets` | String | 필수 | 쉼표로 구분된 마켓 코드 (예: KRW-BTC, BTC-ETH) |

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `market` | String | 마켓 코드 |
| `trade_price` | Double | 종가 (현재가) |
| `opening_price` | Double | 시가 |
| `high_price` | Double | 고가 |
| `low_price` | Double | 저가 |
| `change` | String | 가격 변화 상태 (EVEN/RISE/FALL) |
| `change_price` | Double | 가격 변화량 |
| `change_rate` | Double | 가격 변화율 |
| `signed_change_price` | Double | 부호가 있는 가격 변화량 |
| `signed_change_rate` | Double | 부호가 있는 가격 변화율 |
| `trade_volume` | Double | 가장 최근 거래량 |
| `acc_trade_price` | Double | 누적 거래대금 |
| `acc_trade_price_24h` | Double | 24시간 누적 거래대금 |
| `acc_trade_volume` | Double | 누적 거래량 |
| `acc_trade_volume_24h` | Double | 24시간 누적 거래량 |
| `highest_52_week_price` | Double | 52주 신고가 |
| `highest_52_week_date` | String | 52주 신고가 달성일 |
| `lowest_52_week_price` | Double | 52주 신저가 |
| `lowest_52_week_date` | String | 52주 신저가 달성일 |
| `timestamp` | Long | 타임스탬프 |

## 응답 예시
```json
[
  {
    "market": "KRW-BTC",
    "trade_price": 50000000.0,
    "opening_price": 49000000.0,
    "high_price": 51000000.0,
    "low_price": 48000000.0,
    "change": "RISE",
    "change_price": 1000000.0,
    "change_rate": 0.0204,
    "signed_change_price": 1000000.0,
    "signed_change_rate": 0.0204,
    "trade_volume": 0.1,
    "acc_trade_price": 1000000000.0,
    "acc_trade_price_24h": 2000000000.0,
    "acc_trade_volume": 20.0,
    "acc_trade_volume_24h": 40.0,
    "highest_52_week_price": 80000000.0,
    "highest_52_week_date": "2023-01-01",
    "lowest_52_week_price": 30000000.0,
    "lowest_52_week_date": "2022-06-01",
    "timestamp": 1672574400000
  }
]
```

## 주의사항
- 변화 관련 필드들은 전일 종가 대비 계산됩니다.
- 타임스탬프는 UTC 및 KST 형식으로 제공됩니다.
- 단일 종목부터 복수 종목까지 조회 가능합니다.

---

## WebSocket API (웹소켓 API)

<a name="websocket-general-info"></a>
### 일반 정보

## 개요
Upbit의 WebSocket API는 실시간 암호화폐 데이터를 제공하는 웹소켓 서비스입니다.

## API 엔드포인트
- **Public 타입**: `wss://api.upbit.com/websocket/v1`
- **Private 타입**: `wss://api.upbit.com/websocket/v1/private`

## 데이터 타입

### Public 타입 (인증 불필요)
- `ticker`: 현재가 (스냅샷 및 실시간)
- `trade`: 체결 (스냅샷 및 실시간)
- `orderbook`: 호가 (스냅샷 및 실시간)
- `candle.1s`: 초봉 (스냅샷 및 실시간)

### Private 타입 (인증 필요)
- `myOrder`: 내 주문 (실시간)
- `myAsset`: 내 자산 (실시간)

## 주요 특징
- **스냅샷**: 요청 당시의 상태 정보
- **실시간**: 지속적으로 데이터 스트림 제공
- 요청 방법에 따라 수신 가능한 데이터 다름

## 시간대 설정
- **중요**: Upbit API의 시간대는 KST(한국 표준 시간) 기준으로 맞춰야 함
- 모든 시간 데이터는 KST 기준으로 처리

## 참고 사항
- 자세한 인증 방법은 WebSocket 요청 형식 페이지 참조
- Rate Limit 적용 (별도 페이지 참고)
- 연결 유지를 위해 주기적 ping/pong 권장

---

<a name="websocket-ticker"></a>
### 현재가 (Ticker)

## 개요
WebSocket을 통해 실시간 암호화폐 현재가 정보를 수신할 수 있는 API입니다.

## WebSocket 엔드포인트
```
wss://api.upbit.com/websocket/v1
```

## 요청 형식

### 기본 요청 구조
```json
[
  {
    "ticket": "test example"
  },
  {
    "type": "ticker",
    "codes": ["KRW-BTC", "KRW-ETH"],
    "is_only_snapshot": false,
    "is_only_realtime": false
  },
  {
    "format": "DEFAULT"
  }
]
```

### 주요 요청 파라미터
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `ticket` | String | 필수 | 요청 식별자 |
| `type` | String | 필수 | `"ticker"` 고정 |
| `codes` | Array | 필수 | 마켓 코드 리스트 (대문자) |
| `is_only_snapshot` | Boolean | 선택 | 스냅샷 데이터만 수신 |
| `is_only_realtime` | Boolean | 선택 | 실시간 데이터만 수신 |
| `format` | String | 선택 | 응답 형식 (DEFAULT/SIMPLE) |

## 응답 필드

### 주요 응답 필드
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `type` | 데이터 타입 | String | "ticker" |
| `code` | 마켓 코드 | String | 예: "KRW-BTC" |
| `trade_price` | 현재가 | Double | 최근 체결 가격 |
| `change` | 전일 대비 상태 | String | RISE/FALL/EVEN |
| `change_rate` | 전일 대비 등락율 | Double | 소수점 형태 |
| `change_price` | 전일 대비 등락액 | Double | |
| `signed_change_rate` | 부호가 있는 등락율 | Double | |
| `signed_change_price` | 부호가 있는 등락액 | Double | |
| `trade_volume` | 가장 최근 거래량 | Double | |
| `acc_trade_volume` | 누적 거래량(UTC 0시 기준) | Double | |
| `acc_trade_volume_24h` | 24시간 누적 거래량 | Double | |
| `acc_trade_price` | 누적 거래대금(UTC 0시 기준) | Double | |
| `acc_trade_price_24h` | 24시간 누적 거래대금 | Double | |
| `trade_date` | 최근 거래 일자(UTC) | String | yyyyMMdd |
| `trade_time` | 최근 거래 시각(UTC) | String | HHmmss |
| `trade_timestamp` | 체결 타임스탬프 | Long | 밀리초 |
| `ask_bid` | 매수/매도 구분 | String | ASK/BID |
| `acc_ask_volume` | 누적 매도량 | Double | |
| `acc_bid_volume` | 누적 매수량 | Double | |
| `highest_52_week_price` | 52주 신고가 | Double | |
| `highest_52_week_date` | 52주 신고가 달성일 | String | |
| `lowest_52_week_price` | 52주 신저가 | Double | |
| `lowest_52_week_date` | 52주 신저가 달성일 | String | |
| `trade_status` | 거래 상태 | String | |
| `market_state` | 마켓 경고 구분 | String | |
| `market_state_for_ios` | iOS 앱용 마켓 상태 | String | |
| `is_trading_suspended` | 거래 정지 여부 | Boolean | |
| `delisting_date` | 상장폐지일 | String | |
| `market_warning` | 마켓 경고 구분 | String | |
| `timestamp` | 타임스탬프 | Long | 밀리초 |
| `stream_type` | 스트림 타입 | String | SNAPSHOT/REALTIME |

## 응답 예시

### DEFAULT 포맷
```json
{
  "type": "ticker",
  "code": "KRW-BTC",
  "opening_price": 32285000,
  "high_price": 32400000,
  "low_price": 32100000,
  "trade_price": 32287000,
  "prev_closing_price": 32280000,
  "change": "RISE",
  "change_rate": 0.0002168050,
  "signed_change_rate": 0.0002168050,
  "change_price": 7000,
  "signed_change_price": 7000,
  "trade_volume": 0.03073852,
  "acc_trade_volume": 2624.53894846,
  "acc_trade_volume_24h": 3711.41927673,
  "acc_trade_price": 84795524793.68954,
  "acc_trade_price_24h": 119897717952.65657,
  "trade_date": "20240101",
  "trade_time": "120000",
  "trade_timestamp": 1704096000000,
  "ask_bid": "BID",
  "acc_ask_volume": 1274.29425598,
  "acc_bid_volume": 1350.24469248,
  "highest_52_week_price": 50000000,
  "highest_52_week_date": "2023-12-15",
  "lowest_52_week_price": 20000000,
  "lowest_52_week_date": "2023-01-01",
  "market_state": "ACTIVE",
  "is_trading_suspended": false,
  "delisting_date": null,
  "market_warning": "NONE",
  "timestamp": 1704096000123,
  "stream_type": "REALTIME"
}
```

## 주의사항

- **마켓 코드**: 반드시 대문자로 입력 (예: "KRW-BTC")
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리해야 함
- **데이터 타입**: 숫자 필드는 모두 float64/Double 타입
- **스트림 타입**: SNAPSHOT(초기 데이터) / REALTIME(실시간 업데이트) 구분
- **연결 유지**: 장시간 연결 시 ping/pong 메커니즘 구현 권장
- **에러 처리**: 네트워크 오류나 연결 끊김에 대한 적절한 예외 처리 필요

---

## 문서 정보

### 통합 완료 API 목록

#### Exchange API (29개)
1. 전체 계좌 조회
2. API 키 리스트 조회  
3. 주문 일괄 취소 접수
4. 취소 후 재주문
5. 주문 취소 접수
6. ID로 주문리스트 취소 접수
7. 입금 주소 생성 요청
8. 개별 입금 주소 조회
9. 개별 입금 조회
10. 원화 입금하기
11. 입금 리스트 조회
12. 입출금 현황 조회
13. 디지털 자산 입금 정보 조회
14. 주문 가능 정보 조회
15. 개별 주문 조회
16. ID로 주문 조회
17. 전체 미체결 주문 조회
18. 전체 체결 주문 조회
19. 주문하기
20. Travel Rule 인증
21. Travel Rule 거래소 리스트
22. Travel Rule TXID 검증
23. 출금 가능 주소 리스트 조회
24. 출금 취소
25. 출금 가능 정보
26. 디지털 자산 출금하기
27. 개별 출금 조회
28. 원화 출금하기
29. 출금 리스트 조회

#### Quotation API (12개)
1. 전체 마켓 조회
2. 초봉 캔들 조회  
3. 분봉 캔들 조회
4. 일봉 캔들 조회
5. 주봉 캔들 조회
6. 월봉 캔들 조회
7. 연봉 캔들 조회
8. 최근 체결 내역
9. 현재가 정보
10. 마켓별 현재가 정보
11. 호가 정보 조회
12. 지원 레벨 조회

#### WebSocket API (13개)
1. 일반 정보
2. 인증
3. 테스트 샘플
4. 요청 형식
5. 현재가 (Ticker)
6. 체결
7. 호가
8. 캔들
9. 내 주문
10. 내 자산
11. 구독
12. 에러 코드
13. 연결

### 참고 사항
- 이 통합 문서는 2024년 6월 기준으로 작성되었습니다.
- 최신 API 변경사항은 업비트 공식 문서를 참조하시기 바랍니다.
- 문서에 포함되지 않은 세부 API는 개별 파일을 참조하세요.
- 모든 예제 코드와 응답은 참고용이며, 실제 구현 시 최신 스펙을 확인하시기 바랍니다.

### 작성 정보
- 작성일: 2024년 6월 21일
- 문서 유형: API 통합 가이드
- 대상: 개발자, API 사용자
- 언어: 한국어 (코드 예제: 영어)

---

**이 문서는 Upbit API의 모든 주요 기능을 포괄하는 종합 가이드입니다. 실제 개발 시에는 각 API의 최신 스펙을 확인하시기 바랍니다.**
