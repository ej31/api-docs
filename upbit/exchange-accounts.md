# Upbit Exchange API - 전체 계좌 조회

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