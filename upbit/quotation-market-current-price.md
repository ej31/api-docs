# 마켓 단위 현재가 정보 API

## 개요
마켓 단위별 종목들의 스냅샷을 반환합니다.

## HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/ticker/all`

## 요청 파라미터

| 파라미터명 | 타입 | 필수 여부 | 설명 |
|-----------|------|-----------|------|
| `quote_currencies` | String | 선택 | 쉼표로 구분된 거래 통화 코드 (예: KRW, BTC, USDT) |

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
  },
  {
    "market": "BTC-ETH",
    "trade_price": 0.065,
    "opening_price": 0.064,
    "high_price": 0.066,
    "low_price": 0.063,
    "change": "RISE",
    "change_price": 0.001,
    "change_rate": 0.0156,
    "signed_change_price": 0.001,
    "signed_change_rate": 0.0156,
    "trade_volume": 10.0,
    "acc_trade_price": 100.0,
    "acc_trade_price_24h": 200.0,
    "acc_trade_volume": 1500.0,
    "acc_trade_volume_24h": 3000.0,
    "highest_52_week_price": 0.08,
    "highest_52_week_date": "2023-01-01",
    "lowest_52_week_price": 0.05,
    "lowest_52_week_date": "2022-06-01",
    "timestamp": 1672574400000
  }
]
```

## 주의사항
- 변화 관련 필드들은 전일 종가 대비 계산됩니다.
- 타임스탬프는 UTC 및 KST 형식으로 제공됩니다.
- 거래 통화 코드를 지정하지 않으면 전체 마켓의 현재가 정보를 반환합니다.
- 52주 고가/저가 정보도 포함되어 있습니다.