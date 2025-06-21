# 원화 출금하기 (Withdraw Korean Won)

## 개요
원화 출금을 요청하는 API입니다. 등록된 출금 계좌로 출금됩니다.

## API 정보
- **HTTP Method**: POST
- **Endpoint**: `https://api.upbit.com/v1/withdraws/krw`
- **인증**: API Key 필요

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `amount` | Number | 필수 | 출금 금액 |
| `two_factor_type` | String | 필수 | 2단계 인증 방식 |

### 2단계 인증 방식 (two_factor_type)
- `kakao`: 카카오톡 인증
- `naver`: 네이버 인증  
- `hana`: 하나은행 인증

## 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `type` | String | 거래 유형 |
| `uuid` | String | 출금 고유 ID |
| `currency` | String | 화폐 코드 (KRW) |
| `txid` | String | 트랜잭션 ID |
| `state` | String | 출금 상태 |
| `created_at` | DateString | 출금 생성 시간 |
| `done_at` | DateString | 출금 완료 시간 |
| `amount` | NumberString | 출금 금액 |
| `fee` | NumberString | 출금 수수료 |
| `transaction_type` | String | 출금 유형 |
| `is_cancelable` | Boolean | 출금 취소 가능 여부 |

## 응답 예시

```json
{
  "type": "withdraw",
  "uuid": "35a4f1dc-1db5-4d6b-89b5-7ec137875956",
  "currency": "KRW",
  "txid": "krw-withdraw-tx-001",
  "state": "PROCESSING",
  "created_at": "2023-12-18T15:17:51+09:00",
  "done_at": null,
  "amount": "100000",
  "fee": "1000",
  "transaction_type": "default",
  "is_cancelable": false
}
```

## 중요 변경사항

### 🔄 2023년 12월 18일부터 적용
- **`two_factor_type` 필수 필드**: 12월 18일부터 `two_factor_type`은 필수 필드입니다.
- **카카오페이 지원 중단**: `kakao_pay` 인증 방식은 더 이상 지원되지 않습니다.

## 주의사항

### ⚠️ 출금 제한
- **원화 출금 취소 불가**: 원화 출금은 취소를 지원하지 않습니다.
- **등록 계좌만 사용**: 미리 등록된 출금 계좌로만 출금됩니다.
- **2단계 인증 필수**: 반드시 2단계 인증이 설정되어 있어야 합니다.

### 📋 사전 요구사항
1. 업비트 계정에 출금 계좌 등록
2. 2단계 인증 설정 (카카오톡, 네이버, 하나은행 중 선택)
3. 출금 가능 금액 확인

### 💡 처리 시간
- 영업일 기준으로 처리됩니다.
- 은행 운영 시간에 따라 실제 입금 시간이 달라질 수 있습니다.
- 출금 수수료가 별도로 차감됩니다.