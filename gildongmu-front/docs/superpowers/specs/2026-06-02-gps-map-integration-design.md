# GPS + 지도 연동 설계

**날짜:** 2026-06-02  
**프로젝트:** gildongmu-front  
**상태:** 확정

---

## 개요

백엔드(Mapbox Directions API)가 계산한 `points[]` 배열을 프론트에서 `react-native-maps`(Google Maps)로 표시한다.  
GPS 실시간 추적으로 러닝 중 현재 위치·페이스를 보여주고, 백엔드에 러닝 데이터를 저장한다.

---

## 기술 선택

| 역할 | 라이브러리 | 이유 |
|------|-----------|------|
| 지도 표시 | `react-native-maps` | Expo Go 호환, Google Maps 한국 표시 정상 동작 |
| GPS 추적 | `expo-location` | 이미 설치됨 |
| 경로 계산 | Mapbox Directions API (백엔드) | 한국 경로 계산 지원 |

> Mapbox는 경로 계산(백엔드)에만 사용. 지도 렌더링은 react-native-maps.

---

## 아키텍처

```
course/detail.tsx
  └─ RouteMapView.tsx (신규)
       └─ MapView + Polyline: 전체 루트 미리보기 (정적)
       └─ Marker: 시작점, 도착점

running/index.tsx
  └─ RunningMapView.tsx (신규)
       └─ MapView + Polyline: 지나온 구간(파랑) / 남은 구간(회색)
       └─ Marker: 현재 위치 (실시간 이동)
  └─ useRunningGPS.ts (신규 훅)
       └─ expo-location watchPositionAsync (3초 주기)
       └─ /api/runs/start → /api/runs/update → /api/runs/end
       └─ 구간 페이스 계산 (실시간 + 종료 후 분석용)
```

---

## 신규 파일

| 파일 | 역할 |
|------|------|
| `components/RouteMapView.tsx` | 정적 루트 미리보기 지도 |
| `components/RunningMapView.tsx` | 실시간 네비게이션 지도 |
| `hooks/useRunningGPS.ts` | GPS 추적 + API 전송 + 페이스 계산 |

### 기존 파일 수정

| 파일 | 변경 내용 |
|------|----------|
| `components/MapPlaceholder.tsx` | 삭제 (RouteMapView/RunningMapView로 대체) |
| `app/(tabs)/course/detail.tsx` | MapPlaceholder → RouteMapView |
| `app/running/index.tsx` | MapPlaceholder → RunningMapView, useRunningGPS 연결 |
| `utils/api.ts` | runs API 함수 추가 (start/update/end/history) |

---

## GPS 훅 설계 (useRunningGPS)

```typescript
// 반환값
{
  currentLocation: { latitude, longitude } | null
  elapsedSeconds: number
  distanceMeters: number
  currentPaceSecPerKm: number       // 실시간 페이스
  trackedPoints: RoutePoint[]       // 지나온 경로
  lapPaces: { km: number, paceSecPerKm: number }[]  // km 구간별 페이스
  start: (runId: number, route: RoutePoint[]) => void
  stop: () => Promise<void>
}
```

**페이스 계산:** 직전 포인트와의 거리(m) / 시간(s) → m/s → sec/km 변환  
**구간 페이스:** 누적 거리 1km 달성 시 평균 페이스 저장

---

## API 연동 (utils/api.ts 추가)

| 함수 | 엔드포인트 | 타이밍 |
|------|-----------|--------|
| `startRun(routeId)` | POST /api/runs/start | 달리기 시작 버튼 |
| `updateRun(runId, lat, lng, pace)` | POST /api/runs/update | 3초마다 |
| `endRun(runId, points, lapPaces)` | POST /api/runs/end | 달리기 종료 |
| `getRunHistory()` | GET /api/runs/history | 기록 탭 진입 |

---

## 지도 표시 상세

### RouteMapView (코스 상세 미리보기)
- 전체 루트 폴리라인 (파란색)
- 시작점 마커 (초록), 도착점 마커 (빨강)
- 전체 경로가 보이도록 카메라 자동 피팅

### RunningMapView (러닝 실행)
- 남은 경로: 회색 폴리라인
- 지나온 경로: 파란색 폴리라인
- 현재 위치: 파란 점 마커 (실시간 이동)
- 카메라: 현재 위치 추적 (followUser)

---

## 테스트 방법

- Expo Go (iPhone) 그대로 사용
- `expo start` → QR 스캔으로 즉시 테스트
- GPS: 실제 기기 이동 or 에뮬레이터 좌표 시뮬레이션
