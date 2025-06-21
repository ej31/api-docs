# Upbit WebSocket API 에러 처리 가이드

## 에러 응답 형식

WebSocket 에러는 다음과 같은 JSON 형식으로 반환됩니다:

```json
{
  "error": {
    "name": "에러 종류",
    "message": "에러 메시지"
  }
}
```

## 주요 에러 유형

### 1. 인증 관련 에러

#### INVALID_AUTH
- **상황**: Private 타입 요청 시 인증 정보 오류
- **원인**: 
  - JWT 토큰이 잘못됨
  - 토큰 만료
  - API 키 또는 Secret 키 오류
- **예시**: 
```json
{
  "error": {
    "name": "INVALID_AUTH",
    "message": "인증 정보가 올바르지 않습니다."
  }
}
```
- **대응 방법**:
  - API 키와 Secret 키 확인
  - JWT 토큰 재생성
  - 토큰 서명 알고리즘 확인 (HS256)

### 2. 요청 형식 에러

#### WRONG_FORMAT
- **상황**: 요청 형식이 잘못된 경우
- **원인**:
  - JSON 형식 오류
  - 필수 필드 누락
  - 잘못된 데이터 타입
- **예시**:
```json
{
  "error": {
    "name": "WRONG_FORMAT",
    "message": "Format이 맞지 않습니다."
  }
}
```
- **대응 방법**:
  - 요청 JSON 형식 검토
  - 필수 필드 확인
  - 데이터 타입 확인

#### NO_TICKET
- **상황**: 유효하지 않은 티켓
- **원인**:
  - ticket 필드 누락
  - 빈 문자열 ticket
  - 잘못된 ticket 형식
- **예시**:
```json
{
  "error": {
    "name": "NO_TICKET",
    "message": "티켓이 존재하지 않거나, 유효하지 않습니다."
  }
}
```
- **대응 방법**:
  - ticket 필드 추가
  - 고유한 ticket 값 사용 (UUID 권장)

#### NO_TYPE
- **상황**: type 필드 누락
- **원인**:
  - type 필드가 없음
  - 빈 문자열 type
- **예시**:
```json
{
  "error": {
    "name": "NO_TYPE",
    "message": "type 필드가 존재하지 않습니다."
  }
}
```
- **대응 방법**:
  - type 필드 추가
  - 올바른 type 값 사용 (ticker, trade, orderbook 등)

#### NO_CODES
- **상황**: codes 필드 누락
- **원인**:
  - codes 필드가 없음
  - 빈 배열 codes
  - 잘못된 마켓 코드
- **예시**:
```json
{
  "error": {
    "name": "NO_CODES",
    "message": "codes 필드가 존재하지 않습니다."
  }
}
```
- **대응 방법**:
  - codes 필드 추가
  - 유효한 마켓 코드 사용 (대문자)
  - 최소 1개 이상의 마켓 코드 포함

### 3. 연결 관련 에러

#### CONNECTION_LIMIT_EXCEEDED
- **상황**: 연결 수 제한 초과
- **원인**:
  - 동시 연결 수 초과
  - 계정당 연결 제한 도달
- **대응 방법**:
  - 기존 연결 정리
  - 연결 수 관리
  - 연결 풀링 구현

#### RATE_LIMIT_EXCEEDED
- **상황**: 요청 빈도 제한 초과
- **원인**:
  - 너무 빈번한 요청
  - API 호출 제한 도달
- **대응 방법**:
  - 요청 간격 조정
  - 백오프 전략 구현
  - 구독 최적화

### 4. 데이터 관련 에러

#### INVALID_MARKET
- **상황**: 존재하지 않는 마켓 코드
- **원인**:
  - 잘못된 마켓 코드
  - 상장폐지된 마켓
  - 소문자 마켓 코드 사용
- **대응 방법**:
  - 유효한 마켓 코드 확인
  - 대문자 사용 (KRW-BTC)
  - 마켓 목록 API로 유효성 검증

#### UNSUPPORTED_TYPE
- **상황**: 지원하지 않는 데이터 타입
- **원인**:
  - 잘못된 type 값
  - deprecated 타입 사용
- **대응 방법**:
  - 지원되는 타입 확인
  - 최신 API 문서 참조

## Go 언어 에러 처리 구현

