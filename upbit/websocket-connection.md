# Upbit WebSocket API 연결 관리

## 연결 개요

Upbit WebSocket API는 안정적인 실시간 데이터 스트림을 제공하기 위해 다양한 연결 관리 기능을 제공합니다.

## WebSocket 엔드포인트
- **Public**: `wss://api.upbit.com/websocket/v1`
- **Private**: `wss://api.upbit.com/websocket/v1/private`

## PING/PONG 메커니즘

### 연결 유지 필요성
- 서버는 약 **120초** 동안 데이터 미수신 시 WebSocket 연결을 종료합니다
- 클라이언트는 주기적으로 연결 상태를 확인해야 합니다

### 연결 유지 방법

#### 1. WebSocket PING Frame 전송
대부분의 WebSocket 라이브러리에 내장된 ping 함수를 활용하는 것이 권장됩니다.

```go
// Go에서 gorilla/websocket 사용 예시
conn.WriteMessage(websocket.PingMessage, []byte{})
```

#### 2. 상태 확인 메시지 전송
```json
{"status":"UP"}
```

### 권장 설정
- **PING 간격**: 10초마다 전송
- **타임아웃**: 30초 이내 PONG 응답 없으면 재연결
- **상태 메시지**: 10초 간격으로 수신 가능

## WebSocket 압축

### 압축 지원 특징
- **WebSocket Compression** 제공
- 클라이언트별 압축 옵션 활성화 가능
- 사용자 코드 레벨에서는 압축 해제된 raw 데이터 제공

### 호환성
- 압축을 지원하는 WebSocket 클라이언트 필요
- 미지원 클라이언트는 Raw JSON 데이터 사용
- 대부분의 최신 WebSocket 라이브러리에서 지원

## Go 언어 연결 관리 구현

