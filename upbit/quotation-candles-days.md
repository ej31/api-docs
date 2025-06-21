# 일(Day) 캔들 (Day Candles)

## 개요
일봉 단위의 캔들 데이터를 조회합니다.

## API 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/candles/days`

## 요청 파라미터

| 파라미터 | 타입 | 필수 여부 | 설명 |
|---------|------|----------|------|
| `market` | String | 필수 | 마켓 코드 (예: KRW-BTC) |
| `to` | String | 선택 | 마지막 캔들 시각, ISO8061 형식 |
| `count` | Integer | 선택 | 캔들 개수 (최대 200개) |
| `converting_price_unit` | String | 선택 | 가격 변환 통화 단위 |

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `market` | String | 마켓 코드 |
| `candle_date_time_utc` | String | 캔들 기준 시각 (UTC) |
| `candle_date_time_kst` | String | 캔들 기준 시각 (KST) |
| `opening_price` | Double | 시가 |
| `high_price` | Double | 고가 |
| `low_price` | Double | 저가 |
| `trade_price` | Double | 종가 |
| `timestamp` | Long | 마지막 틱 저장 시각 |
| `candle_acc_trade_price` | Double | 누적 거래 금액 |
| `candle_acc_trade_volume` | Double | 누적 거래량 |
| `prev_closing_price` | Double | 전일 종가 |
| `change_price` | Double | 전일 대비 변화 금액 |
| `change_rate` | Double | 전일 대비 변화율 |
| `converted_trade_price` | Double | 변환된 가격 (converting_price_unit 사용 시) |

## 응답 예시

```json
[
  {
    "market": "KRW-BTC",
    "candle_date_time_utc": "2023-01-01T00:00:00",
    "candle_date_time_kst": "2023-01-01T09:00:00",
    "opening_price": 21500000.0,
    "high_price": 21800000.0,
    "low_price": 21200000.0,
    "trade_price": 21650000.0,
    "timestamp": 1672531200000,
    "candle_acc_trade_price": 15000000000.0,
    "candle_acc_trade_volume": 705.5,
    "prev_closing_price": 21400000.0,
    "change_price": 250000.0,
    "change_rate": 0.0117,
    "converted_trade_price": null
  }
]
```

## 가격 변환 단위
`converting_price_unit` 파라미터를 사용하여 다른 통화 단위로 가격을 변환할 수 있습니다:
- `KRW`: 원화
- `BTC`: 비트코인
- `USDT`: 테더

## 주의사항
- 캔들은 해당 시간대에 체결이 발생한 경우에만 생성됩니다.
- 최대 200개의 캔들을 한 번에 요청할 수 있습니다.
- 일봉 데이터에는 전일 대비 변화량과 변화율 정보가 포함됩니다.
- `converting_price_unit`을 사용하면 `converted_trade_price` 필드에 변환된 가격이 제공됩니다.