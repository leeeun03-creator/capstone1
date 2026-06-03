# GPS + Mapbox 吏???곕룞 援ы쁽 怨꾪쉷

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** WebView + Mapbox GL JS濡?肄붿뒪 誘몃━蹂닿린쨌?щ떇 ?ㅼ떆媛?吏?꾨? 援ы쁽?섍퀬, expo-location + 諛깆뿏??Runs API濡?GPS 異붿쟻/?섏씠??怨꾩궛???곕룞?쒕떎.

**Architecture:** react-native-webview(Expo Go ?명솚)??Mapbox GL JS HTML??濡쒕뱶??吏?꾨? ?뚮뜑留곹븳?? GPS??expo-location??3珥덈쭏???섏쭛?섍퀬, useRunningGPS ?낆씠 ?섏씠??怨꾩궛 + 諛깆뿏???꾩넚???대떦?쒕떎. 肄붿뒪 寃쎈줈 points[]??course/index ??detail ??running ?쒖쑝濡?JSON 臾몄옄???뚮엺?쇰줈 ?꾨떖?쒕떎.

**Tech Stack:** react-native-webview, Mapbox GL JS v3.3.0(CDN), expo-location, Spring Boot Runs API

---

## ?뚯씪 援ъ“

| ?곹깭 | ?뚯씪 | ??븷 |
|------|------|------|
| ?좉퇋 | `utils/geo.ts` | Haversine 嫄곕━ 怨꾩궛, ?섏씠??怨꾩궛 (?쒖닔 ?⑥닔) |
| ?좉퇋 | `utils/mapHtml.ts` | ?뺤쟻/?щ떇 Mapbox GL JS HTML ?앹꽦 ?⑥닔 |
| ?좉퇋 | `components/RouteMapView.tsx` | ?뺤쟻 猷⑦듃 誘몃━蹂닿린 WebView |
| ?좉퇋 | `components/RunningMapView.tsx` | ?ㅼ떆媛??ㅻ퉬寃뚯씠??WebView |
| ?좉퇋 | `hooks/useRunningGPS.ts` | GPS 異붿쟻 + API ?꾩넚 + ?섏씠??怨꾩궛 ??|
| ?섏젙 | `utils/api.ts` | startRun / updateRun / endRun / getHistory 異붽? |
| ?섏젙 | `app/(tabs)/course/index.tsx` | points[] JSON ?뚮엺 異붽? |
| ?섏젙 | `app/(tabs)/course/detail.tsx` | RouteMapView ?곌껐, points ?뚮엺 ?뚯떛 |
| ?섏젙 | `app/running/index.tsx` | RunningMapView + useRunningGPS濡?援먯껜 |
| ?섏젙 | `.env` (?좉퇋) | EXPO_PUBLIC_MAPBOX_TOKEN 異붽? |
| ??젣 | `components/MapPlaceholder.tsx` | RouteMapView/RunningMapView濡??泥?|

---

## Task 1: react-native-webview ?ㅼ튂 + .env ?ㅼ젙

**Files:**
- Create: `.env`
- Modify: `app.json`

- [ ] **Step 1: react-native-webview ?ㅼ튂**

```bash
npx expo install react-native-webview
```

Expected: `package.json`??`react-native-webview` 異붽???
- [ ] **Step 2: .env ?뚯씪 ?앹꽦**

`.env` ?뚯씪???꾨줈?앺듃 猷⑦듃???앹꽦:

```
EXPO_PUBLIC_MAPBOX_TOKEN=your_mapbox_token_here
```

> ?좏겙? `gildongmu-back/.env`??`VITE_MAPBOX_TOKEN` 媛믨낵 ?숈씪

