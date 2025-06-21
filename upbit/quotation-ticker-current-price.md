# 종목 단위 현재가 정보 API

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