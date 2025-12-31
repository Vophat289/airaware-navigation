# 🌬️ AirAware - Pollution-Aware Navigation System

> Hệ thống điều hướng thông minh tích hợp dữ liệu chất lượng không khí (AQI) để giúp người dùng tránh các khu vực ô nhiễm khi di chuyển.

## 🎯 Tổng Quan

AirAware là một ứng dụng điều hướng tương tự Google Maps nhưng có thêm lớp dữ liệu chất lượng không khí. Khi người dùng bật chế độ "Ưu tiên không khí sạch", hệ thống sẽ:

- ✅ Phát hiện các đoạn đường có AQI cao
- ✅ Đề xuất tuyến đường thay thế có AQI thấp hơn
- ✅ Hiển thị thời gian di chuyển real-time
- ✅ Cập nhật AQI theo thời gian thực khi đang di chuyển

## 🛠 Tech Stack

### Backend
- **Framework:** NestJS (Node.js, TypeScript)
- **Real-time:** WebSocket (Socket.io)
- **Cache:** Redis
- **Database:** PostgreSQL (optional, Phase 3)

### Frontend
- **Web:** Next.js 16, React 19, TypeScript
- **Mobile:** React Native (Expo), TypeScript

### Services & APIs
- **Maps:** Mapbox
- **AQI Data:** OpenAQ (MVP), IQAir (Production)
- **Routing:** Mapbox Directions API

## 📁 Cấu Trúc Dự Án

```
airaware-navigation/
├── airpath-api/          # NestJS Backend API
│   ├── src/
│   │   ├── map/         # Map Module (Mapbox integration)
│   │   ├── aqi/         # AQI Module (OpenAQ/IQAir)
│   │   ├── route/       # Route Module (Directions + AQI)
│   │   ├── websocket/   # WebSocket Module (Real-time updates)
│   │   └── cache/       # Cache Module (Redis)
│   └── .env.example
│
├── airpath-app/          # React Native Mobile App
│   ├── app/             # Expo Router (file-based routing)
│   ├── components/      # Reusable components
│   └── .env.example
│
├── airpath-web/          # Next.js Web App
│   ├── app/             # Next.js App Router
│   ├── components/      # React components
│   └── .env.example
│
├── ROADMAP.md           # Roadmap chi tiết theo phases
├── ARCHITECTURE.md      # Kiến trúc hệ thống
└── README.md            # File này
```

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ 
- npm hoặc yarn
- Redis (cho Phase 2+)
- Mapbox account (free tier OK)
- OpenAQ API key (free)

### Setup Backend

```bash
cd airpath-api
npm install

# Copy và điền thông tin
cp .env.example .env

# Chạy development server
npm run start:dev
```

Backend sẽ chạy tại: `http://localhost:8000`

### Setup Web App

```bash
cd airpath-web
npm install

# Copy và điền thông tin
cp .env.example .env.local

# Chạy development server
npm run dev
```

Web app sẽ chạy tại: `http://localhost:3000`

### Setup Mobile App

```bash
cd airpath-app
npm install

# Copy và điền thông tin
cp .env.example .env

# Chạy Expo
npm start
```

Scan QR code với Expo Go app hoặc chạy trên simulator.

## 📚 Documentation

- **[ROADMAP.md](./ROADMAP.md)** - Roadmap chi tiết theo 3 phases
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Kiến trúc hệ thống và luồng dữ liệu

## 🎯 Roadmap Overview

### Phase 1: MVP (2-3 tuần)
- ✅ Hiển thị bản đồ với AQI overlay
- ✅ Tìm đường cơ bản
- ✅ Hiển thị AQI cho route
- ✅ Web + Mobile support

### Phase 2: Nâng Cao (2-3 tuần)
- ✅ Real-time AQI updates (WebSocket)
- ✅ Route optimization (tìm route thay thế)
- ✅ Redis caching
- ✅ Performance optimization

### Phase 3: Thông Minh (3-4 tuần)
- ✅ AQI prediction
- ✅ User preferences
- ✅ Analytics dashboard
- ✅ Database integration

Xem chi tiết tại [ROADMAP.md](./ROADMAP.md)

## 🔧 Environment Variables

### Backend (`airpath-api/.env`)
```env
PORT=8000
NODE_ENV=development
MAPBOX_ACCESS_TOKEN=your_mapbox_token
OPENAQ_API_KEY=your_openaq_key
REDIS_URL=redis://localhost:6379
CORS_ORIGINS=http://localhost:3000,http://localhost:19006
```

### Web (`airpath-web/.env.local`)
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_MAPBOX_TOKEN=your_mapbox_token
```

### Mobile (`airpath-app/.env`)
```env
EXPO_PUBLIC_API_URL=http://localhost:8000
EXPO_PUBLIC_MAPBOX_TOKEN=your_mapbox_token
```

## 🧪 Testing

```bash
# Backend tests
cd airpath-api
npm run test

# E2E tests
npm run test:e2e
```

## 📦 Deployment

### Backend
- **Recommended:** Railway, Render, hoặc Vercel (serverless)
- **Redis:** Upstash (serverless Redis)

### Web
- **Recommended:** Vercel (automatic deployment từ GitHub)

### Mobile
- **Development:** Expo Go
- **Production:** Expo EAS Build → App Store / Play Store

## 🤝 Contributing

Đây là dự án học tập, bạn có thể:
- Fork và tự phát triển
- Submit issues nếu gặp bug
- Đóng góp improvements

## 📝 License

MIT License - Tự do sử dụng cho mục đích học tập và thương mại.

## 🙏 Acknowledgments

- **OpenAQ** - Cung cấp dữ liệu AQI miễn phí
- **Mapbox** - Maps và routing services
- **Expo** - React Native development platform
- **NestJS** - Progressive Node.js framework

## 📞 Contact & Support

Nếu có câu hỏi hoặc cần hỗ trợ, hãy tạo issue trên GitHub.

---

**Happy Coding! 🚀**

