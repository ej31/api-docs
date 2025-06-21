# 원화 입금하기 (Deposit Korean Won)

## 개요
한국 원화(KRW) 입금을 요청하는 API입니다. 2단계 인증을 통해 안전하게 원화 입금을 진행할 수 있습니다.

## API 정보
- **HTTP 메서드**: POST
- **엔드포인트**: `https://api.upbit.com/v1/deposits/krw`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|-----|-----|
| amount | Number | 필수 | 입금 금액 |
| two_factor_type | String | 필수 | 2단계 인증 방식 |

### 2단계 인증 방식 (two_factor_type) 옵션
- `kakao`: 카카오 인증
- `naver`: 네이버 인증  
- `hana`: 하나은행 공인인증서 인증

## 응답 필드

| 필드명 | 타입 | 설명 |
|-------|------|-----|
| type | String | 입출금 타입 |
| uuid | String | 고유 입금 ID |
| currency | String | 화폐 코드 |
| txid | String | 트랜잭션 ID |
| state | String | 입금 상태 |
| created_at | DateString | 입금 생성 시간 |
| done_at | DateString | 입금 완료 시간 |
| amount | NumberString | 입금 금액 |
| fee | NumberString | 입금 수수료 |
| transaction_type | String | 입금 타입 |

## 주의사항

### 필수 인증
- **2024년 12월 18일부터 `two_factor_type`은 필수 필드입니다**
- 반드시 지원되는 인증 방식 중 하나를 선택해야 합니다

### 지원 중단된 인증 방식
- `kakao_pay` 인증 방식은 더 이상 사용할 수 없습니다

### 보안
- 2단계 인증을 통해 안전한 입금 처리가 이루어집니다
- 인증 과정을 거쳐야만 입금이 완료됩니다

### 처리 시간
- 인증 방식에 따라 처리 시간이 달라질 수 있습니다
- 은행 업무 시간 및 시스템 점검 시간을 고려해야 합니다

## 응답 예시
```json
{
  "type": "deposit",
  "uuid": "35a4f1dc-1db5-4d6b-89b5-7ec137875956",
  "currency": "KRW",
  "txid": "txid12345",
  "state": "PROCESSING",
  "created_at": "2024-01-15T09:30:00+09:00",
  "done_at": null,
  "amount": "100000.0",
  "fee": "0.0",
  "transaction_type": "default"
}
```

## 사용 플로우
1. 입금할 금액과 인증 방식 선택
2. API 호출로 입금 요청
3. 선택한 인증 방식으로 2단계 인증 진행
4. 인증 완료 후 입금 처리