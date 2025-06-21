# API 키 리스트 조회

## API 개요
계정에 등록된 Open API 키 목록과 만료 일자를 조회하는 API입니다. API 키 관리 및 모니터링을 위해 사용됩니다.

## 요청 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/api_keys`
- **인증**: JWT 토큰 필요

## 요청 파라미터
이 API는 별도의 요청 파라미터가 필요하지 않습니다.

## 응답 필드
구체적인 응답 필드는 문서에서 명시되지 않았으나, 일반적으로 다음과 같은 정보를 포함합니다:

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `access_key` | String | API 액세스 키 |
| `expire_at` | DateString | API 키 만료 일시 |
| `created_at` | DateString | API 키 생성 일시 |
| `permissions` | Array | API 키 권한 목록 |
| `allowed_ips` | Array | 허용된 IP 주소 목록 |

## 응답 예시

```json
[
  {
    "access_key": "aBcDeFgH1234567890",
    "expire_at": "2024-07-01T00:00:00+09:00",
    "created_at": "2023-07-01T10:30:00+09:00",
    "permissions": ["read", "trade"],
    "allowed_ips": ["192.168.1.100", "203.0.113.1"]
  }
]
```

## API 키 제한사항

### 키 개수 제한
- **최대 10개**: 계정당 최대 10개의 API 키 생성 가능
- **Secret Key 보안**: Secret Key는 생성 시에만 확인 가능하며, 이후에는 조회할 수 없습니다.

### 유효 기간
- **1년 유효**: API 키 토큰은 1년간 유효하며 연장 불가능
- **만료 전 재생성**: 만료 전에 새로운 API 키를 생성해야 합니다.

### 보안 요구사항
- **2차 인증**: API 키 관리 시 2단계 인증(2FA) 필요
- **사용 기능 지정**: 키 생성 시 사용할 기능을 명시해야 함
- **IP 주소 등록**: 등록된 IP 주소에서만 접근 가능

## IP 주소 제한

### 등록 가능 IP
- **최대 10개**: API 키당 최대 10개 IP 주소 등록 가능
- **사전 등록 필수**: 반드시 사전에 등록된 IP에서만 API 호출 가능
- **동적 IP 주의**: 동적 IP 환경에서는 주기적인 IP 업데이트가 필요할 수 있습니다.

## 권한 관리

### 기능별 권한
API 키 생성 시 다음과 같은 기능별 권한을 설정할 수 있습니다:
- **조회 (READ)**: 잔고, 주문 내역 등 조회 기능
- **거래 (TRADE)**: 주문 생성, 취소 등 거래 기능
- **출금 (WITHDRAW)**: 출금 요청 기능

## 주의사항

### 보안 관련
- **Secret Key 보관**: Secret Key는 안전한 곳에 보관하고 타인과 공유하지 마세요.
- **정기적 갱신**: 보안을 위해 주기적으로 API 키를 갱신하는 것을 권장합니다.
- **사용하지 않는 키 삭제**: 더 이상 사용하지 않는 API 키는 즉시 삭제하세요.

### 제한 정책 변경사항
- **v1.4.1 업데이트**: 요청 수 제한 정책이 '계정 단위'로 변경되었습니다.
- **Rate Limiting**: 계정 전체에 대한 API 호출 제한이 적용됩니다.

## 사용 사례

### API 키 만료 확인
```javascript
// API 키 만료 일자 확인
const apiKeys = await fetch('/v1/api_keys');
const soonToExpire = apiKeys.filter(key => {
    const expireDate = new Date(key.expire_at);
    const now = new Date();
    const daysUntilExpire = (expireDate - now) / (1000 * 60 * 60 * 24);
    return daysUntilExpire < 30; // 30일 이내 만료
});

if (soonToExpire.length > 0) {
    console.log('만료 예정 API 키:', soonToExpire);
}
```

### 권한별 키 관리
```javascript
// 권한별 API 키 분류
const readOnlyKeys = apiKeys.filter(key => 
    key.permissions.includes('read') && !key.permissions.includes('trade')
);
const tradeKeys = apiKeys.filter(key => 
    key.permissions.includes('trade')
);
```

## 관련 기능
- API 키 생성 (업비트 웹사이트에서 수행)
- API 키 삭제 (업비트 웹사이트에서 수행)
- 2단계 인증 설정
- IP 주소 관리