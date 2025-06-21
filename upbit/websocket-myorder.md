# Upbit WebSocket API: 내 주문 (MyOrder) 가이드

## 개요
Upbit의 WebSocket MyOrder 타입은 사용자의 주문 및 체결 정보를 실시간으로 제공하는 Private 엔드포인트입니다.

## WebSocket 엔드포인트
```
wss://api.upbit.com/websocket/v1/private
```

## 인증 요구사항
- **Private 엔드포인트**: JWT 토큰 기반 인증 필수
- **Authorization 헤더**: `Bearer {JWT_TOKEN}` 형식
- 자세한 인증 방법은 [인증 가이드](./websocket-authentication.md) 참조

## 데이터 전송 특징
- **이벤트 기반**: 주문 생성/체결/취소 시에만 데이터 전송
- **실시간 알림**: 특정 주문 이벤트 발생 시 즉시 전송
- **초기 데이터**: 연결 후 즉시 데이터가 수신되지 않을 수 있음

## 요청 형식

### 기본 요청 구조
```json
[
  {
    "ticket": "test-myOrder"
  },
  {
    "type": "myOrder",
    "codes": ["KRW-BTC"]
  }
]
```

### 전체 마켓 주문 정보 요청
```json
[
  {
    "ticket": "all-myOrder"
  },
  {
    "type": "myOrder"
  }
]
```

### 주요 요청 필드
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `ticket` | String | 필수 | 요청 식별자 |
| `type` | String | 필수 | `"myOrder"` 고정 |
| `codes` | Array | 선택 | 특정 마켓 코드 (생략 시 모든 마켓) |

## 응답 필드

### 주요 응답 필드
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `type` | 데이터 타입 | String | "myOrder" |
| `code` | 마켓 코드 | String | 예: "KRW-BTC" |
| `uuid` | 주문 고유 ID | String | 주문 식별자 |
| `ask_bid` | 매수/매도 구분 | String | ASK(매도)/BID(매수) |
| `state` | 주문 상태 | String | wait/watch/done/cancel |
| `market` | 마켓 코드 | String | code와 동일 |
| `created_at` | 주문 생성 시각 | String | ISO 8601 형식 |
| `volume` | 주문량 | String | 전체 주문량 |
| `remaining_volume` | 잔여 주문량 | String | 미체결 수량 |
| `reserved_fee` | 수수료 | String | |
| `remaining_fee` | 잔여 수수료 | String | |
| `paid_fee` | 지불된 수수료 | String | |
| `locked` | 묶인 금액 | String | |
| `executed_volume` | 체결량 | String | |
| `trades_count` | 체결 횟수 | Integer | |
| `order_type` | 주문 타입 | String | limit/price/market |
| `price` | 주문 가격 | String | 지정가 주문 시 |
| `avg_price` | 평균 체결 가격 | String | |
| `identifier` | 조회용 사용자 지정값 | String | |
| `time_in_force` | 주문 조건 | String | IOC/FOK 등 |
| `stream_type` | 스트림 타입 | String | REALTIME |

### 주문 상태 (state)
| 상태 | 설명 |
|------|------|
| `wait` | 체결 대기 |
| `watch` | 예약 주문 대기 |
| `done` | 전체 체결 완료 |
| `cancel` | 주문 취소 |

### 주문 타입 (order_type)
| 타입 | 설명 |
|------|------|
| `limit` | 지정가 주문 |
| `price` | 시장가 주문(매수) |
| `market` | 시장가 주문(매도) |

## 응답 예시

### 주문 생성 시
```json
{
  "type": "myOrder",
  "code": "KRW-BTC",
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "ask_bid": "BID",
  "state": "wait",
  "market": "KRW-BTC",
  "created_at": "2024-01-01T12:00:00+09:00",
  "volume": "0.001",
  "remaining_volume": "0.001",
  "reserved_fee": "50.0",
  "remaining_fee": "50.0",
  "paid_fee": "0.0",
  "locked": "100050.0",
  "executed_volume": "0.0",
  "trades_count": 0,
  "order_type": "limit",
  "price": "100000000.0",
  "avg_price": "0.0",
  "identifier": null,
  "time_in_force": "IOC",
  "stream_type": "REALTIME"
}
```

### 부분 체결 시
```json
{
  "type": "myOrder",
  "code": "KRW-BTC",
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "ask_bid": "BID",
  "state": "wait",
  "market": "KRW-BTC",
  "created_at": "2024-01-01T12:00:00+09:00",
  "volume": "0.001",
  "remaining_volume": "0.0005",
  "reserved_fee": "50.0",
  "remaining_fee": "25.0",
  "paid_fee": "25.0",
  "locked": "50025.0",
  "executed_volume": "0.0005",
  "trades_count": 1,
  "order_type": "limit",
  "price": "100000000.0",
  "avg_price": "100000000.0",
  "identifier": null,
  "time_in_force": "IOC",
  "stream_type": "REALTIME"
}
```

## Go 언어 구현 예시

