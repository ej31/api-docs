# 개별 입금 주소 조회 (Individual Deposit Address Lookup)

## 개요
생성된 입금 주소를 조회하는 API입니다. 특정 화폐와 네트워크에 대한 입금 주소 정보를 확인할 수 있습니다.

## API 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/deposits/coin_address`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|-----|-----|
| currency | String | 필수 | 화폐 코드 |
| net_type | String | 필수 | 입금 네트워크 |

## 응답 필드

| 필드명 | 타입 | 설명 |
|-------|------|-----|
| currency | String | 화폐 코드 (대문자) |
| net_type | String | 입금 네트워크 |
| deposit_address | String | 입금 주소 |
| secondary_address | String | 보조 입금 주소 |

## 주의사항

### 주소 생성 상태
- **중요**: 입금 주소 생성 요청 이후 아직 발급되지 않은 상태일 경우 `deposit_address`가 `null`일 수 있습니다
- 주소가 `null`인 경우, 주소 생성이 완료될 때까지 기다린 후 재시도하세요

### 비동기 처리
- 주소 생성은 비동기로 처리되므로 즉시 결과를 얻을 수 없을 수 있습니다
- 주소 생성 요청 후 적절한 대기 시간을 두고 조회하세요

### 네트워크 확인
- 올바른 네트워크 타입을 지정해야 합니다
- 잘못된 네트워크 타입 지정 시 오류가 발생할 수 있습니다

## 언어별 SDK 지원
- NodeJS
- Python
- Ruby
- Java

## 사용 예시 플로우
1. 입금 주소 생성 요청 (POST `/v1/deposits/generate_coin_address`)
2. 적절한 대기 시간 후 본 API로 주소 조회
3. `deposit_address`가 `null`인 경우 재시도
4. 주소가 생성되면 해당 주소로 입금 진행

## 응답 예시
```json
{
  "currency": "BTC",
  "net_type": "BTC",
  "deposit_address": "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa",
  "secondary_address": null
}
```