### 에러 구조체 정의
```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// WebSocket 에러 구조체
type WebSocketError struct {
    Error struct {
        Name    string `json:"name"`
        Message string `json:"message"`
    } `json:"error"`
}

// 커스텀 에러 타입들
type (
    InvalidAuthError       struct{ Msg string }
    WrongFormatError      struct{ Msg string }
    NoTicketError         struct{ Msg string }
    NoTypeError           struct{ Msg string }
    NoCodesError          struct{ Msg string }
    ConnectionLimitError   struct{ Msg string }
    RateLimitError        struct{ Msg string }
    InvalidMarketError    struct{ Msg string }
    UnsupportedTypeError  struct{ Msg string }
    UnknownError          struct{ Name, Msg string }
)

// 에러 인터페이스 구현
func (e InvalidAuthError) Error() string       { return fmt.Sprintf("인증 오류: %s", e.Msg) }
func (e WrongFormatError) Error() string      { return fmt.Sprintf("형식 오류: %s", e.Msg) }
func (e NoTicketError) Error() string         { return fmt.Sprintf("티켓 오류: %s", e.Msg) }
func (e NoTypeError) Error() string           { return fmt.Sprintf("타입 오류: %s", e.Msg) }
func (e NoCodesError) Error() string          { return fmt.Sprintf("코드 오류: %s", e.Msg) }
func (e ConnectionLimitError) Error() string   { return fmt.Sprintf("연결 제한: %s", e.Msg) }
func (e RateLimitError) Error() string        { return fmt.Sprintf("요청 제한: %s", e.Msg) }
func (e InvalidMarketError) Error() string    { return fmt.Sprintf("마켓 오류: %s", e.Msg) }
func (e UnsupportedTypeError) Error() string  { return fmt.Sprintf("타입 미지원: %s", e.Msg) }
func (e UnknownError) Error() string          { return fmt.Sprintf("알 수 없는 오류 [%s]: %s", e.Name, e.Msg) }

// 에러 파싱 함수
func ParseWebSocketError(data []byte) error {
    var wsError WebSocketError
    if err := json.Unmarshal(data, &wsError); err != nil {
        return fmt.Errorf("에러 파싱 실패: %v", err)
    }
    
    switch wsError.Error.Name {
    case "INVALID_AUTH":
        return InvalidAuthError{Msg: wsError.Error.Message}
    case "WRONG_FORMAT":
        return WrongFormatError{Msg: wsError.Error.Message}
    case "NO_TICKET":
        return NoTicketError{Msg: wsError.Error.Message}
    case "NO_TYPE":
        return NoTypeError{Msg: wsError.Error.Message}
    case "NO_CODES":
        return NoCodesError{Msg: wsError.Error.Message}
    case "CONNECTION_LIMIT_EXCEEDED":
        return ConnectionLimitError{Msg: wsError.Error.Message}
    case "RATE_LIMIT_EXCEEDED":
        return RateLimitError{Msg: wsError.Error.Message}
    case "INVALID_MARKET":
        return InvalidMarketError{Msg: wsError.Error.Message}
    case "UNSUPPORTED_TYPE":
        return UnsupportedTypeError{Msg: wsError.Error.Message}
    default:
        return UnknownError{Name: wsError.Error.Name, Msg: wsError.Error.Message}
    }
}

// WebSocket 메시지가 에러인지 확인
func IsErrorMessage(data []byte) bool {
    var check map[string]interface{}
    if err := json.Unmarshal(data, &check); err != nil {
        return false
    }
    
    _, exists := check["error"]
    return exists
}
```

### 에러 처리 클라이언트
```go
package main

import (
    "encoding/json"
    "log"
    "time"
    "github.com/gorilla/websocket"
)

type ErrorHandlingClient struct {
    conn           *websocket.Conn
    retryCount     int
    maxRetries     int
    retryInterval  time.Duration
    errorCallback  func(error)
}

func NewErrorHandlingClient(wsURL string, maxRetries int, retryInterval time.Duration) *ErrorHandlingClient {
    return &ErrorHandlingClient{
        maxRetries:    maxRetries,
        retryInterval: retryInterval,
        errorCallback: func(err error) {
            log.Printf("WebSocket 에러: %v", err)
        },
    }
}

func (ehc *ErrorHandlingClient) SetErrorCallback(callback func(error)) {
    ehc.errorCallback = callback
}

func (ehc *ErrorHandlingClient) Connect(wsURL string) error {
    conn, _, err := websocket.DefaultDialer.Dial(wsURL, nil)
    if err != nil {
        return err
    }
    
    ehc.conn = conn
    ehc.retryCount = 0
    return nil
}

func (ehc *ErrorHandlingClient) SendRequest(request interface{}) error {
    if ehc.conn == nil {
        return fmt.Errorf("연결이 없습니다")
    }
    
    return ehc.conn.WriteJSON(request)
}

func (ehc *ErrorHandlingClient) ReadMessage() ([]byte, error) {
    if ehc.conn == nil {
        return nil, fmt.Errorf("연결이 없습니다")
    }
    
    _, data, err := ehc.conn.ReadMessage()
    if err != nil {
        return nil, err
    }
    
    // 에러 메시지 확인
    if IsErrorMessage(data) {
        wsErr := ParseWebSocketError(data)
        ehc.handleError(wsErr)
        return nil, wsErr
    }
    
    return data, nil
}

func (ehc *ErrorHandlingClient) handleError(err error) {
    if ehc.errorCallback != nil {
        ehc.errorCallback(err)
    }
    
    // 에러 타입별 처리
    switch e := err.(type) {
    case InvalidAuthError:
        log.Printf("인증 오류 발생: %v", e)
        // 인증 토큰 재생성 로직
        
    case RateLimitError:
        log.Printf("요청 제한 도달: %v", e)
        // 백오프 대기
        time.Sleep(ehc.retryInterval)
        
    case ConnectionLimitError:
        log.Printf("연결 제한 도달: %v", e)
        // 기존 연결 정리 후 재연결
        
    case WrongFormatError, NoTicketError, NoTypeError, NoCodesError:
        log.Printf("요청 형식 오류: %v", e)
        // 요청 형식 수정 필요
        
    case InvalidMarketError:
        log.Printf("잘못된 마켓 코드: %v", e)
        // 마켓 코드 검증 및 수정
        
    default:
        log.Printf("처리되지 않은 에러: %v", e)
    }
}

func (ehc *ErrorHandlingClient) ConnectWithRetry(wsURL string) error {
    for ehc.retryCount < ehc.maxRetries {
        err := ehc.Connect(wsURL)
        if err == nil {
            return nil
        }
        
        log.Printf("연결 실패 (시도 %d/%d): %v", ehc.retryCount+1, ehc.maxRetries, err)
        ehc.retryCount++
        
        if ehc.retryCount < ehc.maxRetries {
            time.Sleep(ehc.retryInterval)
        }
    }
    
    return fmt.Errorf("최대 재시도 횟수 초과")
}

func (ehc *ErrorHandlingClient) Close() error {
    if ehc.conn != nil {
        return ehc.conn.Close()
    }
    return nil
}
```

