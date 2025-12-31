# 🏗️ AirAware - Kiến Trúc Hệ Thống Chi Tiết

## 📐 Sơ Đồ Kiến Trúc Tổng Quan

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                            │
├──────────────────────────────┬──────────────────────────────────┤
│      Web (Next.js)           │   Mobile (React Native/Expo)    │
│                              │                                  │
│  ┌────────────────────────┐  │  ┌──────────────────────────┐   │
│  │  Mapbox GL JS          │  │  │  React Native Maps      │   │
│  │  - Map rendering       │  │  │  - Map rendering        │   │
│  │  - Route polyline      │  │  │  - Route polyline       │   │
│  │  - AQI overlay         │  │  │  - AQI overlay          │   │
│  └────────────────────────┘  │  └──────────────────────────┘   │
│                              │                                  │
│  ┌────────────────────────┐  │  ┌──────────────────────────┐   │
│  │  WebSocket Client      │  │  │  WebSocket Client       │   │
│  │  - Real-time updates   │  │  │  - Real-time updates    │   │
│  └────────────────────────┘  │  └──────────────────────────┘   │
│                              │                                  │
│  ┌────────────────────────┐  │  ┌──────────────────────────┐   │
│  │  HTTP Client (Axios)   │  │  │  HTTP Client (Fetch)   │   │
│  │  - API calls           │  │  │  - API calls            │   │
│  └────────────────────────┘  │  └──────────────────────────┘   │
└──────────────┬───────────────┴──────────────┬───────────────────┘
               │                              │
               │  HTTPS / WSS                 │
               │                              │
┌──────────────▼──────────────────────────────▼───────────────────┐
│                    API GATEWAY (NestJS)                         │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Middleware Layer                            │   │
│  │  - CORS, Rate Limiting, Logging, Error Handling         │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┐ │
│  │ Map Module   │ Route Module │ AQI Module   │ WS Module    │ │
│  │              │              │              │              │ │
│  │ - Mapbox     │ - Directions │ - OpenAQ     │ - Socket.io  │ │
│  │   wrapper    │ - AQI calc   │ - IQAir      │ - Rooms      │ │
│  │ - Config     │ - Optimize   │ - Cache      │ - Broadcast  │ │
│  └──────┬───────┴──────┬───────┴──────┬───────┴──────┬───────┘ │
│         │              │              │              │         │
│  ┌──────▼──────────────▼──────────────▼──────────────▼───────┐ │
│  │              Service Layer (Business Logic)               │ │
│  │  - Route calculation with AQI                               │ │
│  │  - AQI aggregation & normalization                        │ │
│  │  - Route optimization algorithm                            │ │
│  └───────────────────────────────────────────────────────────┘ │
└──────────────┬──────────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────────┐
│                    DATA & EXTERNAL LAYER                        │
│                                                                  │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┐ │
│  │  PostgreSQL  │    Redis     │   Mapbox     │   OpenAQ     │ │
│  │  (Optional)  │   (Cache)    │    API       │    API       │ │
│  │              │              │              │              │ │
│  │  - Users     │  - AQI cache │  - Routing   │  - AQI data  │ │
│  │  - History   │  - Route     │  - Tiles     │  - Stations  │ │
│  │  - Favorites │    cache     │  - Geocoding │              │ │
│  └──────────────┴──────────────┴──────────────┴──────────────┘ │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Future: IQAir API (Production)              │  │
│  └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Luồng Dữ Liệu Chi Tiết

### 1. Luồng Tìm Đường Với AQI

```
User Input (Origin, Destination)
    ↓
[Web/Mobile] → POST /api/route/directions
    ↓
[API Gateway] → Route Controller
    ↓
[Route Service]
    ├─→ Mapbox Directions API → Get route geometry
    └─→ Parse route coordinates (sample every 100-200m)
    ↓
[Route Service] → AQI Service
    ├─→ For each coordinate:
    │   ├─→ Check Redis cache (key: "aqi:{lat}:{lng}")
    │   ├─→ If cache miss → Call OpenAQ API
    │   └─→ Store in Redis (TTL: 10 min)
    └─→ Aggregate AQI (avg, max, min)
    ↓
[Route Service] → Calculate route metrics
    ├─→ Total distance
    ├─→ Total duration
    ├─→ Average AQI
    ├─→ Max AQI segment
    └─→ AQI segments array
    ↓
[API Gateway] → Response JSON
    ↓
[Web/Mobile] → Render route + AQI overlay
```

### 2. Luồng Real-time AQI Updates

```
User starts navigation
    ↓
[Web/Mobile] → WebSocket: "subscribe:aqi" { lat, lng, radius }
    ↓
[WebSocket Gateway] → Join room (based on location grid)
    ↓
[Background Job] → Poll OpenAQ every 5 minutes
    ├─→ Fetch AQI for subscribed areas
    └─→ Compare with cached values
    ↓
[If AQI changed significantly]
    ├─→ Update Redis cache
    └─→ Broadcast to WebSocket room: "aqi:update"
    ↓
[Web/Mobile] → Receive update → Update map overlay
```

### 3. Luồng Route Optimization