- [ ] **Step 3: .env瑜?.gitignore??異붽? (?놁쑝硫?**

```bash
echo ".env" >> .gitignore
```

- [ ] **Step 4: Expo媛 env瑜??쎈뒗吏 ?뺤씤 (app.json ?섏젙 遺덊븘????Expo SDK 49+??EXPO_PUBLIC_ ?먮룞 吏??**

`app.json`??蹂꾨룄 ?ㅼ젙 ?놁뼱???? `process.env.EXPO_PUBLIC_MAPBOX_TOKEN`?쇰줈 ?묎렐 媛??

- [ ] **Step 5: 而ㅻ컠**

```bash
git add package.json package-lock.json .gitignore
git commit -m "feat: install react-native-webview"
```

---

## Task 2: Geo ?좏떥由ы떚 + ?⑥쐞 ?뚯뒪??
**Files:**
- Create: `utils/geo.ts`
- Create: `__tests__/utils/geo.test.ts`

- [ ] **Step 1: ?ㅽ뙣?섎뒗 ?뚯뒪???묒꽦**

`__tests__/utils/geo.test.ts`:

```typescript
import { haversineMeters, paceSecPerKm } from '../../utils/geo'

describe('haversineMeters', () => {
  it('媛숈? 醫뚰몴??0 諛섑솚', () => {
    expect(haversineMeters(
      { latitude: 33.4507, longitude: 126.5707 },
      { latitude: 33.4507, longitude: 126.5707 },
    )).toBe(0)
  })

  it('?꾨룄 1??李⑥씠????111km', () => {
    const d = haversineMeters(
      { latitude: 0, longitude: 0 },
      { latitude: 1, longitude: 0 },
    )
    expect(d).toBeGreaterThan(110000)
    expect(d).toBeLessThan(112000)
  })

  it('?쒖＜ 怨듯빆 ??以묐Ц ??20km', () => {
    const d = haversineMeters(
      { latitude: 33.5070, longitude: 126.4930 }, // ?쒖＜怨듯빆
      { latitude: 33.2440, longitude: 126.4120 }, // 以묐Ц
    )
    expect(d).toBeGreaterThan(28000)
    expect(d).toBeLessThan(32000)
  })
})

describe('paceSecPerKm', () => {
  it('嫄곕━ 0?대㈃ 0 諛섑솚', () => {
    expect(paceSecPerKm(0, 100)).toBe(0)
  })

  it('1000m 360珥???360 sec/km (6遺?km)', () => {
    expect(paceSecPerKm(1000, 360)).toBe(360)
  })

  it('500m 180珥???360 sec/km', () => {
    expect(paceSecPerKm(500, 180)).toBe(360)
  })
})
```

- [ ] **Step 2: ?뚯뒪???ㅽ뻾 ???ㅽ뙣 ?뺤씤**

```bash
npx jest __tests__/utils/geo.test.ts
```

Expected: FAIL (geo.ts ?놁쓬)

- [ ] **Step 3: geo.ts 援ы쁽**

`utils/geo.ts`:

```typescript
export function haversineMeters(
  a: { latitude: number; longitude: number },
  b: { latitude: number; longitude: number },
): number {
  const R = 6371000
  const lat1 = (a.latitude * Math.PI) / 180
  const lat2 = (b.latitude * Math.PI) / 180
  const dLat = ((b.latitude - a.latitude) * Math.PI) / 180
  const dLon = ((b.longitude - a.longitude) * Math.PI) / 180
  const sinDLat = Math.sin(dLat / 2)
  const sinDLon = Math.sin(dLon / 2)
  const c = sinDLat * sinDLat + Math.cos(lat1) * Math.cos(lat2) * sinDLon * sinDLon
  return R * 2 * Math.atan2(Math.sqrt(c), Math.sqrt(1 - c))
}

export function paceSecPerKm(distanceMeters: number, timeSeconds: number): number {
  if (distanceMeters < 1) return 0
  return Math.round((timeSeconds / distanceMeters) * 1000)
}
```

- [ ] **Step 4: ?뚯뒪???듦낵 ?뺤씤**

```bash
npx jest __tests__/utils/geo.test.ts
```

Expected: PASS (3 tests)

- [ ] **Step 5: 而ㅻ컠**

```bash
git add utils/geo.ts __tests__/utils/geo.test.ts
git commit -m "feat: add geo utilities with haversine and pace calculation"
```

---

## Task 3: Runs API ?⑥닔 異붽?

**Files:**
- Modify: `utils/api.ts`

- [ ] **Step 1: api.ts?????+ ?⑥닔 異붽?**

`utils/api.ts` ?뚯씪 ?앹뿉 異붽?:

```typescript
export type LapPace = { km: number; paceSecPerKm: number }

export type RunResult = {
  totalDistanceMeters: number
  totalTimeSeconds: number
  averagePaceSeconds: number
  lapPaces: LapPace[]
}

export async function startRun(courseId: number): Promise<number> {
  const res = await fetch(`${BASE_URL}/api/runs/start`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId: 1, courseId }),
  })
  if (!res.ok) throw new Error('?щ떇 ?쒖옉 ?ㅽ뙣')
  const data = await res.json()
  return data.recordId
}

export async function updateRun(
  recordId: number,
  latitude: number,
  longitude: number,
): Promise<void> {
  await fetch(`${BASE_URL}/api/runs/update`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      recordId,
      latitude,
      longitude,
      recordedAt: new Date().toISOString(),
    }),
  })
}

export async function endRun(
  recordId: number,
  totalDistanceMeters: number,
  totalTimeSeconds: number,
  averagePaceSeconds: number,
): Promise<void> {
  await fetch(`${BASE_URL}/api/runs/end`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      recordId,
      totalDistanceMeters: Math.round(totalDistanceMeters),
      totalTimeSeconds,
      averagePaceSeconds,
    }),
  })
}
```

- [ ] **Step 2: TypeScript ?ㅻ쪟 ?놁쓬 ?뺤씤**

```bash
npx tsc --noEmit
```

Expected: ?ㅻ쪟 ?놁쓬

- [ ] **Step 3: 而ㅻ컠**

```bash
git add utils/api.ts
git commit -m "feat: add runs API functions (start/update/end)"
```

---

## Task 4: Mapbox GL JS HTML ?쒗뵆由?
**Files:**
- Create: `utils/mapHtml.ts`

- [ ] **Step 1: mapHtml.ts ?앹꽦**

`utils/mapHtml.ts`:

```typescript
import { RoutePoint } from './api'

export function getStaticMapHtml(token: string, points: RoutePoint[]): string {
  const coords = points.map((p) => [p.longitude, p.latitude])

  return `<!DOCTYPE html>
<html>
<head>
  <meta charset='utf-8' />
  <meta name='viewport' content='width=device-width, initial-scale=1, maximum-scale=1' />
  <link href='https://api.mapbox.com/mapbox-gl-js/v3.3.0/mapbox-gl.css' rel='stylesheet' />
  <script src='https://api.mapbox.com/mapbox-gl-js/v3.3.0/mapbox-gl.js'></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { width: 100vw; height: 100vh; overflow: hidden; }
    #map { width: 100%; height: 100%; }
  </style>
</head>
<body>
  <div id='map'></div>
  <script>
    mapboxgl.accessToken = '${token}';
    const coords = ${JSON.stringify(coords)};
    const map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v12',
      center: coords[Math.floor(coords.length / 2)],
      zoom: 14,
    });
    map.on('load', () => {
      map.addSource('route', {
        type: 'geojson',
        data: { type: 'Feature', geometry: { type: 'LineString', coordinates: coords } },
      });
      map.addLayer({
        id: 'route', type: 'line', source: 'route',
        layout: { 'line-join': 'round', 'line-cap': 'round' },
        paint: { 'line-color': '#2563eb', 'line-width': 5 },
      });
      const bounds = coords.reduce(
        (b, c) => b.extend(c),
        new mapboxgl.LngLatBounds(coords[0], coords[0])
      );
      map.fitBounds(bounds, { padding: 50, maxZoom: 16 });
      new mapboxgl.Marker({ color: '#059669' }).setLngLat(coords[0]).addTo(map);
      new mapboxgl.Marker({ color: '#dc2626' }).setLngLat(coords[coords.length - 1]).addTo(map);
    });
  </script>