### 기본 연결 관리 클래스
```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "sync"
    "time"
    
    "github.com/gorilla/websocket"
)

type ConnectionManager struct {
    url             string
    conn            *websocket.Conn
    isConnected     bool
    mutex           sync.RWMutex
    pingInterval    time.Duration
    pingTimeout     time.Duration
    reconnectDelay  time.Duration
    maxRetries      int
    retryCount      int
    stopChan        chan struct{}
    messageChan     chan []byte
    errorChan       chan error
    onConnect       func()
    onDisconnect    func()
    onError         func(error)
}

func NewConnectionManager(url string) *ConnectionManager {
    return &ConnectionManager{
        url:            url,
        pingInterval:   10 * time.Second,
        pingTimeout:    30 * time.Second,
        reconnectDelay: 5 * time.Second,
        maxRetries:     5,
        stopChan:       make(chan struct{}),
        messageChan:    make(chan []byte, 100),
        errorChan:      make(chan error, 10),
    }
}

func (cm *ConnectionManager) SetCallbacks(onConnect, onDisconnect func(), onError func(error)) {
    cm.onConnect = onConnect
    cm.onDisconnect = onDisconnect
    cm.onError = onError
}

func (cm *ConnectionManager) Connect() error {
    cm.mutex.Lock()
    defer cm.mutex.Unlock()
    
    if cm.isConnected {
        return fmt.Errorf("이미 연결되어 있습니다")
    }
    
    // WebSocket 압축 옵션 활성화
    dialer := websocket.Dialer{
        EnableCompression: true,
        HandshakeTimeout:  30 * time.Second,
    }
    
    conn, _, err := dialer.Dial(cm.url, nil)
    if err != nil {
        return fmt.Errorf("WebSocket 연결 실패: %v", err)
    }
    
    cm.conn = conn
    cm.isConnected = true
    cm.retryCount = 0
    
    // 연결 성공 콜백
    if cm.onConnect != nil {
        cm.onConnect()
    }
    
    // 백그라운드 작업 시작
    go cm.readPump()
    go cm.writePump()
    go cm.pingPump()
    
    return nil
}

func (cm *ConnectionManager) Disconnect() {
    cm.mutex.Lock()
    defer cm.mutex.Unlock()
    
    if !cm.isConnected {
        return
    }
    
    close(cm.stopChan)
    
    if cm.conn != nil {
        cm.conn.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""))
        cm.conn.Close()
    }
    
    cm.isConnected = false
    
    // 연결 해제 콜백
    if cm.onDisconnect != nil {
        cm.onDisconnect()
    }
}

func (cm *ConnectionManager) IsConnected() bool {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    return cm.isConnected
}

func (cm *ConnectionManager) SendMessage(data interface{}) error {
    cm.mutex.RLock()
    defer cm.mutex.RUnlock()
    
    if !cm.isConnected || cm.conn == nil {
        return fmt.Errorf("연결이 없습니다")
    }
    
    return cm.conn.WriteJSON(data)
}

func (cm *ConnectionManager) GetMessageChan() <-chan []byte {
    return cm.messageChan
}

func (cm *ConnectionManager) GetErrorChan() <-chan error {
    return cm.errorChan
}

// 메시지 수신 펌프
func (cm *ConnectionManager) readPump() {
    defer func() {
        cm.handleDisconnection()
    }()
    
    // Read timeout 설정
    cm.conn.SetReadDeadline(time.Now().Add(cm.pingTimeout))
    cm.conn.SetPongHandler(func(string) error {
        cm.conn.SetReadDeadline(time.Now().Add(cm.pingTimeout))
        return nil
    })
    
    for {
        select {
        case <-cm.stopChan:
            return
        default:
            _, message, err := cm.conn.ReadMessage()
            if err != nil {
                if websocket.IsUnexpectedCloseError(err, websocket.CloseGoingAway, websocket.CloseAbnormalClosure) {
                    cm.errorChan <- fmt.Errorf("읽기 오류: %v", err)
                }
                return
            }
            
            cm.messageChan <- message
        }
    }
}

// 메시지 전송 펌프 (향후 확장용)
func (cm *ConnectionManager) writePump() {
    defer func() {
        cm.handleDisconnection()
    }()
    
    for {
        select {
        case <-cm.stopChan:
            return
        }
    }
}

// PING 전송 펌프
func (cm *ConnectionManager) pingPump() {
    ticker := time.NewTicker(cm.pingInterval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            cm.mutex.RLock()
            if cm.conn != nil && cm.isConnected {
                cm.conn.SetWriteDeadline(time.Now().Add(10 * time.Second))
                if err := cm.conn.WriteMessage(websocket.PingMessage, nil); err != nil {
                    cm.mutex.RUnlock()
                    cm.errorChan <- fmt.Errorf("PING 전송 실패: %v", err)
                    return
                }
            }
            cm.mutex.RUnlock()
        case <-cm.stopChan:
            return
        }
    }
}

// 연결 해제 처리
func (cm *ConnectionManager) handleDisconnection() {
    cm.mutex.Lock()
    wasConnected := cm.isConnected
    cm.isConnected = false
    cm.mutex.Unlock()
    
    if wasConnected && cm.onDisconnect != nil {
        cm.onDisconnect()
    }
    
    // 자동 재연결 시도
    go cm.attemptReconnection()
}

// 재연결 시도
func (cm *ConnectionManager) attemptReconnection() {
    for cm.retryCount < cm.maxRetries {
        time.Sleep(cm.reconnectDelay)
        
        log.Printf("재연결 시도 중... (%d/%d)", cm.retryCount+1, cm.maxRetries)
        
        if err := cm.Connect(); err != nil {
            cm.retryCount++
            log.Printf("재연결 실패: %v", err)
            continue
        }
        
        log.Println("재연결 성공!")
        return
    }
    
    log.Printf("최대 재연결 시도 횟수 초과 (%d)", cm.maxRetries)
    if cm.onError != nil {
        cm.onError(fmt.Errorf("재연결 실패: 최대 시도 횟수 초과"))
    }
}
```

