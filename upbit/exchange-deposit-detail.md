# 개별 입금 조회 (Individual Deposit Inquiry)

## 개요
특정 입금 건에 대한 상세 정보를 조회하는 API입니다. UUID, TXID, 또는 화폐 코드를 통해 개별 입금 내역을 확인할 수 있습니다.

## API 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/deposit`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|-----|-----|
| uuid | String | 선택 | 입금 UUID |
| txid | String | 선택 | 입금 트랜잭션 ID |
| currency | String | 선택 | 화폐 코드 |

*※ 위 파라미터 중 하나는 반드시 제공되어야 합니다.*

## 응답 필드

| 필드명 | 타입 | 설명 |
|-------|------|-----|
| type | String | 입출금 타입 |
| uuid | String | 고유 입금 ID |
| currency | String | 화폐 코드 |
| net_type | String | 입금 네트워크 |
| txid | String | 트랜잭션 ID |
| state | String | 입금 상태 |
| created_at | DateString | 입금 생성 시간 |
| done_at | DateString | 입금 완료 시간 |
| amount | NumberString | 입금 수량 |
| fee | NumberString | 입금 수수료 |
| transaction_type | String | 입금 타입 |

### 입금 상태 (state) 옵션
- `PROCESSING`: 입금 진행 중
- `ACCEPTED`: 완료
- `CANCELLED`: 취소됨
- `REJECTED`: 거절됨
- `TRAVEL_RULE_SUSPECTED`: 추가 검증 대기 중
- `REFUNDING`: 환불 진행 중
- `REFUNDED`: 환불됨

### 거래 타입 (transaction_type) 옵션
- `default`: 일반 입금
- `internal`: 즉시 입금

## 주의사항
- UUID, TXID, 또는 화폐 코드 중 하나의 파라미터를 반드시 제공해야 합니다
- 특정 입금 건의 상세 정보를 조회할 때 사용합니다
- 입금 상태 변화를 추적하기 위해 활용할 수 있습니다