</body>
</html>`
}

export function getRunningMapHtml(token: string, points: RoutePoint[]): string {
  const coords = points.map((p) => [p.longitude, p.latitude])

  return `<!DOCTYPE html>
<html>
<head>
  <meta charset='utf-8' />
  <meta name='viewport' content='width=device-width, initial-scale=1, maximum-scale=1' />
  <link href='https://api.mapbox.com/mapbox-gl-js/v3.3.0/mapbox-gl.css' rel='stylesheet' />
  <script src='https://api.mapbox.com/mapbox-gl-js/v3.3.0/mapbox-gl.js'></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { width: 100vw; height: 100vh; overflow: hidden; }
    #map { width: 100%; height: 100%; }
  </style>
</head>
<body>
  <div id='map'></div>
  <script>
    mapboxgl.accessToken = '${token}';
    const routeCoords = ${JSON.stringify(coords)};
    let posMarker = null;
    let trackedCoords = [];

    function lineGeoJSON(c) {
      const safe = c.length >= 2 ? c : [routeCoords[0], routeCoords[0]];
      return { type: 'Feature', geometry: { type: 'LineString', coordinates: safe } };
    }

    const map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v12',
      center: routeCoords[0],
      zoom: 15,
    });

    map.on('load', () => {
      map.addSource('full-route', { type: 'geojson', data: lineGeoJSON(routeCoords) });
      map.addLayer({
        id: 'full-route', type: 'line', source: 'full-route',
        layout: { 'line-join': 'round', 'line-cap': 'round' },
        paint: { 'line-color': '#d1d5db', 'line-width': 5 },
      });
      map.addSource('tracked', { type: 'geojson', data: lineGeoJSON([]) });
      map.addLayer({
        id: 'tracked', type: 'line', source: 'tracked',
        layout: { 'line-join': 'round', 'line-cap': 'round' },
        paint: { 'line-color': '#2563eb', 'line-width': 6 },
      });
    });

    function updatePosition(lat, lng) {
      const coord = [lng, lat];
      trackedCoords.push(coord);
      if (map.getSource('tracked')) {
        map.getSource('tracked').setData(lineGeoJSON(trackedCoords));
      }
      if (!posMarker) {
        const el = document.createElement('div');
        el.style.cssText = 'width:16px;height:16px;border-radius:50%;background:#2563eb;border:3px solid white;box-shadow:0 2px 8px rgba(0,0,0,0.3)';
        posMarker = new mapboxgl.Marker({ element: el }).setLngLat(coord).addTo(map);
      } else {
        posMarker.setLngLat(coord);
      }
      map.easeTo({ center: coord, duration: 800 });
    }

    window.addEventListener('message', (e) => {
      try {
        const data = JSON.parse(e.data);
        if (data.type === 'UPDATE_POSITION') updatePosition(data.latitude, data.longitude);
      } catch (_) {}
    });
  </script>
</body>
</html>`
}
```

- [ ] **Step 2: TypeScript ?뺤씤**

```bash
npx tsc --noEmit
```

Expected: ?ㅻ쪟 ?놁쓬

- [ ] **Step 3: 而ㅻ컠**

```bash
git add utils/mapHtml.ts
git commit -m "feat: add Mapbox GL JS HTML template generators"
```

---

## Task 5: RouteMapView 而댄룷?뚰듃

**Files:**
- Create: `components/RouteMapView.tsx`

- [ ] **Step 1: RouteMapView.tsx ?앹꽦**

`components/RouteMapView.tsx`:

```typescript
import React from 'react'
import { StyleSheet, View, Text } from 'react-native'
import WebView from 'react-native-webview'
import { RoutePoint } from '../utils/api'
import { getStaticMapHtml } from '../utils/mapHtml'

const MAPBOX_TOKEN = process.env.EXPO_PUBLIC_MAPBOX_TOKEN ?? ''

interface Props {
  points: RoutePoint[]
  height?: number
  flex?: boolean
}

export default function RouteMapView({ points, height = 400, flex = false }: Props) {
  if (points.length < 2) {
    return (
      <View style={[styles.fallback, flex ? { flex: 1 } : { height }]}>
        <Text style={styles.fallbackText}>寃쎈줈 ?곗씠???놁쓬</Text>
      </View>
    )
  }

  return (
    <WebView
      style={[styles.map, flex ? { flex: 1 } : { height }]}
      source={{ html: getStaticMapHtml(MAPBOX_TOKEN, points) }}
      originWhitelist={['*']}
      scrollEnabled={false}
      javaScriptEnabled
    />
  )
}

const styles = StyleSheet.create({
  map: { width: '100%' },
  fallback: {
    width: '100%',
    backgroundColor: '#f3f4f6',
    alignItems: 'center',
    justifyContent: 'center',
  },
  fallbackText: { fontSize: 13, color: '#6b7280' },
})
```

- [ ] **Step 2: TypeScript ?뺤씤**

```bash
npx tsc --noEmit
```

Expected: ?ㅻ쪟 ?놁쓬

