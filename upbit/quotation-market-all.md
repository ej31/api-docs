# 종목 코드 조회 (Market Code Lookup)

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