### 고급 연결 관리자
```go
type AdvancedConnectionManager struct {
    *ConnectionManager
    subscriptions    map[string]interface{}
    healthCheck      *time.Ticker
    lastMessageTime  time.Time
    statistics       ConnectionStats
}

type ConnectionStats struct {
    ConnectTime      time.Time
    MessagesReceived int64
    MessagesSent     int64
    Reconnections    int
    LastError        error
    LastErrorTime    time.Time
}

func NewAdvancedConnectionManager(url string) *AdvancedConnectionManager {
    base := NewConnectionManager(url)
    
    acm := &AdvancedConnectionManager{
        ConnectionManager: base,
        subscriptions:     make(map[string]interface{}),
        lastMessageTime:   time.Now(),
    }
    
    // 콜백 설정
    base.SetCallbacks(
        acm.handleConnect,
        acm.handleDisconnect,
        acm.handleError,
    )
    
    return acm
}

func (acm *AdvancedConnectionManager) handleConnect() {
    acm.statistics.ConnectTime = time.Now()
    log.Println("✅ WebSocket 연결 성공")
    
    // 기존 구독 복원
    acm.restoreSubscriptions()
    
    // 헬스체크 시작
    acm.startHealthCheck()
}

func (acm *AdvancedConnectionManager) handleDisconnect() {
    log.Println("❌ WebSocket 연결 해제")
    acm.stopHealthCheck()
}

func (acm *AdvancedConnectionManager) handleError(err error) {
    acm.statistics.LastError = err
    acm.statistics.LastErrorTime = time.Now()
    log.Printf("⚠️ WebSocket 에러: %v", err)
}

func (acm *AdvancedConnectionManager) Subscribe(request interface{}, key string) error {
    // 구독 정보 저장
    acm.subscriptions[key] = request
    
    // 즉시 전송
    return acm.SendMessage(request)
}

func (acm *AdvancedConnectionManager) restoreSubscriptions() {
    for key, request := range acm.subscriptions {
        if err := acm.SendMessage(request); err != nil {
            log.Printf("구독 복원 실패 [%s]: %v", key, err)
        } else {
            log.Printf("구독 복원 성공 [%s]", key)
        }
    }
}

func (acm *AdvancedConnectionManager) startHealthCheck() {
    acm.healthCheck = time.NewTicker(30 * time.Second)
    
    go func() {
        for range acm.healthCheck.C {
            if time.Since(acm.lastMessageTime) > 60*time.Second {
                log.Println("⚠️ 메시지 수신이 오랫동안 없습니다. 연결 상태를 확인합니다.")
                
                // 상태 확인 메시지 전송
                statusMsg := map[string]string{"status": "UP"}
                if err := acm.SendMessage(statusMsg); err != nil {
                    log.Printf("상태 확인 메시지 전송 실패: %v", err)
                }
            }
        }
    }()
}

func (acm *AdvancedConnectionManager) stopHealthCheck() {
    if acm.healthCheck != nil {
        acm.healthCheck.Stop()
        acm.healthCheck = nil
    }
}

func (acm *AdvancedConnectionManager) ProcessMessage(data []byte) {
    acm.lastMessageTime = time.Now()
    acm.statistics.MessagesReceived++
    
    // 메시지 처리 로직
    var message map[string]interface{}
    if err := json.Unmarshal(data, &message); err != nil {
        log.Printf("메시지 파싱 실패: %v", err)
        return
    }
    
    // 상태 메시지 확인
    if status, exists := message["status"]; exists && status == "UP" {
        log.Println("💚 서버 상태 확인 완료")
        return
    }
    
    // 일반 데이터 메시지 처리
    // 여기서 실제 비즈니스 로직 처리
}

func (acm *AdvancedConnectionManager) GetStatistics() ConnectionStats {
    return acm.statistics
}

func (acm *AdvancedConnectionManager) PrintStatistics() {
    stats := acm.GetStatistics()
    
    fmt.Println("=== 연결 통계 ===")
    fmt.Printf("연결 시간: %v\n", stats.ConnectTime.Format("2006-01-02 15:04:05"))
    fmt.Printf("수신 메시지: %d개\n", stats.MessagesReceived)
    fmt.Printf("전송 메시지: %d개\n", stats.MessagesSent)
    fmt.Printf("재연결 횟수: %d회\n", stats.Reconnections)
    
    if !stats.LastErrorTime.IsZero() {
        fmt.Printf("마지막 에러: %v (%v)\n", stats.LastError, stats.LastErrorTime.Format("15:04:05"))
    }
    
    if acm.IsConnected() {
        uptime := time.Since(stats.ConnectTime)
        fmt.Printf("연결 지속 시간: %v\n", uptime.Round(time.Second))
    }
}
```