- [ ] **Step 3: 而ㅻ컠**

```bash
git add components/RouteMapView.tsx
git commit -m "feat: add RouteMapView component (Mapbox GL JS via WebView)"
```

---

## Task 6: useRunningGPS ??
**Files:**
- Create: `hooks/useRunningGPS.ts`

- [ ] **Step 1: hooks ?붾젆?좊━ ?앹꽦 + useRunningGPS.ts ?묒꽦**

`hooks/useRunningGPS.ts`:

```typescript
import { useState, useRef, useCallback } from 'react'
import * as Location from 'expo-location'
import { RoutePoint, LapPace, RunResult, startRun, updateRun, endRun } from '../utils/api'
import { haversineMeters, paceSecPerKm } from '../utils/geo'

export interface UseRunningGPSReturn {
  currentLocation: RoutePoint | null
  trackedPoints: RoutePoint[]
  elapsedSeconds: number
  distanceMeters: number
  currentPaceSecPerKm: number
  lapPaces: LapPace[]
  start: (courseId: number) => Promise<void>
  stop: () => Promise<RunResult>
}

export function useRunningGPS(): UseRunningGPSReturn {
  const [currentLocation, setCurrentLocation] = useState<RoutePoint | null>(null)
  const [trackedPoints, setTrackedPoints] = useState<RoutePoint[]>([])
  const [elapsedSeconds, setElapsedSeconds] = useState(0)
  const [distanceMeters, setDistanceMeters] = useState(0)
  const [currentPaceSecPerKm, setCurrentPaceSecPerKm] = useState(0)
  const [lapPaces, setLapPaces] = useState<LapPace[]>([])

  const recordIdRef = useRef<number | null>(null)
  const locationSubRef = useRef<Location.LocationSubscription | null>(null)
  const timerRef = useRef<ReturnType<typeof setInterval> | null>(null)
  const startTimeRef = useRef(0)
  const distRef = useRef(0)
  const trackedRef = useRef<RoutePoint[]>([])
  const lapPacesRef = useRef<LapPace[]>([])
  const lapStartTimeRef = useRef(0)
  const lapStartDistRef = useRef(0)

  const start = useCallback(async (courseId: number) => {
    const { status } = await Location.requestForegroundPermissionsAsync()
    if (status !== 'granted') throw new Error('?꾩튂 沅뚰븳???꾩슂?⑸땲??)

    const recordId = await startRun(courseId)
    recordIdRef.current = recordId
    startTimeRef.current = Date.now()
    lapStartTimeRef.current = Date.now()
    lapStartDistRef.current = 0
    distRef.current = 0
    trackedRef.current = []
    lapPacesRef.current = []

    timerRef.current = setInterval(() => {
      setElapsedSeconds(Math.floor((Date.now() - startTimeRef.current) / 1000))
    }, 1000)

    locationSubRef.current = await Location.watchPositionAsync(
      { accuracy: Location.Accuracy.High, timeInterval: 3000, distanceInterval: 5 },
      (loc) => {
        const point: RoutePoint = {
          latitude: loc.coords.latitude,
          longitude: loc.coords.longitude,
        }
        setCurrentLocation(point)

        const prev = trackedRef.current[trackedRef.current.length - 1]
        if (prev) {
          const delta = haversineMeters(prev, point)
          distRef.current += delta
          setDistanceMeters(distRef.current)

          const elapsed = Math.floor((Date.now() - startTimeRef.current) / 1000)
          setCurrentPaceSecPerKm(paceSecPerKm(distRef.current, elapsed))

          // km 援ш컙 ?ъ꽦 泥댄겕
          const prevKm = Math.floor((distRef.current - delta) / 1000)
          const newKm = Math.floor(distRef.current / 1000)
          if (newKm > prevKm) {
            const lapTime = Math.floor((Date.now() - lapStartTimeRef.current) / 1000)
            const lapDist = distRef.current - lapStartDistRef.current
            const lap: LapPace = { km: newKm, paceSecPerKm: paceSecPerKm(lapDist, lapTime) }
            lapPacesRef.current = [...lapPacesRef.current, lap]
            setLapPaces([...lapPacesRef.current])
            lapStartTimeRef.current = Date.now()
            lapStartDistRef.current = distRef.current
          }
        }

        trackedRef.current = [...trackedRef.current, point]
        setTrackedPoints([...trackedRef.current])

        if (recordIdRef.current !== null) {
          updateRun(recordIdRef.current, point.latitude, point.longitude).catch(() => {})
        }
      },
    )
  }, [])

  const stop = useCallback(async (): Promise<RunResult> => {
    locationSubRef.current?.remove()
    locationSubRef.current = null
    if (timerRef.current) clearInterval(timerRef.current)

    const totalTime = Math.floor((Date.now() - startTimeRef.current) / 1000)
    const totalDist = distRef.current
    const avgPace = paceSecPerKm(totalDist, totalTime)

    if (recordIdRef.current !== null) {
      await endRun(recordIdRef.current, Math.round(totalDist), totalTime, avgPace)
    }

    return { totalDistanceMeters: totalDist, totalTimeSeconds: totalTime, averagePaceSeconds: avgPace, lapPaces: lapPacesRef.current }
  }, [])

  return { currentLocation, trackedPoints, elapsedSeconds, distanceMeters, currentPaceSecPerKm, lapPaces, start, stop }
}
```

- [ ] **Step 2: TypeScript ?뺤씤**

```bash
npx tsc --noEmit
```

Expected: ?ㅻ쪟 ?놁쓬

- [ ] **Step 3: 而ㅻ컠**

```bash
git add hooks/useRunningGPS.ts
git commit -m "feat: add useRunningGPS hook with GPS tracking and pace calculation"
```

