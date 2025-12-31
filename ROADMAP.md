# 🗺️ AirAware - Pollution-Aware Navigation System
## Roadmap Tổng Thể & Kiến Trúc Hệ Thống

---

## 📋 Mục Lục
1. [Tổng Quan Dự Án](#tổng-quan-dự-án)
2. [Kiến Trúc Hệ Thống](#kiến-trúc-hệ-thống)
3. [Phase 1: MVP (Minimum Viable Product)](#phase-1-mvp-minimum-viable-product)
4. [Phase 2: Nâng Cao & Tối Ưu](#phase-2-nâng-cao--tối-ưu)
5. [Phase 3: Thông Minh & Cá Nhân Hóa](#phase-3-thông-minh--cá-nhân-hóa)
6. [Checklist Tổng Thể](#checklist-tổng-thể)
7. [Định Hướng Scale](#định-hướng-scale)
8. [Trình Bày Dự Án](#trình-bày-dự-án)

---

## 🎯 Tổng Quan Dự Án

### Mục Tiêu
Xây dựng hệ thống điều hướng tích hợp dữ liệu chất lượng không khí (AQI) để giúp người dùng tránh các khu vực ô nhiễm khi di chuyển.

### Core Features
- ✅ Hiển thị bản đồ với overlay AQI theo khu vực
- ✅ Tìm đường với ưu tiên không khí sạch
- ✅ Real-time AQI updates
- ✅ So sánh tuyến đường (thời gian vs chất lượng không khí)
- ✅ Hỗ trợ Web + Mobile

---

## 🏗️ Kiến Trúc Hệ Thống

### Sơ Đồ Tổng Quan (Text-based)

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
├──────────────────────┬───────────────────────────────────────┤
│   Web (Next.js)      │   Mobile (React Native/Expo)         │
│   - Mapbox GL JS     │   - React Native Maps                │
│   - WebSocket Client │   - WebSocket Client                 │
└──────────┬───────────┴──────────────┬────────────────────────┘
           │                          │
           │  HTTP/REST + WebSocket   │
           │                          │
┌──────────▼──────────────────────────▼────────────────────────┐
│                    API GATEWAY (NestJS)                      │
│  ┌──────────────┬──────────────┬──────────────┬──────────┐  │
│  │  Map Module  │  Route Module│  AQI Module  │  WS Module│ │
│  └──────┬───────┴──────┬───────┴──────┬───────┴──────┬────┘  │
│         │              │              │              │       │
│  ┌──────▼──────────────▼──────────────▼──────────────▼────┐ │
│  │              Service Layer (Business Logic)             │ │
│  └────────────────────────────────────────────────────────┘ │
└──────────┬──────────────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────────────┐
│                    DATA LAYER                                │
│  ┌──────────────┬──────────────┬──────────────┐            │
│  │   PostgreSQL │    Redis     │  External    │            │
│  │   (Optional) │   (Cache)    │  APIs        │            │
│  └──────────────┴──────────────┴──────────────┘            │
│                                                              │
│  External APIs:                                             │
│  - OpenAQ (AQI Data)                                         │
│  - Mapbox (Routing, Tiles)                                  │
│  - IQAir (Production AQI)                                    │
└──────────────────────────────────────────────────────────────┘
```

### Luồng Dữ Liệu Chính

1. **User Request Route** → API Gateway → Route Service → Mapbox API → Tính toán AQI → Trả về route với AQI
2. **Real-time AQI Update** → WebSocket → Broadcast đến clients đang active
3. **Cache Strategy** → Redis cache AQI data theo khu vực (TTL: 5-15 phút)

---

## 🚀 Phase 1: MVP (Minimum Viable Product)

### 🎯 Mục Tiêu Phase 1
- ✅ Có thể demo được: hiển thị bản đồ + AQI overlay
- ✅ Tìm đường cơ bản với Mapbox
- ✅ Hiển thị AQI cho điểm xuất phát/đích
- ✅ Chạy được trên Web và Mobile
- ⏱️ **Timeline dự kiến: 2-3 tuần**

### 📦 Backend Modules (NestJS)

#### **Module 1: Map Module** (Ưu tiên đầu tiên)
**Mục đích:** Quản lý tương tác với Mapbox API

**Files cần tạo:**
- `src/map/map.module.ts`
- `src/map/map.service.ts`
- `src/map/map.controller.ts`
- `src/map/dto/map-request.dto.ts`

**Chức năng:**
- Lấy Mapbox access token từ config
- Validate coordinates
- Wrapper cho Mapbox API calls

**APIs:**
```
GET /api/map/config
  → Trả về Mapbox token (public, không cần auth)
  → Response: { mapboxToken: string }
```

**Thứ tự code:**
1. Tạo module structure
2. Setup Mapbox service với axios
3. Tạo controller trả về config
4. Test với Postman/Thunder Client

---

#### **Module 2: AQI Module** (Ưu tiên thứ 2)
**Mục đích:** Lấy và xử lý dữ liệu AQI từ OpenAQ

**Files cần tạo:**
- `src/aqi/aqi.module.ts`
- `src/aqi/aqi.service.ts`
- `src/aqi/aqi.controller.ts`
- `src/aqi/dto/aqi-request.dto.ts`
- `src/aqi/interfaces/openaq.interface.ts`

**Chức năng:**
- Fetch AQI từ OpenAQ API
- Parse và normalize AQI data
- Tính AQI trung bình cho một khu vực (lat/lng + radius)

**APIs:**
```
GET /api/aqi/current?lat={lat}&lng={lng}
  → Lấy AQI tại một điểm
  → Response: { aqi: number, pm25: number, pm10: number, location: {...} }

GET /api/aqi/area?lat={lat}&lng={lng}&radius={km}
  → Lấy AQI trung bình trong bán kính
  → Response: { avgAqi: number, stations: [...] }
```

**Thứ tự code:**
1. Setup OpenAQ service (axios client)
2. Implement fetch AQI by coordinates
3. Implement calculate area AQI
4. Tạo controller endpoints
5. Test với Postman

**⚠️ Lưu ý:**
- OpenAQ free tier có rate limit → cần cache (sẽ làm ở Phase 2)
- AQI data có thể không có ở mọi nơi → handle null/empty gracefully

---

#### **Module 3: Route Module** (Ưu tiên thứ 3)
**Mục đích:** Tìm đường và tích hợp AQI vào route

**Files cần tạo:**
- `src/route/route.module.ts`
- `src/route/route.service.ts`
- `src/route/route.controller.ts`
- `src/route/dto/route-request.dto.ts`
- `src/route/interfaces/route.interface.ts`

**Chức năng:**
- Gọi Mapbox Directions API
- Lấy route geometry (danh sách coordinates)
- Tính AQI cho từng segment của route
- Trả về route + AQI data

**APIs:**
```
POST /api/route/directions
  Body: {
    origin: { lat, lng },
    destination: { lat, lng },
    profile: "driving" | "walking" | "cycling",
    avoidPollution?: boolean  // Phase 1: chỉ log, chưa thực sự avoid
  }
  → Response: {
    route: { geometry, distance, duration },
    aqiSegments: [{ lat, lng, aqi }],
    avgAqi: number,
    maxAqi: number
  }
```

**Thứ tự code:**
1. Setup Mapbox Directions service
2. Implement get route từ origin → destination
3. Parse route geometry (coordinates)
4. Tích hợp AQI Module: lấy AQI cho từng điểm trên route
5. Tính toán avg/max AQI
6. Tạo controller endpoint
7. Test với Postman

**⚠️ Lưu ý:**
- Phase 1: Chỉ tính AQI cho route, chưa tìm route thay thế
- Route geometry có thể có nhiều điểm → sample mỗi 100-200m để giảm API calls

---

#### **Module 4: Health & Config Module** (Ưu tiên thứ 4)
**Mục đích:** Health check, config management

**Files cần tạo:**
- `src/config/config.module.ts`
- `src/config/config.service.ts`
- `src/health/health.controller.ts`

**APIs:**
```
GET /api/health
  → { status: "ok", timestamp: "..." }

GET /api/config
  → { mapboxToken: "...", openaqUrl: "..." }
```

---

### 🎨 Frontend/Mobile Features

#### **Web (Next.js)**

**Pages cần tạo:**
1. `app/page.tsx` - Trang chính với bản đồ
2. `app/components/Map.tsx` - Mapbox component
3. `app/components/RouteSearch.tsx` - Form tìm đường
4. `app/components/AQILegend.tsx` - Legend hiển thị AQI colors

**Features:**
- ✅ Hiển thị Mapbox map
- ✅ Search box: nhập địa chỉ xuất phát/đích
- ✅ Hiển thị route trên map (polyline)
- ✅ Overlay AQI colors trên route (gradient theo AQI)
- ✅ Hiển thị AQI số tại điểm xuất phát/đích
- ✅ Sidebar: thông tin route (distance, duration, avg AQI)

**Thứ tự code:**
1. Setup Mapbox GL JS trong Next.js
2. Tạo Map component cơ bản
3. Tích hợp RouteSearch form
4. Call API `/api/route/directions`
5. Render route polyline
6. Thêm AQI overlay (heatmap hoặc gradient)
7. Styling UI

---

#### **Mobile (React Native/Expo)**

**Screens cần tạo:**
1. `app/(tabs)/index.tsx` - Map screen
2. `app/(tabs)/explore.tsx` - Search screen (tạm thời)
3. `app/components/MapView.tsx` - React Native Maps component
4. `app/components/RouteInfo.tsx` - Bottom sheet hiển thị route info

**Features:**
- ✅ Hiển thị map với react-native-maps hoặc Mapbox SDK
- ✅ Search bar ở top
- ✅ Hiển thị route với AQI colors
- ✅ Bottom sheet: route details
- ✅ Floating button: toggle "Avoid Pollution" mode

**Thứ tự code:**
1. Setup react-native-maps hoặc @rnmapbox/maps
2. Tạo MapView component
3. Tích hợp search
4. Call API và render route
5. Thêm AQI visualization
6. Styling

---

### ❌ Những Gì CHƯA Nên Làm Ở Phase 1

- ❌ Authentication/Authorization (chưa cần)
- ❌ User accounts, profiles
- ❌ Real-time WebSocket (dùng polling tạm thời)
- ❌ Redis cache (dùng in-memory cache đơn giản)
- ❌ Database (chưa cần lưu routes, users)
- ❌ Route optimization với AQI (chỉ hiển thị, chưa tìm route thay thế)
- ❌ History, favorites
- ❌ Push notifications

**Lý do:** Tập trung vào core feature trước, tránh over-engineering.

---

## 🚀 Phase 2: Nâng Cao & Tối Ưu

### 🎯 Mục Tiêu Phase 2
- ✅ Real-time AQI updates qua WebSocket
- ✅ Route optimization: tìm route thay thế có AQI thấp hơn
- ✅ Redis cache để giảm API calls
- ✅ Tối ưu hiệu năng
- ⏱️ **Timeline dự kiến: 2-3 tuần**

### 📦 Backend Modules Mới

#### **Module 5: WebSocket Module**
**Mục đích:** Real-time AQI updates

**Files cần tạo:**
- `src/websocket/websocket.module.ts`
- `src/websocket/websocket.gateway.ts`
- `src/websocket/websocket.service.ts`

**Chức năng:**
- Khi user đang di chuyển, gửi location updates
- Server broadcast AQI updates cho khu vực đó
- Room-based: group users theo khu vực

**Events:**
```
Client → Server:
  - "subscribe:aqi" { lat, lng, radius }
  - "location:update" { lat, lng }

Server → Client:
  - "aqi:update" { lat, lng, aqi, timestamp }
```

**Thứ tự code:**
1. Setup @nestjs/websocket
2. Tạo WebSocket gateway
3. Implement subscribe/unsubscribe logic
4. Tích hợp với AQI service để push updates
5. Test với WebSocket client

---

#### **Module 6: Cache Module (Redis)**
**Mục đích:** Cache AQI data để giảm API calls

**Files cần tạo:**
- `src/cache/cache.module.ts`
- `src/cache/cache.service.ts`

**Chức năng:**
- Cache AQI theo grid cell (ví dụ: 1km x 1km)
- TTL: 5-15 phút
- Cache route results

**Thứ tự code:**
1. Setup Redis client (ioredis)
2. Tạo cache service với get/set/delete
3. Tích hợp vào AQI service
4. Tích hợp vào Route service
5. Test cache hit/miss

---

#### **Module 7: Route Optimization**
**Mục đích:** Tìm route thay thế có AQI thấp hơn

**Files cần tạo:**
- `src/route/route-optimizer.service.ts` (thêm vào Route module)

**Chức năng:**
- Tìm 2-3 route alternatives từ Mapbox
- So sánh AQI của các routes
- Recommend route có AQI thấp nhất (trade-off với thời gian)

**APIs:**
```
POST /api/route/optimize
  Body: { origin, destination, avoidPollution: true }
  → Response: {
    routes: [
      { route, aqi, duration, recommendation: "best" | "fastest" | "cleanest" }
    ]
  }
```

**Thứ tự code:**
1. Gọi Mapbox với alternatives=true
2. Tính AQI cho mỗi alternative
3. Ranking algorithm: balance giữa duration và AQI
4. Trả về recommendations

---

### 🎨 Frontend/Mobile Updates

**Web:**
- ✅ WebSocket client connection
- ✅ Real-time AQI updates trên map
- ✅ Route comparison UI (hiển thị 2-3 routes, so sánh)
- ✅ Toggle "Avoid Pollution" mode

**Mobile:**
- ✅ WebSocket connection
- ✅ Background location tracking (khi đang di chuyển)
- ✅ Route alternatives carousel
- ✅ Notification khi AQI thay đổi đáng kể

---

### ❌ Những Gì CHƯA Nên Làm Ở Phase 2

- ❌ Machine Learning cho dự đoán AQI
- ❌ User preferences, personalization
- ❌ Database cho lưu lịch sử
- ❌ Advanced analytics

---

## 🚀 Phase 3: Thông Minh & Cá Nhân Hóa

### 🎯 Mục Tiêu Phase 3
- ✅ Dự đoán AQI (ML model hoặc rule-based)
- ✅ Cá nhân hóa: user preferences, health conditions
- ✅ Database: lưu lịch sử, favorites
- ✅ Analytics dashboard
- ⏱️ **Timeline dự kiến: 3-4 tuần**

### 📦 Backend Modules Mới

#### **Module 8: Prediction Module**
**Mục đích:** Dự đoán AQI trong tương lai (1-3 giờ)

**Approach:**
- Option 1: Rule-based (đơn giản hơn)
  - Dựa trên historical data, time of day, weather
- Option 2: ML Model (phức tạp hơn)
  - Train model với historical AQI data
  - Features: time, weather, traffic, location

**APIs:**
```
GET /api/aqi/predict?lat={lat}&lng={lng}&hours={1-3}
  → Response: { currentAqi, predictedAqi, confidence }
```

---

#### **Module 9: User Module** (nếu cần auth)
**Mục đích:** User accounts, preferences

**Features:**
- Health conditions (asthma, allergies)
- Preferred AQI threshold
- Route history
- Favorites

---

#### **Module 10: Analytics Module**
**Mục đích:** Thống kê, insights

**Features:**
- AQI trends theo khu vực
- Most polluted routes
- Best time to travel

---

### 🎨 Frontend/Mobile Updates

- ✅ Prediction timeline UI
- ✅ User settings/preferences
- ✅ Route history
- ✅ Analytics charts

---

## ✅ Checklist Tổng Thể

### Phase 1: MVP
- [ ] Backend: Map Module
- [ ] Backend: AQI Module
- [ ] Backend: Route Module
- [ ] Backend: Health/Config Module
- [ ] Web: Map display
- [ ] Web: Route search & display
- [ ] Web: AQI visualization
- [ ] Mobile: Map display
- [ ] Mobile: Route search & display
- [ ] Mobile: AQI visualization
- [ ] Testing: API endpoints
- [ ] Testing: Web app
- [ ] Testing: Mobile app
- [ ] Deployment: Backend (Vercel/Railway/Render)
- [ ] Deployment: Web (Vercel)
- [ ] Deployment: Mobile (Expo EAS)

### Phase 2: Nâng Cao
- [ ] Backend: WebSocket Module
- [ ] Backend: Redis Cache Module
- [ ] Backend: Route Optimization
- [ ] Web: WebSocket integration
- [ ] Web: Route comparison UI
- [ ] Mobile: WebSocket integration
- [ ] Mobile: Background location tracking
- [ ] Performance: Load testing
- [ ] Performance: Cache optimization

### Phase 3: Thông Minh
- [ ] Backend: Prediction Module
- [ ] Backend: User Module (nếu cần)
- [ ] Backend: Analytics Module
- [ ] Frontend: Prediction UI
- [ ] Frontend: User preferences
- [ ] Frontend: Analytics dashboard
- [ ] Database: Setup PostgreSQL/MongoDB
- [ ] Database: Migrations

---

## 📈 Định Hướng Scale

### Horizontal Scaling
- **API Servers:** Stateless → dễ scale với load balancer
- **WebSocket:** Dùng Redis Pub/Sub để sync giữa multiple servers
- **Cache:** Redis cluster cho high availability

### Database Scaling
- **Read Replicas:** Tách read/write
- **Sharding:** Nếu cần, shard theo location (geo-based)

### CDN & Caching
- **Static Assets:** Vercel CDN (Web), Expo CDN (Mobile)
- **API Response:** Cache AQI data ở edge (Cloudflare Workers)

### Monitoring & Observability
- **APM:** Sentry, Datadog
- **Logs:** Winston + ELK stack hoặc CloudWatch
- **Metrics:** Prometheus + Grafana

---

## 📝 Trình Bày Dự Án

### GitHub Repository Structure
```
airaware-navigation/
├── airpath-api/          # NestJS Backend
├── airpath-app/          # React Native Mobile
├── airpath-web/          # Next.js Web
├── README.md             # Tổng quan dự án
├── ROADMAP.md            # File này
├── ARCHITECTURE.md        # Chi tiết kiến trúc (optional)
└── docs/                 # Documentation
    ├── api.md            # API documentation
    └── setup.md          # Setup guide
```

### README.md Template
```markdown
# AirAware - Pollution-Aware Navigation System

## 🎯 Mô Tả
Hệ thống điều hướng tích hợp dữ liệu chất lượng không khí (AQI)...

## 🛠 Tech Stack
- Backend: NestJS, TypeScript
- Mobile: React Native (Expo)
- Web: Next.js
- Maps: Mapbox
- AQI Data: OpenAQ, IQAir
- Realtime: WebSocket
- Cache: Redis

## 🚀 Features
- [x] Real-time AQI overlay trên bản đồ
- [x] Route optimization với ưu tiên không khí sạch
- [x] Web + Mobile support
- [x] Real-time updates qua WebSocket

## 📦 Setup
[Instructions...]

## 📚 Documentation
- [API Docs](./docs/api.md)
- [Roadmap](./ROADMAP.md)
```

### CV/Portfolio Description
```
AirAware - Pollution-Aware Navigation System
• Full-stack navigation app với tích hợp AQI data
• Tech: NestJS, React Native, Next.js, Mapbox, WebSocket, Redis
• Features: Real-time AQI overlay, route optimization, WebSocket updates
• Architecture: Microservices-ready, Redis caching, horizontal scaling
• Timeline: [X] weeks, [Y] phases
```

### Demo Video Script
1. **Intro (10s):** "AirAware helps you avoid polluted areas..."
2. **Web Demo (30s):** Show map, search route, AQI overlay
3. **Mobile Demo (30s):** Show mobile app, real-time updates
4. **Route Optimization (20s):** Compare routes with/without pollution avoidance
5. **Tech Stack (10s):** Quick overview

---

## 🎓 Tips Cho Người Mới Học

### Thứ Tự Học & Code
1. **Bắt đầu với Backend:**
   - Học NestJS basics (modules, services, controllers)
   - Học cách gọi external APIs (axios)
   - Học cách structure code (DTOs, interfaces)

2. **Sau đó Frontend:**
   - Học Mapbox GL JS (Web) hoặc React Native Maps
   - Học cách render data trên map
   - Học WebSocket client

3. **Tối Ưu Dần:**
   - Bắt đầu không cache → thêm cache sau
   - Bắt đầu polling → chuyển WebSocket sau
   - Bắt đầu simple → optimize sau

### Common Pitfalls
- ❌ Đừng code tất cả modules cùng lúc → làm từng module, test xong mới chuyển
- ❌ Đừng optimize sớm → làm cho nó chạy được trước
- ❌ Đừng bỏ qua error handling → handle null/empty data
- ❌ Đừng hardcode API keys → dùng env variables

### Resources Hữu Ích
- NestJS Docs: https://docs.nestjs.com
- Mapbox Docs: https://docs.mapbox.com
- OpenAQ API: https://docs.openaq.org
- Expo Router: https://docs.expo.dev/router/introduction

---

## 📞 Next Steps

1. **Review roadmap này** → đảm bảo hiểu rõ từng phase
2. **Setup môi trường** → install dependencies, config env
3. **Bắt đầu Phase 1** → code từng module theo thứ tự
4. **Test thường xuyên** → không code quá nhiều mới test
5. **Document code** → comment, README, commit messages rõ ràng

**Chúc bạn code vui vẻ! 🚀**

