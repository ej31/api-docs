# 출금 허용 주소 리스트 조회 (Withdrawal Allowed Address List Inquiry)

## 개요
등록된 출금 허용 주소 목록을 조회하는 API입니다. 디지털 자산 출금 시 사용할 수 있는 주소 목록을 확인할 수 있습니다.

## API 정보
- **HTTP Method**: GET
- **Endpoint**: `https://api.upbit.com/v1/withdraws/coin_addresses`
- **인증**: API Key 필요

## 요청 파라미터
이 API는 별도의 요청 파라미터가 없습니다.

## 응답 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `currency` | String | 출금 화폐 |
| `net_type` | String | 출금 네트워크 유형 |
| `network_name` | String | 출금 네트워크 이름 |
| `withdraw_address` | String | 출금 주소 |
| `secondary_address` | String | 2차 출금 주소 (특정 디지털 자산용) |

## 응답 예시

```json
[
  {
    "currency": "BTC",
    "net_type": "BTC",
    "network_name": "Bitcoin",
    "withdraw_address": "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa",
    "secondary_address": null
  },
  {
    "currency": "ETH",
    "net_type": "ETH",
    "network_name": "Ethereum",
    "withdraw_address": "0x742d35Cc6634C0532925a3b8D",
    "secondary_address": null
  },
  {
    "currency": "XRP",
    "net_type": "XRP",
    "network_name": "Ripple",
    "withdraw_address": "rLNaPoKeeBjZe2qs6x52yVPZpZ8td4dc6w",
    "secondary_address": "12345678"
  }
]
```

## 주소 등록 방법

### 📍 등록 위치
업비트 웹사이트 > [MY] > [Open API 관리] > [디지털자산 출금주소 관리]에서 주소를 등록할 수 있습니다.

### 📋 등록 절차
1. 업비트 웹사이트 로그인
2. 마이페이지 > Open API 관리 메뮴 접근
3. 디지털자산 출금주소 관리 선택
4. 출금하려는 자산의 네트워크 선택
5. 출금 주소 입력 및 검증
6. 보안 인증 완료

## 네트워크별 주의사항

### ⚠️ 네트워크 일치 필수
**"네트워크가 일치하지 않는 경우 정상 입출금이 어려울 수 있으니"** 반드시 정확한 네트워크를 선택해야 합니다.

### 🔗 지원 네트워크 예시
- **BTC**: Bitcoin 네트워크
- **ETH**: Ethereum 네트워크  
- **USDT**: Ethereum, Tron, BSC 등 다중 네트워크 지원
- **XRP**: Ripple 네트워크 (Destination Tag 필요)

### 📝 2차 주소 (Secondary Address)
일부 디지털 자산은 메인 주소 외에 추가 정보가 필요합니다:
- **XRP**: Destination Tag
- **EOS**: Memo
- **STEEM**: Memo

## 주의사항

### 🔒 보안 요구사항
- 출금 기능을 사용하려면 주소 등록이 필수입니다.
- 등록된 주소로만 출금이 가능합니다.
- 잘못된 주소 등록 시 자산 손실 위험이 있습니다.

### 💡 사용 팁
- 출금 전에 이 API로 등록된 주소를 확인하세요.
- 네트워크 유형을 정확히 확인하세요.
- 2차 주소가 필요한 자산의 경우 반드시 입력하세요.