---

## Task 7: RunningMapView 而댄룷?뚰듃

**Files:**
- Create: `components/RunningMapView.tsx`

- [ ] **Step 1: RunningMapView.tsx ?앹꽦**

`components/RunningMapView.tsx`:

```typescript
import React, { useRef, useEffect } from 'react'
import { StyleSheet } from 'react-native'
import WebView from 'react-native-webview'
import { RoutePoint } from '../utils/api'
import { getRunningMapHtml } from '../utils/mapHtml'

const MAPBOX_TOKEN = process.env.EXPO_PUBLIC_MAPBOX_TOKEN ?? ''

interface Props {
  points: RoutePoint[]
  currentLocation: RoutePoint | null
}

export default function RunningMapView({ points, currentLocation }: Props) {
  const webViewRef = useRef<WebView>(null)

  useEffect(() => {
    if (!currentLocation || !webViewRef.current) return
    const script = `
      updatePosition(${currentLocation.latitude}, ${currentLocation.longitude});
      true;
    `
    webViewRef.current.injectJavaScript(script)
  }, [currentLocation])

  return (
    <WebView
      ref={webViewRef}
      style={styles.map}
      source={{ html: getRunningMapHtml(MAPBOX_TOKEN, points) }}
      originWhitelist={['*']}
      scrollEnabled={false}
      javaScriptEnabled
    />
  )
}

const styles = StyleSheet.create({
  map: { flex: 1, width: '100%' },
})
```

- [ ] **Step 2: TypeScript ?뺤씤**

```bash
npx tsc --noEmit
```

Expected: ?ㅻ쪟 ?놁쓬

- [ ] **Step 3: 而ㅻ컠**

```bash
git add components/RunningMapView.tsx
git commit -m "feat: add RunningMapView component with real-time GPS tracking"
```

---

## Task 8: 肄붿뒪 ?좏깮 ???곸꽭 ?붾㈃ ?곌껐 (points ?뚮엺 異붽?)

**Files:**
- Modify: `app/(tabs)/course/index.tsx:54-65`
- Modify: `app/(tabs)/course/detail.tsx`

- [ ] **Step 1: course/index.tsx??handleSelectCourse??points 異붽?**

`app/(tabs)/course/index.tsx` 54-65踰?以꾩쓽 `handleSelectCourse` ?⑥닔瑜?援먯껜:

```typescript
function handleSelectCourse(course: RouteOption) {
  setShowResults(false)
  router.push({
    pathname: '/(tabs)/course/detail',
    params: {
      courseId: course.courseId.toString(),
      courseName: course.courseName,
      distance: course.totalDistanceMeters.toString(),
      duration: course.estimatedDurationSeconds.toString(),
      points: JSON.stringify(course.points),
    },
  })
}
```

- [ ] **Step 2: course/detail.tsx ?꾩껜 援먯껜**

`app/(tabs)/course/detail.tsx`:

```typescript
import React, { useMemo } from 'react'
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { router, useLocalSearchParams } from 'expo-router'
import { Colors } from '../../../constants/Colors'
import RouteMapView from '../../../components/RouteMapView'
import { metersToKm } from '../../../utils/format'
import { RoutePoint } from '../../../utils/api'

export default function CourseDetailScreen() {
  const { courseId, courseName, distance, duration, points } = useLocalSearchParams<{
    courseId: string
    courseName: string
    distance: string
    duration: string
    points: string
  }>()

  const parsedPoints = useMemo<RoutePoint[]>(() => {
    try { return JSON.parse(points ?? '[]') } catch { return [] }
  }, [points])

  function handleStart() {
    router.push({
      pathname: '/running',
      params: { courseId, courseName, distance, duration, points },
    })
  }

  return (
    <SafeAreaView style={styles.safe}>
      <View style={styles.header}>
        <View style={styles.nameRow}>
          <Text style={styles.courseName}>{courseName}</Text>
          <Text style={styles.distBadge}>{metersToKm(Number(distance))}km</Text>
        </View>
      </View>

      <RouteMapView points={parsedPoints} height={520} />

      <View style={styles.footer}>
        <TouchableOpacity style={styles.startBtn} onPress={handleStart}>
          <Text style={styles.startBtnText}>異쒕컻</Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  )
}

const styles = StyleSheet.create({
  safe: { flex: 1, backgroundColor: Colors.BACKGROUND },
  header: {
    paddingHorizontal: 16, paddingVertical: 12,
    borderBottomWidth: 1, borderBottomColor: Colors.BORDER,
    backgroundColor: Colors.CARD,
  },
  nameRow: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' },
  courseName: { fontSize: 14, fontWeight: '900', color: Colors.TEXT_PRIMARY },
  distBadge: { fontSize: 11, fontWeight: '800', color: Colors.TEXT_SECONDARY },
  footer: {
    padding: 16, backgroundColor: Colors.NAVBAR_BG,
    borderTopWidth: 1, borderTopColor: Colors.BORDER,
  },
  startBtn: {
    height: 56, backgroundColor: Colors.PRIMARY,
    borderRadius: 12, alignItems: 'center', justifyContent: 'center',
  },
  startBtnText: { fontSize: 16, fontWeight: '900', color: Colors.TEXT_WHITE },
})
```

- [ ] **Step 3: TypeScript ?뺤씤**

```bash
npx tsc --noEmit
```

Expected: ?ㅻ쪟 ?놁쓬

- [ ] **Step 4: 而ㅻ컠**

```bash
git add app/(tabs)/course/index.tsx app/(tabs)/course/detail.tsx
git commit -m "feat: pass route points through course selection to detail screen"
```

---