### 사용 예시
```go
package main

import (
    "fmt"
    "log"
    "time"
)

func main() {
    // 에러 처리 클라이언트 생성
    client := NewErrorHandlingClient("wss://api.upbit.com/websocket/v1", 3, 5*time.Second)
    
    // 커스텀 에러 콜백 설정
    client.SetErrorCallback(func(err error) {
        switch e := err.(type) {
        case InvalidAuthError:
            fmt.Println("⚠️ 인증 정보를 확인해주세요")
        case RateLimitError:
            fmt.Println("⏱️ 요청 속도를 줄여주세요")
        case WrongFormatError:
            fmt.Println("📝 요청 형식을 확인해주세요")
        default:
            fmt.Printf("❌ 에러 발생: %v\n", e)
        }
    })
    
    // 재시도와 함께 연결
    if err := client.ConnectWithRetry("wss://api.upbit.com/websocket/v1"); err != nil {
        log.Fatal("연결 실패:", err)
    }
    defer client.Close()
    
    // 잘못된 요청 예시 (에러 테스트용)
    badRequest := []interface{}{
        map[string]string{}, // ticket 없음 -> NO_TICKET 에러
    }
    
    if err := client.SendRequest(badRequest); err != nil {
        log.Printf("요청 전송 실패: %v", err)
    }
    
    // 올바른 요청
    goodRequest := []interface{}{
        map[string]string{"ticket": "test"},
        map[string]interface{}{
            "type":  "ticker",
            "codes": []string{"KRW-BTC"},
        },
    }
    
    if err := client.SendRequest(goodRequest); err != nil {
        log.Printf("요청 전송 실패: %v", err)
    }
    
    // 메시지 수신 루프
    for {
        data, err := client.ReadMessage()
        if err != nil {
            log.Printf("메시지 수신 실패: %v", err)
            continue
        }
        
        fmt.Printf("수신된 데이터: %s\n", string(data))
    }
}
```

## 에러 예방 체크리스트

### 요청 전 검증
```go
func ValidateRequest(request []interface{}) error {
    if len(request) < 2 {
        return fmt.Errorf("최소 2개의 객체가 필요합니다")
    }
    
    // Ticket 검증
    if ticketObj, ok := request[0].(map[string]interface{}); ok {
        if ticket, exists := ticketObj["ticket"]; !exists || ticket == "" {
            return fmt.Errorf("유효한 ticket이 필요합니다")
        }
    }
    
    // Type 검증
    if typeObj, ok := request[1].(map[string]interface{}); ok {
        if dataType, exists := typeObj["type"]; !exists || dataType == "" {
            return fmt.Errorf("유효한 type이 필요합니다")
        }
        
        // Codes 검증 (Private 타입 제외)
        if dataType != "myOrder" && dataType != "myAsset" {
            if codes, exists := typeObj["codes"]; !exists {
                return fmt.Errorf("codes 필드가 필요합니다")
            } else if codesSlice, ok := codes.([]string); !ok || len(codesSlice) == 0 {
                return fmt.Errorf("최소 1개의 마켓 코드가 필요합니다")
            }
        }
    }
    
    return nil
}
```

## 주의사항

- **에러 로깅**: 모든 에러는 로그로 기록하여 디버깅에 활용
- **재시도 로직**: 일시적 에러의 경우 적절한 재시도 구현
- **백오프 전략**: Rate Limit 에러 시 지수 백오프 적용
- **연결 관리**: 연결 에러 시 자동 재연결 로직 구현
- **에러 분류**: 복구 가능한 에러와 불가능한 에러 구분
- **사용자 알림**: 중요한 에러는 사용자에게 적절히 알림
- **모니터링**: 에러 발생 빈도와 패턴 모니터링
- **문서 업데이트**: 새로운 에러 타입 발견 시 문서 업데이트