### 구조체 정의
```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "time"
    "github.com/golang-jwt/jwt/v4"
    "github.com/google/uuid"
    "github.com/gorilla/websocket"
    "net/http"
)

// MyOrder 응답 구조체
type MyOrderResponse struct {
    Type            string `json:"type"`
    Code            string `json:"code"`
    UUID            string `json:"uuid"`
    AskBid          string `json:"ask_bid"`
    State           string `json:"state"`
    Market          string `json:"market"`
    CreatedAt       string `json:"created_at"`
    Volume          string `json:"volume"`
    RemainingVolume string `json:"remaining_volume"`
    ReservedFee     string `json:"reserved_fee"`
    RemainingFee    string `json:"remaining_fee"`
    PaidFee         string `json:"paid_fee"`
    Locked          string `json:"locked"`
    ExecutedVolume  string `json:"executed_volume"`
    TradesCount     int    `json:"trades_count"`
    OrderType       string `json:"order_type"`
    Price           string `json:"price"`
    AvgPrice        string `json:"avg_price"`
    Identifier      string `json:"identifier"`
    TimeInForce     string `json:"time_in_force"`
    StreamType      string `json:"stream_type"`
}

// 주문 생성 시간 파싱
func (m *MyOrderResponse) GetCreatedTime() (time.Time, error) {
    return time.Parse(time.RFC3339, m.CreatedAt)
}

// 매수/매도 구분 한글 변환
func (m *MyOrderResponse) GetAskBidKorean() string {
    switch m.AskBid {
    case "ASK":
        return "매도"
    case "BID":
        return "매수"
    default:
        return "알 수 없음"
    }
}

// 주문 상태 한글 변환
func (m *MyOrderResponse) GetStateKorean() string {
    switch m.State {
    case "wait":
        return "체결 대기"
    case "watch":
        return "예약 주문 대기"
    case "done":
        return "체결 완료"
    case "cancel":
        return "주문 취소"
    default:
        return "알 수 없음"
    }
}

// 주문 타입 한글 변환
func (m *MyOrderResponse) GetOrderTypeKorean() string {
    switch m.OrderType {
    case "limit":
        return "지정가"
    case "price":
        return "시장가(매수)"
    case "market":
        return "시장가(매도)"
    default:
        return "알 수 없음"
    }
}

// Private WebSocket 연결 설정
type PrivateWebSocketClient struct {
    accessKey string
    secretKey string
    conn      *websocket.Conn
}

func NewPrivateWebSocketClient(accessKey, secretKey string) *PrivateWebSocketClient {
    return &PrivateWebSocketClient{
        accessKey: accessKey,
        secretKey: secretKey,
    }
}

func (p *PrivateWebSocketClient) Connect() error {
    // JWT 토큰 생성
    payload := jwt.MapClaims{
        "access_key": p.accessKey,
        "nonce":      uuid.New().String(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, payload)
    tokenString, err := token.SignedString([]byte(p.secretKey))
    if err != nil {
        return err
    }
    
    // WebSocket 연결
    header := http.Header{}
    header.Set("Authorization", "Bearer "+tokenString)
    
    conn, _, err := websocket.DefaultDialer.Dial("wss://api.upbit.com/websocket/v1/private", header)
    if err != nil {
        return err
    }
    
    p.conn = conn
    return nil
}

func (p *PrivateWebSocketClient) SubscribeMyOrder(codes []string) error {
    request := []interface{}{
        map[string]string{"ticket": "myOrder_" + uuid.New().String()},
        map[string]interface{}{
            "type":  "myOrder",
            "codes": codes,
        },
    }
    
    return p.conn.WriteJSON(request)
}

func (p *PrivateWebSocketClient) SubscribeAllMyOrders() error {
    request := []interface{}{
        map[string]string{"ticket": "allMyOrder_" + uuid.New().String()},
        map[string]interface{}{
            "type": "myOrder",
        },
    }
    
    return p.conn.WriteJSON(request)
}

func (p *PrivateWebSocketClient) ReadMessage() (*MyOrderResponse, error) {
    var order MyOrderResponse
    err := p.conn.ReadJSON(&order)
    return &order, err
}

func (p *PrivateWebSocketClient) Close() error {
    if p.conn != nil {
        return p.conn.Close()
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
    "strconv"
)

func main() {
    // API 키 설정 (실제 키로 교체)
    accessKey := "your_access_key"
    secretKey := "your_secret_key"
    
    // Private WebSocket 클라이언트 생성
    client := NewPrivateWebSocketClient(accessKey, secretKey)
    
    // 연결
    if err := client.Connect(); err != nil {
        log.Fatal("WebSocket 연결 실패:", err)
    }
    defer client.Close()
    
    // 모든 마켓의 내 주문 정보 구독
    if err := client.SubscribeAllMyOrders(); err != nil {
        log.Fatal("구독 요청 실패:", err)
    }
    
    fmt.Println("내 주문 정보를 실시간으로 수신합니다...")
    
    // 메시지 수신 루프
    for {
        order, err := client.ReadMessage()
        if err != nil {
            log.Printf("메시지 수신 실패: %v", err)
            continue
        }
        
        // 주문 정보 출력
        createdTime, _ := order.GetCreatedTime()
        volume, _ := strconv.ParseFloat(order.Volume, 64)
        executedVolume, _ := strconv.ParseFloat(order.ExecutedVolume, 64)
        remainingVolume, _ := strconv.ParseFloat(order.RemainingVolume, 64)
        price, _ := strconv.ParseFloat(order.Price, 64)
        
        fmt.Printf("\n=== 주문 정보 업데이트 ===\n")
        fmt.Printf("마켓: %s\n", order.Code)
        fmt.Printf("UUID: %s\n", order.UUID)
        fmt.Printf("구분: %s\n", order.GetAskBidKorean())
        fmt.Printf("상태: %s\n", order.GetStateKorean())
        fmt.Printf("타입: %s\n", order.GetOrderTypeKorean())
        fmt.Printf("주문시간: %s\n", createdTime.Format("2006-01-02 15:04:05"))
        fmt.Printf("주문량: %.8f개\n", volume)
        fmt.Printf("체결량: %.8f개\n", executedVolume)
        fmt.Printf("잔여량: %.8f개\n", remainingVolume)
        if price > 0 {
            fmt.Printf("주문가격: %.0f원\n", price)
        }
        fmt.Printf("체결횟수: %d회\n", order.TradesCount)
    }
}
```

