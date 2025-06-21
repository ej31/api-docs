# Upbit WebSocket API: 기본 정보

## 개요
Upbit의 WebSocket API는 실시간 암호화폐 데이터를 제공하는 웹소켓 서비스입니다.

## API 엔드포인트
- **Public 타입**: `wss://api.upbit.com/websocket/v1`
- **Private 타입**: `wss://api.upbit.com/websocket/v1/private`

## 데이터 타입

### Public 타입 (인증 불필요)
- `ticker`: 현재가 (스냅샷 및 실시간)
- `trade`: 체결 (스냅샷 및 실시간)
- `orderbook`: 호가 (스냅샷 및 실시간)
- `candle.1s`: 초봉 (스냅샷 및 실시간)

### Private 타입 (인증 필요)
- `myOrder`: 내 주문 (실시간)
- `myAsset`: 내 자산 (실시간)

## 주요 특징
- **스냅샷**: 요청 당시의 상태 정보
- **실시간**: 지속적으로 데이터 스트림 제공
- 요청 방법에 따라 수신 가능한 데이터 다름

## 시간대 설정
- **중요**: Upbit API의 시간대는 KST(한국 표준 시간) 기준으로 맞춰야 함
- 모든 시간 데이터는 KST 기준으로 처리

## 참고 사항
- 자세한 인증 방법은 WebSocket 요청 형식 페이지 참조
- Rate Limit 적용 (별도 페이지 참고)
- 연결 유지를 위해 주기적 ping/pong 권장