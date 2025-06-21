# 호가 정보 조회 API

## 개요
호가 정보를 조회하여 매수/매도 호가를 반환합니다.

## HTTP 메서드 및 엔드포인트
- **메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/orderbook`

## 요청 파라미터

| 파라미터명 | 타입 | 필수 여부 | 설명 |
|-----------|------|-----------|------|
| `markets` | String | 필수 | 쉼표로 구분된 마켓 코드 (예: "KRW-BTC, BTC-ETH") |
| `level` | Double | 선택 | 호가 집계 단위 (원화 마켓만 지원) |

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `market` | String | 마켓 코드 |
| `timestamp` | Long | 호가 생성 시각 |
| `total_ask_size` | Double | 총 매도 호가 잔량 |
| `total_bid_size` | Double | 총 매수 호가 잔량 |
| `orderbook_units` | Array | 호가 정보 목록 (최대 30개 가격 레벨) |

### orderbook_units 배열 내부 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `ask_price` | Double | 매도 호가 |
| `bid_price` | Double | 매수 호가 |
| `ask_size` | Double | 매도 잔량 |
| `bid_size` | Double | 매수 잔량 |
| `level` | Double | 집계 단위 |

## 응답 예시
```json
[
  {
    "market": "KRW-BTC",
    "timestamp": 1672574400000,
    "total_ask_size": 5.5,
    "total_bid_size": 8.2,
    "orderbook_units": [
      {
        "ask_price": 50100000.0,
        "bid_price": 50000000.0,
        "ask_size": 0.5,
        "bid_size": 1.2,
        "level": 0
      },
      {
        "ask_price": 50200000.0,
        "bid_price": 49900000.0,
        "ask_size": 0.8,
        "bid_size": 0.9,
        "level": 0
      }
    ]
  }
]
```

## 주의사항
- `orderbook_units`에서 최대 30개의 가격 레벨을 지원합니다.
- 집계 레벨 기능은 원화(KRW) 마켓에서만 지원됩니다.
- 지원하지 않는 레벨을 요청할 경우 빈 목록이 반환됩니다.
- 호가 정보는 실시간으로 변동되므로 최신 데이터 확인이 필요합니다.