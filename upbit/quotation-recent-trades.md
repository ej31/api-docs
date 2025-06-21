# 최근 체결 내역 API

## 개요
요청 당시 종목의 최근 체결 내역을 반환합니다.

## HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/trades/ticks`

## 요청 파라미터

| 파라미터명 | 타입 | 필수 여부 | 설명 |
|-----------|------|-----------|------|
| `market` | String | 필수 | 마켓 코드 (예: KRW-BTC) |
| `to` | String | 선택 | 마지막 체결 시각 |
| `count` | Integer | 선택 | 체결 개수 |
| `cursor` | String | 선택 | 페이지네이션 커서 |
| `days_ago` | Integer | 선택 | 최근 7일 이내의 이전 체결 데이터 |

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `market` | String | 마켓 코드 |
| `trade_date_utc` | String | 체결 일자 (UTC) |
| `trade_time_utc` | String | 체결 시각 (UTC) |
| `timestamp` | Long | 체결 타임스탬프 |
| `trade_price` | Double | 체결 가격 |
| `trade_volume` | Double | 체결 량 |
| `prev_closing_price` | Double | 전일 종가 |
| `change_price` | Double | 변화 가격 |
| `ask_bid` | String | 매수/매도 구분 |
| `sequential_id` | Long | 체결 번호 (유니크) |

## 응답 예시
```json
[
  {
    "market": "KRW-BTC",
    "trade_date_utc": "2023-01-01",
    "trade_time_utc": "12:00:00",
    "timestamp": 1672574400000,
    "trade_price": 50000000.0,
    "trade_volume": 0.1,
    "prev_closing_price": 49000000.0,
    "change_price": 1000000.0,
    "ask_bid": "BID",
    "sequential_id": 123456789
  }
]
```

## 주의사항
- `sequential_id`는 체결의 유일성을 식별할 수 있지만 체결의 순서를 보장하지는 않습니다.
- 체결 시각은 UTC 및 KST 형식으로 제공됩니다.