### 사용 예시
```go
func main() {
    // 고급 연결 관리자 생성
    manager := NewAdvancedConnectionManager("wss://api.upbit.com/websocket/v1")
    
    // 연결
    if err := manager.Connect(); err != nil {
        log.Fatal("연결 실패:", err)
    }
    defer manager.Disconnect()
    
    // 구독 추가
    tickerRequest := []interface{}{
        map[string]string{"ticket": "ticker_sub"},
        map[string]interface{}{
            "type":  "ticker",
            "codes": []string{"KRW-BTC"},
        },
    }
    
    if err := manager.Subscribe(tickerRequest, "ticker_btc"); err != nil {
        log.Printf("구독 실패: %v", err)
    }
    
    // 메시지 처리 루프
    go func() {
        for message := range manager.GetMessageChan() {
            manager.ProcessMessage(message)
        }
    }()
    
    // 에러 처리 루프
    go func() {
        for err := range manager.GetErrorChan() {
            log.Printf("에러 발생: %v", err)
        }
    }()
    
    // 주기적 통계 출력
    statsTicker := time.NewTicker(60 * time.Second)
    defer statsTicker.Stop()
    
    for range statsTicker.C {
        manager.PrintStatistics()
    }
}
```

## 주요 연결 관리 팁

### 1. 안정적인 연결 유지
- **주기적 PING**: 10초 간격으로 PING 전송
- **타임아웃 설정**: 적절한 읽기/쓰기 타임아웃 설정
- **상태 모니터링**: 연결 상태 실시간 모니터링

### 2. 자동 재연결
- **지수 백오프**: 재연결 간격을 점진적으로 증가
- **최대 시도 횟수**: 무한 재연결 방지
- **구독 복원**: 재연결 시 기존 구독 자동 복원

### 3. 압축 최적화
- **압축 활성화**: 대역폭 절약을 위한 압축 사용
- **호환성 확인**: 클라이언트 라이브러리 압축 지원 확인

### 4. 에러 처리
- **적절한 로깅**: 연결 관련 모든 이벤트 로깅
- **에러 분류**: 복구 가능/불가능 에러 구분
- **알림 시스템**: 중요한 연결 문제 알림

### 5. 성능 모니터링
- **통계 수집**: 메시지 수, 연결 시간 등 통계
- **지연 시간 측정**: 메시지 수신 지연 모니터링
- **리소스 사용량**: 메모리, CPU 사용량 추적

## 주의사항

- **연결 제한**: 계정당 동시 연결 수 제한 존재
- **메시지 버퍼**: 대용량 메시지 처리 시 적절한 버퍼 크기 설정
- **고루틴 관리**: 고루틴 누수 방지를 위한 적절한 종료 처리
- **메모리 관리**: 장기간 연결 시 메모리 누수 모니터링
- **시간대 처리**: 모든 시간 데이터는 KST 기준으로 처리
- **보안**: Private 엔드포인트 사용 시 토큰 보안 관리