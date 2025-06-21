# 입금 TxID로 트래블룰 검증하기

## API 개요
입금 거래 ID(TxID)를 이용하여 트래블룰 검증을 수행하는 API입니다. 상대방 거래소의 입금 거래에 대한 계정주 확인을 처리합니다.

## 요청 정보
- **HTTP 메서드**: POST
- **엔드포인트**: `https://api.upbit.com/v1/travel_rule/deposit/txid`
- **인증**: JWT 토큰 필요

## 요청 파라미터

| 파라미터명 | 타입 | 필수 여부 | 설명 |
|------------|------|-----------|------|
| `vasp_uuid` | String | 필수 | 상대방 거래소 UUID |
| `txid` | String | 선택 | 입금 거래 ID |
| `currency` | String | 선택 | 입금 통화 |
| `net_type` | String | 선택 | 입금 네트워크 타입 |

## 응답 필드

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `deposit_uuid` | String | 입금 UUID |
| `deposit_state` | String | 입금 상태 |
| `verification_result` | String | 검증 결과 |

## 응답 예시

```json
{
  "deposit_uuid": "12345678-1234-1234-1234-123456789abc",
  "deposit_state": "ACCEPTED",
  "verification_result": "SUCCESS"
}
```

## 요청 예시

```json
{
  "vasp_uuid": "12345678-1234-1234-1234-123456789abc",
  "txid": "0x1234567890abcdef1234567890abcdef12345678",
  "currency": "BTC",
  "net_type": "BTC"
}
```

## 주의사항

- **검증 빈도 제한**: 동일한 입금에 대해 10분당 1회만 검증 가능합니다.
- **1회 요청 처리**: 한 번의 요청으로 계정주 확인이 완료됩니다.
- **검증 성공 시**: 검증이 성공하면 입금이 정상적으로 처리됩니다.
- **진행 중인 입금**: 입금이 아직 처리 중인 경우, 처리 완료 후 검증이 완료됩니다.
- **고유 조합 필요**: 상대방 UUID, TxID, 통화, 네트워크 타입의 조합이 일치해야 검증이 가능합니다.

## 검증 절차

1. 상대방 거래소에서 입금 거래 정보 확인
2. 필요한 파라미터(vasp_uuid, txid, currency, net_type) 수집
3. API 요청을 통한 검증 수행
4. 검증 결과에 따른 입금 처리 진행

## 관련 API
- [트래블룰 가능 거래소 리스트](./exchange-travel-rule-exchange-list.md)
- [입금 상세 정보 조회](./exchange-deposit-detail.md)