## Task 9: running/index.tsx ???ㅼ젣 GPS + 吏?꾨줈 援먯껜

**Files:**
- Modify: `app/running/index.tsx`
- Delete: `components/MapPlaceholder.tsx` (Task 9 ?꾨즺 ??

- [ ] **Step 1: running/index.tsx ?꾩껜 援먯껜**

`app/running/index.tsx`:

```typescript
import React, { useState, useMemo } from 'react'
import { View, Text, TouchableOpacity, StyleSheet, ScrollView, Alert } from 'react-native'
import { SafeAreaView } from 'react-native-safe-area-context'
import { router, useLocalSearchParams } from 'expo-router'
import { Star } from 'lucide-react-native'
import { Colors } from '../../constants/Colors'
import RunningMapView from '../../components/RunningMapView'
import StatBox from '../../components/StatBox'
import PaceItem from '../../components/PaceItem'
import { metersToKm, secondsToTime, secondsToPace } from '../../utils/format'
import { useRunningGPS } from '../../hooks/useRunningGPS'
import { RoutePoint, RunResult } from '../../utils/api'

type Status = 'waiting' | 'running' | 'saving' | 'survey'

export default function RunningScreen() {
  const { courseId, courseName, distance, points } = useLocalSearchParams<{
    courseId: string; courseName: string; distance: string; duration: string; points: string
  }>()
  const totalDist = Number(distance ?? 5000)

  const parsedPoints = useMemo<RoutePoint[]>(() => {
    try { return JSON.parse(points ?? '[]') } catch { return [] }
  }, [points])

  const [status, setStatus] = useState<Status>('waiting')
  const [rating, setRating] = useState(0)
  const [saveRoute, setSaveRoute] = useState(false)
  const [result, setResult] = useState<RunResult | null>(null)

  const gps = useRunningGPS()

  const progressPct = Math.min((gps.distanceMeters / totalDist) * 100, 100)

  async function handleStart() {
    try {
      await gps.start(Number(courseId ?? 0))
      setStatus('running')
    } catch (e: any) {
      Alert.alert('?ㅻ쪟', e.message)
    }
  }

  async function handleStop() {
    try {
      const r = await gps.stop()
      setResult(r)
      setStatus('saving')
    } catch {
      setStatus('saving')
    }
  }

  const bestPace = result?.lapPaces.length
    ? Math.min(...result.lapPaces.map((l) => l.paceSecPerKm))
    : 0
  const calories = result ? Math.round(result.totalDistanceMeters / 1000 * 60) : 0

  return (
    <SafeAreaView style={styles.safe}>
      <View style={styles.header}>
        <Text style={styles.courseName}>{courseName ?? '肄붿뒪'}</Text>
        <View style={styles.headerRight}>
          <TouchableOpacity
            style={styles.weatherBtn}
            onPress={() => router.push('/running/weather')}
          >
            <Text style={styles.weatherText}>? 23째</Text>
          </TouchableOpacity>
          {status === 'running' && (
            <TouchableOpacity style={styles.stopBtn} onPress={handleStop}>
              <Text style={styles.stopText}>以묐떒</Text>
            </TouchableOpacity>
          )}
        </View>
      </View>

      {status === 'running' && (
        <View style={styles.progress}>
          <View style={styles.progressRow}>
            <Text style={styles.progressLabel}>吏꾪뻾??/Text>
            <Text style={styles.progressLabel}>
              {(gps.distanceMeters / 1000).toFixed(1)} / {metersToKm(totalDist)}km  {Math.round(progressPct)}%
            </Text>
          </View>
          <View style={styles.progressBar}>
            <View style={[styles.progressFill, { width: `${progressPct}%` as any }]} />
          </View>
        </View>
      )}

      <RunningMapView points={parsedPoints} currentLocation={gps.currentLocation} />

      <View style={styles.footer}>
        {status === 'waiting' && (
          <TouchableOpacity style={styles.startBtn} onPress={handleStart}>
            <Text style={styles.startBtnText}>?쒖옉</Text>
          </TouchableOpacity>
        )}
        <View style={styles.statsRow}>
          <StatBox
            label="嫄곕━"
            value={status === 'running' ? (gps.distanceMeters / 1000).toFixed(2) : '0.00'}
            unit="km"
          />
          <StatBox label="?쒓컙" value={secondsToTime(gps.elapsedSeconds)} />
          <StatBox
            label="?꾩옱"
            value={status === 'running' && gps.currentPaceSecPerKm > 0
              ? secondsToPace(gps.currentPaceSecPerKm)
              : '??}
            unit={status === 'running' && gps.currentPaceSecPerKm > 0 ? '/km' : undefined}
          />
          <StatBox
            label="理쒓퀬"
            value={gps.lapPaces.length > 0
              ? secondsToPace(Math.min(...gps.lapPaces.map((l) => l.paceSecPerKm)))
              : '??}
            unit={gps.lapPaces.length > 0 ? '/km' : undefined}
          />
        </View>
      </View>

      {status === 'saving' && result && (
        <View style={styles.resultOverlay}>
          <ScrollView contentContainerStyle={styles.resultContent}>
            <View style={styles.resultHeader}>
              <Text style={styles.completeLabel}>COMPLETE</Text>
              <Text style={styles.completeTitle}>?꾩＜!</Text>
              <Text style={styles.resultCourseName}>{courseName}</Text>
            </View>
            <View style={styles.mainCard}>
              <View style={styles.mainStatRow}>
                {[
                  { label: '珥?嫄곕━', value: metersToKm(result.totalDistanceMeters), unit: 'km' },
                  { label: '珥??쒓컙', value: secondsToTime(result.totalTimeSeconds), unit: '' },
                  { label: '?됯퇏 ?섏씠??, value: secondsToPace(result.averagePaceSeconds), unit: '/km' },
                ].map((s, i) => (
                  <React.Fragment key={s.label}>
                    {i > 0 && <View style={styles.mainDivider} />}
                    <View style={styles.mainStatItem}>
                      <Text style={styles.mainStatLabel}>{s.label}</Text>
                      <Text style={styles.mainStatValue}>{s.value}</Text>
                      <Text style={styles.mainStatUnit}>{s.unit}</Text>
                    </View>
                  </React.Fragment>
                ))}
              </View>
            </View>
            <View style={styles.extraRow}>
              <View style={styles.extraCard}>
                <Text style={styles.extraLabel}>理쒓퀬 ?섏씠??/Text>
                <Text style={styles.extraValue}>{bestPace > 0 ? secondsToPace(bestPace) : '??}</Text>
              </View>
              <View style={styles.extraCard}>
                <Text style={styles.extraLabel}>移쇰줈由?/Text>
                <Text style={styles.extraValue}>{calories}</Text>
              </View>
            </View>
            {result.lapPaces.length > 0 && (
              <View style={styles.segCard}>
                <Text style={styles.segTitle}>援ш컙蹂??섏씠??/Text>
                {result.lapPaces.map((s) => (
                  <PaceItem key={s.km} km={s.km} paceSeconds={s.paceSecPerKm} />
                ))}
              </View>
            )}
            <TouchableOpacity style={styles.continueBtn} onPress={() => setStatus('survey')}>
              <Text style={styles.continueBtnText}>怨꾩냽</Text>
            </TouchableOpacity>
          </ScrollView>
        </View>
      )}

      {status === 'survey' && (
        <View style={styles.surveyOverlay}>
          <View style={styles.surveyModal}>
            <Text style={styles.surveyTitle}>?섍퀬?섏뀲?댁슂!</Text>
            <Text style={styles.surveySubtitle}>?ㅻ뒛 ?щ떇? ?대뼚?⑤굹??</Text>
            <View style={styles.starRow}>
              {[1, 2, 3, 4, 5].map((star) => (
                <TouchableOpacity key={star} onPress={() => setRating(star)}>
                  <Star size={36} color="#d97706" fill={star <= rating ? '#d97706' : 'transparent'} />
                </TouchableOpacity>
              ))}
            </View>
            <TouchableOpacity style={styles.checkRow} onPress={() => setSaveRoute((v) => !v)}>
              <View style={[styles.checkbox, saveRoute && styles.checkboxChecked]}>
                {saveRoute && <Text style={styles.checkmark}>??/Text>}
              </View>
              <Text style={styles.checkLabel}>??猷⑦듃 ??ν븯湲?/Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={[styles.doneBtn, rating === 0 && styles.doneBtnDisabled]}
              onPress={() => { if (rating > 0) router.replace('/(tabs)/record') }}
            >
              <Text style={styles.doneBtnText}>?꾨즺</Text>
            </TouchableOpacity>
          </View>
        </View>
      )}
    </SafeAreaView>
  )
}

const styles = StyleSheet.create({
  safe: { flex: 1, backgroundColor: Colors.BACKGROUND },
  header: {
    flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center',
    paddingHorizontal: 16, paddingVertical: 12,
    backgroundColor: Colors.CARD, borderBottomWidth: 1, borderBottomColor: Colors.BORDER,
  },
  courseName: { fontSize: 14, fontWeight: '900', color: Colors.TEXT_PRIMARY },
  headerRight: { flexDirection: 'row', alignItems: 'center', gap: 8 },
  weatherBtn: { backgroundColor: Colors.SURFACE_DARK, borderRadius: 8, paddingHorizontal: 10, paddingVertical: 4 },
  weatherText: { fontSize: 12, fontWeight: '800', color: Colors.TEXT_PRIMARY },
  stopBtn: { backgroundColor: Colors.DANGER, borderRadius: 8, paddingHorizontal: 10, paddingVertical: 4 },
  stopText: { fontSize: 12, fontWeight: '800', color: Colors.TEXT_WHITE },
  progress: { paddingHorizontal: 16, paddingVertical: 8, backgroundColor: Colors.CARD },
  progressRow: { flexDirection: 'row', justifyContent: 'space-between', marginBottom: 4 },
  progressLabel: { fontSize: 9, fontWeight: '800', color: Colors.TEXT_SECONDARY },
  progressBar: { height: 6, backgroundColor: Colors.BORDER, borderRadius: 3 },
  progressFill: { height: 6, backgroundColor: Colors.PRIMARY, borderRadius: 3 },
  footer: { padding: 16, gap: 12, backgroundColor: Colors.NAVBAR_BG, borderTopWidth: 1, borderTopColor: Colors.BORDER },
  startBtn: { height: 48, backgroundColor: Colors.PRIMARY, borderRadius: 12, alignItems: 'center', justifyContent: 'center' },
  startBtnText: { fontSize: 16, fontWeight: '900', color: Colors.TEXT_WHITE },
  statsRow: { flexDirection: 'row', gap: 8 },
  resultOverlay: { position: 'absolute', top: 0, left: 0, right: 0, bottom: 0, backgroundColor: Colors.CARD, zIndex: 50 },
  resultContent: { padding: 16, paddingBottom: 40 },
  resultHeader: { alignItems: 'center', marginBottom: 24, marginTop: 16 },
  completeLabel: { fontSize: 11, fontWeight: '800', color: Colors.SUCCESS, letterSpacing: 2 },
  completeTitle: { fontSize: 24, fontWeight: '900', color: Colors.TEXT_PRIMARY, marginTop: 4 },
  resultCourseName: { fontSize: 14, color: Colors.TEXT_SECONDARY, marginTop: 4 },
  mainCard: { backgroundColor: Colors.SURFACE_DARK, borderRadius: 16, paddingVertical: 20, marginBottom: 16 },
  mainStatRow: { flexDirection: 'row', justifyContent: 'space-around', alignItems: 'center' },
  mainStatItem: { alignItems: 'center', flex: 1 },
  mainStatLabel: { fontSize: 10, fontWeight: '800', color: Colors.TEXT_SECONDARY, marginBottom: 4 },
  mainStatValue: { fontSize: 22, fontWeight: '900', color: Colors.TEXT_PRIMARY },
  mainStatUnit: { fontSize: 11, color: Colors.TEXT_SECONDARY, marginTop: 2 },
  mainDivider: { width: 1, height: 40, backgroundColor: Colors.BORDER },
  extraRow: { flexDirection: 'row', gap: 12, marginBottom: 16 },
  extraCard: { flex: 1, backgroundColor: Colors.SURFACE_DARK, borderRadius: 12, padding: 16, alignItems: 'center' },
  extraLabel: { fontSize: 10, fontWeight: '800', color: Colors.TEXT_SECONDARY, marginBottom: 4 },
  extraValue: { fontSize: 18, fontWeight: '900', color: Colors.TEXT_PRIMARY },
  segCard: { backgroundColor: Colors.SURFACE_DARK, borderRadius: 16, padding: 16, marginBottom: 16 },
  segTitle: { fontSize: 14, fontWeight: '900', color: Colors.TEXT_PRIMARY, marginBottom: 12 },
  continueBtn: { height: 48, backgroundColor: Colors.PRIMARY, borderRadius: 12, alignItems: 'center', justifyContent: 'center' },
  continueBtnText: { fontSize: 14, fontWeight: '900', color: Colors.TEXT_WHITE },
  surveyOverlay: { position: 'absolute', top: 0, left: 0, right: 0, bottom: 0, backgroundColor: 'rgba(0,0,0,0.5)', zIndex: 50, justifyContent: 'center', alignItems: 'center' },
  surveyModal: { width: '88%', backgroundColor: Colors.CARD, borderRadius: 18, padding: 24, alignItems: 'center', gap: 16, shadowColor: '#0f172a', shadowOffset: { width: 0, height: 8 }, shadowOpacity: 0.15, shadowRadius: 24, elevation: 8 },
  surveyTitle: { fontSize: 20, fontWeight: '800', color: Colors.TEXT_PRIMARY },
  surveySubtitle: { fontSize: 14, color: Colors.TEXT_SECONDARY, marginTop: -8 },
  starRow: { flexDirection: 'row', gap: 8, marginVertical: 4 },
  checkRow: { flexDirection: 'row', alignItems: 'center', gap: 10, alignSelf: 'flex-start', paddingHorizontal: 4 },
  checkbox: { width: 20, height: 20, borderRadius: 4, borderWidth: 2, borderColor: Colors.BORDER, alignItems: 'center', justifyContent: 'center' },
  checkboxChecked: { backgroundColor: Colors.PRIMARY, borderColor: Colors.PRIMARY },
  checkmark: { fontSize: 12, color: Colors.TEXT_WHITE, fontWeight: '900' },
  checkLabel: { fontSize: 14, fontWeight: '800', color: Colors.TEXT_PRIMARY },
  doneBtn: { width: '100%', height: 48, backgroundColor: Colors.PRIMARY, borderRadius: 10, alignItems: 'center', justifyContent: 'center' },
  doneBtnDisabled: { backgroundColor: Colors.BORDER },
  doneBtnText: { fontSize: 14, fontWeight: '900', color: Colors.TEXT_WHITE },
})
```

- [ ] **Step 2: MapPlaceholder.tsx ??젣**

```bash
git rm components/MapPlaceholder.tsx
```

- [ ] **Step 3: TypeScript ?뺤씤**

```bash
npx tsc --noEmit
```

Expected: ?ㅻ쪟 ?놁쓬

- [ ] **Step 4: 而ㅻ컠**

```bash
git add app/running/index.tsx
git commit -m "feat: wire RunningMapView + useRunningGPS into running screen"
```

---

## ?뚯뒪??泥댄겕由ъ뒪??(Expo Go濡??뺤씤)

```bash
npx expo start
```

- [ ] 肄붿뒪 李얘린 ??寃????異붿쿇 肄붿뒪 ?좏깮 ???곸꽭 ?붾㈃??吏???쒖떆??- [ ] 吏?꾩뿉 ?꾩껜 猷⑦듃(?뚮? ?? + ?쒖옉(珥덈줉)/??鍮④컯) 留덉빱 蹂댁엫
- [ ] "異쒕컻" ???щ떇 ?붾㈃??吏???쒖떆??- [ ] "?쒖옉" ??GPS 沅뚰븳 ?붿껌 ???덉슜 ???щ떇 ?쒖옉
- [ ] ?대룞 ???뚮? ?먯씠 吏?꾩뿉???吏곸씠怨? 吏?섏삩 援ш컙???뚮옑寃??쒖떆??- [ ] ?섎떒 ?ㅽ꺈(嫄곕━, ?쒓컙, ?꾩옱 ?섏씠?? ?ㅼ떆媛??낅뜲?댄듃
- [ ] "以묐떒" ???꾩＜ ?붾㈃???ㅼ젣 嫄곕━/?쒓컙/?섏씠???쒖떆??- [ ] 援ш컙蹂??섏씠??紐⑸줉 ?쒖떆??(1km ?댁긽 ??寃쎌슦)
