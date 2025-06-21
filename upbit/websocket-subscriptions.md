# Upbit WebSocket API: 구독 중인 타입 조회

## 개요
WebSocket API에서 현재 구독 중인 데이터 타입을 조회하는 기능입니다.

## WebSocket 엔드포인트
```
wss://api.upbit.com/websocket/v1
wss://api.upbit.com/websocket/v1/private (Private 데이터용)
```

## 요청 형식

### 기본 요청 구조
```json
[
  {
    "method": "LIST_SUBSCRIPTIONS"
  },
  {
    "ticket": "unique uuid"
  }
]
```

### 포맷 지정 요청
```json
[
  {
    "method": "LIST_SUBSCRIPTIONS"
  },
  {
    "ticket": "subscription_check"
  },
  {
    "format": "SIMPLE"
  }
]
```

### 주요 요청 필드
| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `method` | String | 필수 | `"LIST_SUBSCRIPTIONS"` 고정 |
| `ticket` | String | 필수 | 요청자 식별 값 |
| `format` | String | 선택 | 응답 포맷 (`DEFAULT` 또는 `SIMPLE`) |

## 응답 형식

### 기본 응답 구조
```json
{
  "method": "LIST_SUBSCRIPTIONS",
  "result": [
    {
      "type": "ticker",
      "codes": ["KRW-BTC", "KRW-ETH"]
    },
    {
      "type": "orderbook",
      "codes": ["KRW-BTC", "KRW-ETH"],
      "level": 0
    }
  ],
  "ticket": "unique uuid"
}
```

### 주요 응답 필드
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `method` | 요청 메서드 | String | "LIST_SUBSCRIPTIONS" |
| `result` | 구독 타입 목록 | Array | 현재 구독 중인 모든 타입 |
| `ticket` | 요청 티켓 | String | 요청 시 제공한 값 |

### 구독 타입 정보 (result 배열 요소)
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `type` | 데이터 타입 | String | ticker, trade, orderbook 등 |
| `codes` | 마켓 코드 리스트 | Array | 구독 중인 마켓 목록 |
| `level` | 호가 모아보기 단위 | Double | orderbook 타입인 경우 |

## 응답 예시

### Public 데이터 구독 조회
```json
{
  "method": "LIST_SUBSCRIPTIONS",
  "result": [
    {
      "type": "ticker",
      "codes": ["KRW-BTC", "KRW-ETH", "KRW-XRP"]
    },
    {
      "type": "trade",
      "codes": ["KRW-BTC"]
    },
    {
      "type": "orderbook",
      "codes": ["KRW-BTC", "KRW-ETH"],
      "level": 1000
    },
    {
      "type": "candle.1m",
      "codes": ["KRW-BTC"]
    }
  ],
  "ticket": "public_subscription_check"
}
```

### Private 데이터 구독 조회
```json
{
  "method": "LIST_SUBSCRIPTIONS",
  "result": [
    {
      "type": "myOrder",
      "codes": ["KRW-BTC", "KRW-ETH"]
    },
    {
      "type": "myAsset"
    }
  ],
  "ticket": "private_subscription_check"
}
```

### 구독 없음
```json
{
  "method": "LIST_SUBSCRIPTIONS",
  "result": [],
  "ticket": "empty_subscription_check"
}
```

## Go 언어 구현 예시