### 주문 상태 모니터링 예시
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type OrderMonitor struct {
    orders map[string]*MyOrderResponse
    mutex  sync.RWMutex
    client *PrivateWebSocketClient
}

func NewOrderMonitor(accessKey, secretKey string) *OrderMonitor {
    return &OrderMonitor{
        orders: make(map[string]*MyOrderResponse),
        client: NewPrivateWebSocketClient(accessKey, secretKey),
    }
}

func (om *OrderMonitor) Start() error {
    if err := om.client.Connect(); err != nil {
        return err
    }
    
    if err := om.client.SubscribeAllMyOrders(); err != nil {
        return err
    }
    
    go om.receiveMessages()
    return nil
}

func (om *OrderMonitor) receiveMessages() {
    for {
        order, err := om.client.ReadMessage()
        if err != nil {
            fmt.Printf("메시지 수신 오류: %v\n", err)
            time.Sleep(time.Second)
            continue
        }
        
        om.updateOrder(order)
    }
}

func (om *OrderMonitor) updateOrder(order *MyOrderResponse) {
    om.mutex.Lock()
    defer om.mutex.Unlock()
    
    previousOrder, exists := om.orders[order.UUID]
    om.orders[order.UUID] = order
    
    // 상태 변화 알림
    if exists && previousOrder.State != order.State {
        fmt.Printf("[상태 변화] %s: %s -> %s\n", 
            order.UUID[:8], 
            previousOrder.GetStateKorean(), 
            order.GetStateKorean(),
        )
    } else if !exists {
        fmt.Printf("[새 주문] %s: %s %s\n", 
            order.UUID[:8], 
            order.GetAskBidKorean(), 
            order.Code,
        )
    }
    
    // 체결 알림
    if exists {
        prevExecuted, _ := strconv.ParseFloat(previousOrder.ExecutedVolume, 64)
        currExecuted, _ := strconv.ParseFloat(order.ExecutedVolume, 64)
        
        if currExecuted > prevExecuted {
            executedAmount := currExecuted - prevExecuted
            fmt.Printf("[체결 알림] %s: %.8f개 체결\n", 
                order.UUID[:8], executedAmount,
            )
        }
    }
}

func (om *OrderMonitor) GetActiveOrders() []*MyOrderResponse {
    om.mutex.RLock()
    defer om.mutex.RUnlock()
    
    var activeOrders []*MyOrderResponse
    for _, order := range om.orders {
        if order.State == "wait" || order.State == "watch" {
            activeOrders = append(activeOrders, order)
        }
    }
    
    return activeOrders
}

func (om *OrderMonitor) Close() error {
    return om.client.Close()
}
```

## 주의사항

- **인증 필수**: Private 엔드포인트이므로 JWT 토큰 인증 필요
- **이벤트 기반**: 주문 변화 시에만 데이터 전송
- **초기 지연**: 연결 후 최초 데이터 수신까지 시간 소요 가능
- **토큰 관리**: JWT 토큰은 요청마다 새로 생성 권장
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리
- **문자열 숫자**: 정확도를 위해 숫자 필드가 문자열로 제공됨
- **UUID 고유성**: 주문 UUID는 전역적으로 고유함
- **상태 추적**: 주문 상태 변화를 추적하여 알림 구현 권장
- **에러 처리**: 네트워크 오류나 인증 실패에 대한 적절한 예외 처리 필요
- **보안**: API 키는 안전한 방법으로 관리해야 함