# Upbit WebSocket API 인증 가이드

## 인증 개요

Upbit WebSocket API는 Public 및 Private 데이터로 구분됩니다:
- **Public 데이터**: 별도의 인증 불필요
- **Private 데이터**: 인증 필수

## 인증 절차

### 인증 방법
- REST API와 동일하게 `Authorization` 헤더를 통해 인증
- JWT(JSON Web Token) 사용
- [인증 토큰 생성 가이드](/kr/docs/create-authorization-request) 참고

### 토큰 생성 단계
1. **Payload 구성**
   - `access_key`: 발급받은 Access Key
   - `nonce`: 고유한 UUID 생성

2. **JWT 토큰 서명**
   - Secret Key로 Payload 서명

## 코드 예시

### Node.js
```javascript
const jwt = require("jsonwebtoken");
const {v4: uuidv4} = require('uuid');
const WebSocket = require("ws");

const payload = {
    access_key: "발급받은 Access key",
    nonce: uuidv4(),
};

const jwtToken = jwt.sign(payload, "발급받은 Secret key");

const ws = new WebSocket("wss://api.upbit.com/websocket/v1/private", {
    headers: {
        authorization: `Bearer ${jwtToken}`
    }
});
```

### Go
```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "github.com/golang-jwt/jwt/v4"
    "github.com/google/uuid"
    "github.com/gorilla/websocket"
    "net/http"
)

func main() {
    // JWT 토큰 생성
    payload := jwt.MapClaims{
        "access_key": "발급받은 Access key",
        "nonce":      uuid.New().String(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, payload)
    tokenString, err := token.SignedString([]byte("발급받은 Secret key"))
    if err != nil {
        panic(err)
    }
    
    // WebSocket 연결
    header := http.Header{}
    header.Set("Authorization", "Bearer "+tokenString)
    
    conn, _, err := websocket.DefaultDialer.Dial("wss://api.upbit.com/websocket/v1/private", header)
    if err != nil {
        panic(err)
    }
    defer conn.Close()
}
```

## 주의사항
- **보안**: API Key와 Secret Key는 안전하게 관리
- **토큰 갱신**: 토큰은 요청마다 새로 생성
- **인증 실패**: 인증 실패 시 연결 거부됨
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리해야 함