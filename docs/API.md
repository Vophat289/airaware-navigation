# 📡 AirAware API Documentation

## Base URL
```
Development: http://localhost:8000
Production: https://api.airaware.com
```

## Authentication
Hiện tại API không yêu cầu authentication (Phase 1). Sẽ thêm JWT authentication ở Phase 3.

---

## 🗺️ Map Module

### GET /api/map/config
Lấy Mapbox configuration cho client.

**Response:**
```json
{
  "mapboxToken": "pk.eyJ1Ijoi..."
}
```

---

## 🌬️ AQI Module

### GET /api/aqi/current
Lấy AQI tại một điểm cụ thể.

**Query Parameters:**
- `lat` (required): Latitude
- `lng` (required): Longitude

**Example:**
```
GET /api/aqi/current?lat=10.762622&lng=106.660172
```

**Response:**
```json
{
  "aqi": 85,
  "pm25": 35.2,
  "pm10": 45.8,
  "location": {
    "lat": 10.762622,
    "lng": 106.660172
  },
  "timestamp": "2024-01-15T10:30:00Z",
  "source": "openaq"
}
```

**Error Response:**
```json
{
  "statusCode": 404,
  "message": "AQI data not available for this location"
}
```

---

### GET /api/aqi/area
Lấy AQI trung bình trong một khu vực (bán kính).

**Query Parameters:**
- `lat` (required): Latitude
- `lng` (required): Longitude
- `radius` (optional): Radius in kilometers (default: 1)

**Example:**
```
GET /api/aqi/area?lat=10.762622&lng=106.660172&radius=2
```

**Response:**
```json
{
  "avgAqi": 78,
  "minAqi": 45,
  "maxAqi": 120,
  "stations": [
    {
      "lat": 10.762622,
      "lng": 106.660172,
      "aqi": 85
    }
  ],
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## 🛣️ Route Module

### POST /api/route/directions
Tìm đường từ điểm xuất phát đến điểm đích, kèm AQI data.

**Request Body:**
```json
{
  "origin": {
    "lat": 10.762622,
    "lng": 106.660172
  },
  "destination": {
    "lat": 10.7769,
    "lng": 106.7009
  },
  "profile": "driving",  // "driving" | "walking" | "cycling"
  "avoidPollution": false  // Phase 1: chỉ log, chưa thực sự avoid
}
```

**Response:**
```json
{
  "route": {
    "geometry": [
      { "lat": 10.762622, "lng": 106.660172 },
      { "lat": 10.763, "lng": 106.661 }
    ],
    "distance": 5234,  // meters
    "duration": 720,   // seconds
    "steps": [
      {
        "instruction": "Head north",
        "distance": 100,
        "duration": 15
      }
    ]
  },
  "aqi": {
    "average": 78,
    "max": 120,
    "min": 45,
    "segments": [
      {
        "coordinate": { "lat": 10.762622, "lng": 106.660172 },
        "aqi": 85,
        "pm25": 35.2
      }
    ]
  },
  "summary": {
    "totalDistance": 5234,
    "totalDuration": 720,
    "avgAqi": 78,
    "maxAqiSegment": {
      "lat": 10.765,
      "lng": 106.665,
      "aqi": 120
    }
  }
}
```

**Error Response:**
```json
{
  "statusCode": 400,
  "message": "Invalid coordinates"
}
```

---

### POST /api/route/optimize (Phase 2)
Tìm nhiều route alternatives và so sánh AQI.

**Request Body:**
```json
{
  "origin": {
    "lat": 10.762622,
    "lng": 106.660172
  },
  "destination": {
    "lat": 10.7769,
    "lng": 106.7009
  },
  "profile": "driving",
  "avoidPollution": true
}
```

**Response:**
```json
{
  "routes": [
    {
      "route": { /* same as /directions */ },
      "aqi": { /* same as /directions */ },
      "summary": { /* same as /directions */ },
      "recommendation": "cleanest",  // "best" | "fastest" | "cleanest"
      "score": 85.5
    },
    {
      "route": { /* ... */ },
      "aqi": { /* ... */ },
      "summary": { /* ... */ },
      "recommendation": "fastest",
      "score": 92.3
    }
  ],
  "bestRoute": {
    "index": 0,
    "reason": "Lowest AQI with acceptable duration"
  }
}
```

---

## 🔌 WebSocket Events (Phase 2)

### Connection
```
ws://localhost:8000
```

### Client → Server

#### Subscribe to AQI updates
```json
{
  "event": "subscribe:aqi",
  "data": {
    "lat": 10.762622,
    "lng": 106.660172,
    "radius": 2  // kilometers
  }
}
```

#### Unsubscribe
```json
{
  "event": "unsubscribe:aqi"
}
```

#### Location update (when user is moving)
```json
{
  "event": "location:update",
  "data": {
    "lat": 10.762622,
    "lng": 106.660172
  }
}
```

### Server → Client

#### AQI update
```json
{
  "event": "aqi:update",
  "data": {
    "location": {
      "lat": 10.762622,
      "lng": 106.660172
    },
    "aqi": 85,
    "pm25": 35.2,
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

---

## 🏥 Health & Config

### GET /api/health
Health check endpoint.

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2024-01-15T10:30:00Z",
  "uptime": 3600
}
```

---

### GET /api/config
Lấy public configuration.

**Response:**
```json
{
  "mapboxToken": "pk.eyJ1Ijoi...",
  "openaqUrl": "https://api.openaq.org"
}
```

---

## 📊 Response Status Codes

- `200` - Success
- `400` - Bad Request (invalid parameters)
- `404` - Not Found (AQI data not available)
- `500` - Internal Server Error
- `503` - Service Unavailable (external API down)

---

## 🔒 Rate Limiting

Phase 1: Không có rate limiting
Phase 2+: 
- 100 requests/minute per IP
- 10 WebSocket connections per IP

---

## 📝 Notes

- Tất cả coordinates phải là số thực (float)
- Latitude: -90 đến 90
- Longitude: -180 đến 180
- AQI values: 0-500 (US AQI scale)
- Timestamps: ISO 8601 format (UTC)

---

**API sẽ được cập nhật khi dự án phát triển!**

