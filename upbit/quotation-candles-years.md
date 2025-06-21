# 연(Year) 캔들 (Year Candles)

## 개요
연봉 단위의 캔들 데이터를 조회합니다.

## API 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/candles/years`

## 요청 파라미터

| 파라미터 | 타입 | 필수 여부 | 설명 |
|---------|------|----------|------|
| `market` | String | 필수 | 마켓 코드 (예: KRW-BTC) |
| `to` | String | 선택 | 마지막 캔들 시각 (exclusive), ISO8061 형식, 기본값은 가장 최근 캔들 |
| `count` | Integer | 선택 | 캔들 개수 (최대 200개) |

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
| `first_day_of_period` | String | 캔들 기간의 가장 첫 날 |

## 응답 예시

```json
[
  {
    "market": "KRW-BTC",
    "candle_date_time_utc": "2023-01-01T00:00:00",
    "candle_date_time_kst": "2023-01-01T09:00:00",
    "opening_price": 19500000.0,
    "high_price": 69000000.0,
    "low_price": 15800000.0,
    "trade_price": 43200000.0,
    "timestamp": 1703721600000,
    "candle_acc_trade_price": 378000000000000.0,
    "candle_acc_trade_volume": 12467850.5,
    "first_day_of_period": "2023-01-01"
  }
]
```

## 연봉 기간 정보
- 연봉은 매년 1월 1일부터 12월 31일까지의 기간을 기준으로 합니다.
- `first_day_of_period` 필드를 통해 해당 연봉이 시작된 날짜를 확인할 수 있습니다.
- 캔들의 시각은 해당 연도의 마지막 거래 시각을 나타냅니다.

## 지원하는 프로그래밍 언어 예시
API 문서에서는 다음 언어들의 예시 코드를 제공합니다:
- Shell (curl)
- Python
- Node.js

## 주의사항
- 캔들은 해당 시간대에 체결이 발생한 경우에만 생성됩니다.
- 최대 200개의 캔들을 한 번에 요청할 수 있습니다.
- 연봉 데이터는 1년 단위로 집계된 거래 정보를 제공합니다.
- `to` 파라미터를 사용하여 특정 시점 이전의 데이터를 조회할 수 있습니다.
- 연봉 데이터는 장기적인 시장 트렌드 분석에 유용합니다.
- 숫자 필드들은 Double 또는 Long 타입으로 반환됩니다.