### 구조체 정의
```go
package main

import (
    "encoding/json"
    "github.com/google/uuid"
    "github.com/gorilla/websocket"
)

// 구독 타입 정보 구조체
type SubscriptionInfo struct {
    Type  string   `json:"type"`
    Codes []string `json:"codes,omitempty"`
    Level float64  `json:"level,omitempty"`
}

// 구독 조회 응답 구조체
type ListSubscriptionsResponse struct {
    Method string             `json:"method"`
    Result []SubscriptionInfo `json:"result"`
    Ticket string             `json:"ticket"`
}

// 구독 조회 요청 구조체
type ListSubscriptionsRequest struct {
    Method string `json:"method"`
    Ticket string `json:"ticket"`
    Format string `json:"format,omitempty"`
}

// WebSocket 클라이언트 확장
type SubscriptionManager struct {
    conn *websocket.Conn
}

func NewSubscriptionManager(wsURL string) (*SubscriptionManager, error) {
    conn, _, err := websocket.DefaultDialer.Dial(wsURL, nil)
    if err != nil {
        return nil, err
    }
    
    return &SubscriptionManager{conn: conn}, nil
}

func (sm *SubscriptionManager) ListSubscriptions() (*ListSubscriptionsResponse, error) {
    // 구독 조회 요청 생성
    request := []interface{}{
        map[string]string{
            "method": "LIST_SUBSCRIPTIONS",
        },
        map[string]string{
            "ticket": uuid.New().String(),
        },
    }
    
    // 요청 전송
    if err := sm.conn.WriteJSON(request); err != nil {
        return nil, err
    }
    
    // 응답 수신
    var response ListSubscriptionsResponse
    if err := sm.conn.ReadJSON(&response); err != nil {
        return nil, err
    }
    
    return &response, nil
}

func (sm *SubscriptionManager) AddSubscription(subType string, codes []string, options map[string]interface{}) error {
    // 구독 요청 생성
    typeField := map[string]interface{}{
        "type":  subType,
        "codes": codes,
    }
    
    // 추가 옵션 설정 (예: level for orderbook)
    for key, value := range options {
        typeField[key] = value
    }
    
    request := []interface{}{
        map[string]string{"ticket": uuid.New().String()},
        typeField,
    }
    
    return sm.conn.WriteJSON(request)
}

func (sm *SubscriptionManager) Close() error {
    return sm.conn.Close()
}

// 구독 정보 분석 함수
func (sm *SubscriptionManager) AnalyzeSubscriptions() {
    response, err := sm.ListSubscriptions()
    if err != nil {
        fmt.Printf("구독 조회 실패: %v\n", err)
        return
    }
    
    fmt.Printf("=== 현재 구독 현황 ===\n")
    fmt.Printf("총 구독 타입: %d개\n\n", len(response.Result))
    
    if len(response.Result) == 0 {
        fmt.Println("현재 구독 중인 데이터가 없습니다.")
        return
    }
    
    for i, sub := range response.Result {
        fmt.Printf("%d. 타입: %s\n", i+1, sub.Type)
        
        if len(sub.Codes) > 0 {
            fmt.Printf("   마켓: %v\n", sub.Codes)
        } else {
            fmt.Printf("   마켓: 전체\n")
        }
        
        if sub.Level > 0 {
            fmt.Printf("   호가 단위: %.0f\n", sub.Level)
        }
        
        fmt.Println()
    }
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
    // 구독 관리자 생성
    manager, err := NewSubscriptionManager("wss://api.upbit.com/websocket/v1")
    if err != nil {
        log.Fatal("WebSocket 연결 실패:", err)
    }
    defer manager.Close()
    
    // 여러 구독 추가
    fmt.Println("구독 추가 중...")
    
    // Ticker 구독
    if err := manager.AddSubscription("ticker", []string{"KRW-BTC", "KRW-ETH"}, nil); err != nil {
        log.Printf("Ticker 구독 실패: %v", err)
    }
    
    // Trade 구독
    if err := manager.AddSubscription("trade", []string{"KRW-BTC"}, nil); err != nil {
        log.Printf("Trade 구독 실패: %v", err)
    }
    
    // Orderbook 구독 (1000원 단위)
    options := map[string]interface{}{"level": 1000}
    if err := manager.AddSubscription("orderbook", []string{"KRW-BTC", "KRW-ETH"}, options); err != nil {
        log.Printf("Orderbook 구독 실패: %v", err)
    }
    
    // 잠시 대기 (구독 설정 완료를 위해)
    time.Sleep(2 * time.Second)
    
    // 현재 구독 상태 확인
    manager.AnalyzeSubscriptions()
    
    // 주기적으로 구독 상태 모니터링
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            fmt.Println("\n--- 구독 상태 업데이트 ---")
            manager.AnalyzeSubscriptions()
        }
    }
}
```

### 구독 관리 고급 예시
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type AdvancedSubscriptionManager struct {
    conn             *websocket.Conn
    subscriptions    map[string]SubscriptionInfo
    mutex           sync.RWMutex
    monitorInterval time.Duration
}

func NewAdvancedSubscriptionManager(wsURL string, monitorInterval time.Duration) (*AdvancedSubscriptionManager, error) {
    conn, _, err := websocket.DefaultDialer.Dial(wsURL, nil)
    if err != nil {
        return nil, err
    }
    
    return &AdvancedSubscriptionManager{
        conn:            conn,
        subscriptions:   make(map[string]SubscriptionInfo),
        monitorInterval: monitorInterval,
    }, nil
}

