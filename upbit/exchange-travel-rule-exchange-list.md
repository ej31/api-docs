# 트래블룰 가능 거래소 리스트 조회

## API 개요
입출금 신청 가능한 거래소 목록을 조회하는 API입니다. 계정주 확인(트래블룰 검증)이 가능한 거래소들의 리스트를 제공합니다.

## 요청 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/travel_rule/vasps`
- **인증**: JWT 토큰 필요

## 요청 파라미터
이 API는 별도의 요청 파라미터가 필요하지 않습니다.

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `vasp_name` | String | 거래소명 |
| `vasp_uuid` | String | 거래소 고유 식별자 |
| `depositable` | Boolean | 입금 가능 여부 |
| `withdrawable` | Boolean | 출금 가능 여부 |

## 응답 예시

```json
[
  {
    "vasp_name": "Example Exchange",
    "vasp_uuid": "12345678-1234-1234-1234-123456789abc",
    "depositable": true,
    "withdrawable": true
  }
]
```

## 주의사항

- 이 API는 계정주 확인 서비스에 연동된 VASP(Virtual Asset Service Provider) 정보를 반환합니다.
- 계정주 확인 서비스에 대한 자세한 내용은 업비트 공식 문서를 참조하시기 바랍니다.
- 실제 입출금 가능 여부는 거래소별로 상이할 수 있으므로, 해당 거래소의 현재 상태를 별도로 확인하는 것을 권장합니다.

## 관련 API
- [트래블룰 TxID 검증](./exchange-travel-rule-txid-verification.md)
- [입출금 현황 조회](./exchange-deposit-withdrawal-status.md)