```
User: "Avoid Pollution" mode ON
    ↓
[Web/Mobile] → POST /api/route/optimize { avoidPollution: true }
    ↓
[Route Service] → Get multiple route alternatives
    ├─→ Mapbox Directions API (alternatives: true)
    └─→ Get 2-3 alternative routes
    ↓
[Route Service] → For each alternative:
    ├─→ Calculate AQI (same as normal route)
    └─→ Calculate score: balance(duration, aqi)
    ↓
[Route Service] → Ranking algorithm
    ├─→ Score = (duration_weight * duration) + (aqi_weight * aqi)
    └─→ Sort by score
    ↓
[API Gateway] → Response with ranked routes
    ↓
[Web/Mobile] → Display route comparison UI
```

---

## 📦 Module Dependencies

### Backend Module Tree

```
AppModule (Root)
│
├─→ ConfigModule (Global)
│   └─→ Loads .env, provides ConfigService
│
├─→ MapModule
│   ├─→ MapService
│   └─→ MapController
│   └─→ Depends on: ConfigModule
│
├─→ AQIModule
│   ├─→ AQIService
│   ├─→ AQIController
│   └─→ Depends on: ConfigModule, CacheModule
│
├─→ RouteModule
│   ├─→ RouteService
│   ├─→ RouteOptimizerService
│   ├─→ RouteController
│   └─→ Depends on: MapModule, AQIModule, CacheModule
│
├─→ CacheModule (Global)
│   ├─→ CacheService
│   └─→ Redis client
│
├─→ WebSocketModule
│   ├─→ WebSocketGateway
│   ├─→ WebSocketService
│   └─→ Depends on: AQIModule
│
└─→ HealthModule
    └─→ HealthController
```

---

## 🔐 Security & Best Practices

### API Security
- ✅ CORS: Chỉ allow origins từ Web/Mobile apps
- ✅ Rate Limiting: Giới hạn số requests per IP
- ✅ Input Validation: DTOs với class-validator
- ✅ Error Handling: Không expose internal errors

### Environment Variables
```
# Backend (.env)
PORT=8000
NODE_ENV=development
MAPBOX_ACCESS_TOKEN=xxx
OPENAQ_API_KEY=xxx
REDIS_URL=redis://localhost:6379
CORS_ORIGINS=http://localhost:3000,http://localhost:19006

# Frontend (.env.local)
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_MAPBOX_TOKEN=xxx
```

### Error Handling Strategy
- **4xx Errors:** Client errors → return clear message
- **5xx Errors:** Server errors → log, return generic message
- **External API Errors:** Fallback, retry logic

---

## 📊 Data Models (Conceptual)

### Route Response
```typescript
interface RouteResponse {
  route: {
    geometry: Coordinate[];
    distance: number;      // meters
    duration: number;      // seconds
    steps: RouteStep[];
  };
  aqi: {
    average: number;
    max: number;
    min: number;
    segments: AQISegment[];
  };
  alternatives?: RouteResponse[];  // Phase 2
}
```

### AQI Segment
```typescript
interface AQISegment {
  coordinate: { lat: number; lng: number };
  aqi: number;
  pm25?: number;
  pm10?: number;
  timestamp: string;
}
```

### WebSocket Message
```typescript
interface AQIUpdateMessage {
  type: 'aqi:update';
  data: {
    location: { lat: number; lng: number };
    aqi: number;
    timestamp: string;
  };
}
```

---

## 🚀 Deployment Architecture

### Development
```
Local Machine
├─→ Backend: localhost:8000
├─→ Web: localhost:3000
└─→ Mobile: Expo Go / Simulator
```

### Production (Recommended)
```
┌─────────────────────────────────────────┐
│         CDN / Edge (Vercel)              │
│  - Web App (Next.js)                     │
│  - Static assets                         │
└─────────────────────────────────────────┘
              │
┌─────────────▼─────────────────────────────┐
│      API Server (Railway/Render/Vercel)   │
│  - NestJS Backend                        │
│  - WebSocket Server                      │
└─────────────┬─────────────────────────────┘
              │
┌─────────────▼─────────────────────────────┐
│      Data Layer                           │
│  - Redis (Upstash / Railway)              │
│  - PostgreSQL (Optional, Railway)         │
└───────────────────────────────────────────┘
```

### Mobile Deployment
- **Development:** Expo Go
- **Production:** Expo EAS Build → App Store / Play Store

---

## 📈 Performance Optimization

### Caching Strategy
1. **AQI Data:** Cache theo grid cell (1km x 1km)
   - Key: `aqi:grid:{gridId}`
   - TTL: 10 minutes
   
2. **Route Results:** Cache theo origin + destination
   - Key: `route:{origin}:{destination}:{profile}`
   - TTL: 5 minutes

3. **Mapbox Tiles:** CDN caching (automatic)

### API Call Optimization
- **Batch AQI requests:** Thay vì gọi từng điểm, gọi theo area
- **Route sampling:** Không lấy AQI cho mọi điểm, sample mỗi 100-200m
- **Parallel requests:** Dùng Promise.all() cho multiple AQI calls

### Frontend Optimization
- **Map rendering:** Chỉ render visible area
- **Route polyline:** Simplify geometry nếu quá nhiều points
- **WebSocket:** Throttle updates (max 1 update/second)

---

## 🔄 Future Enhancements

### Phase 4+ Ideas
- **Offline Mode:** Cache map tiles và AQI data
- **Predictive Routing:** ML model dự đoán AQI theo thời gian
- **Social Features:** Share routes, community AQI reports
- **Health Integration:** Connect với health apps
- **Multi-modal:** Combine driving + walking + public transport

---

**Tài liệu này sẽ được cập nhật khi dự án phát triển!**