func (asm *AdvancedSubscriptionManager) StartMonitoring() {
    go func() {
        ticker := time.NewTicker(asm.monitorInterval)
        defer ticker.Stop()
        
        for range ticker.C {
            asm.updateSubscriptionCache()
        }
    }()
}

func (asm *AdvancedSubscriptionManager) updateSubscriptionCache() {
    response, err := asm.ListSubscriptions()
    if err != nil {
        fmt.Printf("구독 상태 업데이트 실패: %v\n", err)
        return
    }
    
    asm.mutex.Lock()
    defer asm.mutex.Unlock()
    
    // 기존 구독 정보 초기화
    asm.subscriptions = make(map[string]SubscriptionInfo)
    
    // 새 구독 정보 저장
    for _, sub := range response.Result {
        key := fmt.Sprintf("%s_%v", sub.Type, sub.Codes)
        asm.subscriptions[key] = sub
    }
}

func (asm *AdvancedSubscriptionManager) GetSubscriptionCount() int {
    asm.mutex.RLock()
    defer asm.mutex.RUnlock()
    return len(asm.subscriptions)
}

func (asm *AdvancedSubscriptionManager) HasSubscription(subType string, codes []string) bool {
    asm.mutex.RLock()
    defer asm.mutex.RUnlock()
    
    key := fmt.Sprintf("%s_%v", subType, codes)
    _, exists := asm.subscriptions[key]
    return exists
}

func (asm *AdvancedSubscriptionManager) GetSubscriptionsByType(subType string) []SubscriptionInfo {
    asm.mutex.RLock()
    defer asm.mutex.RUnlock()
    
    var result []SubscriptionInfo
    for _, sub := range asm.subscriptions {
        if sub.Type == subType {
            result = append(result, sub)
        }
    }
    
    return result
}

func (asm *AdvancedSubscriptionManager) PrintSummary() {
    asm.mutex.RLock()
    defer asm.mutex.RUnlock()
    
    typeCount := make(map[string]int)
    marketSet := make(map[string]bool)
    
    for _, sub := range asm.subscriptions {
        typeCount[sub.Type]++
        for _, market := range sub.Codes {
            marketSet[market] = true
        }
    }
    
    fmt.Printf("구독 요약:\n")
    fmt.Printf("  총 구독 수: %d개\n", len(asm.subscriptions))
    fmt.Printf("  구독 타입별:\n")
    for subType, count := range typeCount {
        fmt.Printf("    %s: %d개\n", subType, count)
    }
    fmt.Printf("  모니터링 마켓: %d개\n", len(marketSet))
}

func (asm *AdvancedSubscriptionManager) ListSubscriptions() (*ListSubscriptionsResponse, error) {
    request := []interface{}{
        map[string]string{
            "method": "LIST_SUBSCRIPTIONS",
        },
        map[string]string{
            "ticket": uuid.New().String(),
        },
    }
    
    if err := asm.conn.WriteJSON(request); err != nil {
        return nil, err
    }
    
    var response ListSubscriptionsResponse
    if err := asm.conn.ReadJSON(&response); err != nil {
        return nil, err
    }
    
    return &response, nil
}

func (asm *AdvancedSubscriptionManager) Close() error {
    return asm.conn.Close()
}
```

## 주의사항

- **요청 수 제한**: 구독 조회도 API 요청 수 제한에 포함됨
- **실시간 반영**: 구독 추가/제거 후 즉시 반영되지 않을 수 있음
- **포맷 영향**: format 필드는 현재 연결의 전체 포맷에 영향을 줄 수 있음
- **Private 구독**: Private 엔드포인트의 경우 별도 인증 필요
- **연결별 구독**: 각 WebSocket 연결마다 독립적인 구독 상태 유지
- **구독 순서**: 구독 순서와 응답 순서가 다를 수 있음
- **중복 확인**: 동일한 구독을 중복으로 추가하지 않도록 주의
- **메모리 관리**: 장기간 연결 시 구독 정보 캐시 관리 필요
- **에러 처리**: 네트워크 오류 시 적절한 재시도 로직 구현 권장