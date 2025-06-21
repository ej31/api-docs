# 분(Minute) 캔들 (Minute Candles)

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