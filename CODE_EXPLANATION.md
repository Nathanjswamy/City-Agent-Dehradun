# 🏙️ CityAgent DEH — Complete Code Explanation

> A full, line-by-line explanation of the project, written so you can understand and explain it to anyone.

---

## 🗂️ Project Structure (Big Picture)

```
cityagent_deh/
├── index.html          ← The single HTML page the browser loads
├── package.json        ← Project config & list of all libraries used
├── vite.config.ts      ← Dev server config (Vite builds & serves the app)
├── tailwind.config.js  ← Design system (colors, fonts, etc.)
└── src/
    ├── main.jsx        ← Entry point — starts the React app
    ├── App.jsx         ← Root component — the brain of the whole app
    ├── index.css       ← Global styles
    ├── api/
    │   ├── cityagent.js   ← Fetches real data (AQI, weather, flood, traffic)
    │   └── kimiChat.js    ← AI chatbot (Llama 3.1 via NVIDIA NIM)
    ├── hooks/
    │   └── useCity.js     ← Manages city selection & GPS detection
    ├── components/
    │   ├── Sidebar.jsx    ← Left nav bar
    │   ├── Topbar.jsx     ← Top bar with search, clock, location
    │   ├── StatCard.jsx   ← Small number cards (sensors, alerts count, etc.)
    │   ├── MetricCard.jsx ← Detailed metric cards (AQI, traffic, etc.)
    │   ├── AnomalyChart.jsx ← Chart showing anomalies over time
    │   ├── AQIGauge.jsx   ← Air quality circular gauge
    │   ├── AlertFeed.jsx  ← Live alert list
    │   ├── MapPanel.jsx   ← Interactive map with AQI overlay
    │   ├── Pipeline.jsx   ← Data pipeline status
    │   ├── ChatBot.jsx    ← Floating AI assistant
    │   └── TelegramButton.jsx ← Send city report to Telegram
    └── data/
        └── mockData.js    ← Fake data used when real APIs aren't available
```

---

## 📄 `index.html` — The Browser's Starting Point

```html
<!doctype html>               <!-- Tells browser: "this is an HTML5 page" -->
<html lang="en">              <!-- Language is English (helps screen readers & SEO) -->
  <head>
    <meta charset="UTF-8" />  <!-- Support all international characters -->
    <link rel="icon" .../>    <!-- The small tab icon (favicon) -->
    <meta name="viewport" ... />  <!-- Makes site look good on phones -->
    <title>frontend</title>   <!-- Text shown in the browser tab -->
  </head>
  <body>
    <div id="root"></div>     <!-- ⭐ This empty div is where React injects the entire app -->
    <script type="module" src="/src/main.jsx"></script>  <!-- Loads your React app -->
  </body>
</html>
```

**Think of it like this:** The HTML file is an empty room. React is the interior designer that fills it with furniture (UI components).

---

## 📦 `package.json` — Project Identity Card

```json
{
  "name": "citygen",          // Name of the project
  "private": true,            // Not published to npm (keeps it private)
  "version": "0.0.0",         // Version number (still in early dev)
  "type": "module",           // Use modern JS imports (import/export syntax)

  "scripts": {
    "dev": "vite",            // npm run dev → starts local dev server
    "build": "vite build",    // npm run build → creates production files
    "lint": "eslint .",       // npm run lint → checks code for errors/style
    "preview": "vite preview" // npm run preview → preview the built app
  },

  "dependencies": {           // Libraries used in the actual app
    "axios": ...,             // Makes HTTP requests (fetch data from APIs)
    "framer-motion": ...,     // Beautiful animations
    "leaflet": ...,           // Interactive maps
    "lucide-react": ...,      // Icon library (Wind, Bell, Search icons etc)
    "react": ...,             // The core React library
    "react-dom": ...,         // Connects React to the browser DOM
    "react-leaflet": ...,     // React wrapper for Leaflet maps
    "recharts": ...,          // Chart/graph library
    "tailwind-merge": ...     // Merges Tailwind CSS classes smartly
  },

  "devDependencies": {        // Tools used ONLY during development
    "vite": ...,              // The super-fast build tool / dev server
    "tailwindcss": ...,       // CSS framework for styling
    "typescript": ...,        // TypeScript support (type checking)
    "eslint": ...             // Code linting (checks for bugs/bad patterns)
  }
}
```

---

## 🚀 `src/main.jsx` — The App's Ignition Switch

```jsx
import { StrictMode } from 'react'           // StrictMode = React's safety inspector
import { createRoot } from 'react-dom/client' // Creates the React root in HTML
import './index.css'                          // Loads global styles
import App from './App.jsx'                   // Imports our main App component

createRoot(document.getElementById('root'))  // Finds the <div id="root"> in index.html
  .render(                                   // Tells React: "draw stuff here"
    <StrictMode>                             // Wraps app in strict mode (helps catch bugs)
      <App />                                // Renders our entire app
    </StrictMode>
  )
```

**Analogy:** `main.jsx` is the key that starts the car. It finds the engine (`App`) and the road (`#root` div) and starts driving.

---

## 🎛️ `src/index.css` — Global Styling

```css
/* Load Google Fonts — DM Mono (code font) and DM Sans (main font) */
@import url('https://fonts.googleapis.com/css2?...');

@tailwind base;       /* Tailwind's base reset styles */
@tailwind components; /* Tailwind's reusable component classes */
@tailwind utilities;  /* Tailwind's utility classes (flex, p-4, text-sm, etc.) */

@layer base {
  body {
    @apply bg-bg-deep      /* Dark background color */
           text-primary    /* Default text color */
           font-sans       /* Use DM Sans font */
           antialiased     /* Smooth font rendering */
           overflow-hidden; /* No scroll on body — content scrolls inside panels */
  }
}

/* Custom thin scrollbar instead of ugly browser default */
@layer utilities {
  .custom-scrollbar::-webkit-scrollbar { width: 4px; }
  .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
  .custom-scrollbar::-webkit-scrollbar-thumb {
    background: rgba(255,255,255,0.07); /* Very subtle scroll thumb */
    border-radius: 2px;
  }
}
```

---

## 🧠 `src/App.jsx` — The Brain / Root Component

This is the most important file. It:
1. Tracks which city is selected
2. Fetches all live data
3. Passes data to all child components

### Imports

```jsx
import React, { useEffect, useState } from 'react'; // Core React hooks

// Import all visual components
import Sidebar from './components/Sidebar';
import Topbar  from './components/Topbar';
import StatCard from './components/StatCard';
// ... etc.

// Import icons from Lucide icon library
import { Wind, Briefcase, CloudRain, MessageSquare, Construction } from 'lucide-react';

// Import city management hook
import { useCity } from './hooks/useCity';

// Import all API fetch functions
import { fetchAQI, fetchWeather, fetchFlood, fetchTraffic, ... } from './api/cityagent';
```

### State Variables

```jsx
function App() {
  const [data, setData]             = useState(null);       // All fetched city data
  const [alerts, setAlerts]         = useState([]);          // Live alert notifications
  const [loading, setLoading]       = useState(true);        // true = show loading screen
  const [refreshing, setRefreshing] = useState(false);       // true = show refresh bar
  const [activePage, setActivePage] = useState('overview'); // Which tab is active
```

> **useState** is like a variable that, when changed, automatically re-draws the component on screen.

### City Hook

```jsx
  const {
    city,           // Current city object { name, state, lat, lon }
    locating,       // Is GPS currently searching? (true/false)
    locationError,  // Error message if GPS failed
    detectLocation, // Function: use GPS to find location
    searchCities,   // Function: search for a city by name
    selectCity      // Function: set a city from search results
  } = useCity();
```

### Data Fetching (useEffect)

```jsx
  useEffect(() => {         // Runs whenever city.lat or city.lon changes
    let cancelled = false;  // Safety flag to prevent updating stale data

    if (!data) setLoading(true);   // First load → show loading screen
    else setRefreshing(true);      // City change → show refresh indicator

    const loadData = async () => {
      const { lat, lon, name: cityName } = city;

      // ⭐ Fetch multiple APIs AT THE SAME TIME (parallel, not one-by-one)
      const [aqi, weather, flood, traffic, stats] = await Promise.all([
        fetchAQI(lat, lon),      // Air Quality Index
        fetchWeather(lat, lon),  // Temperature, wind, humidity
        fetchFlood(lat, lon),    // River discharge / flood risk
        fetchTraffic(lat, lon),  // Vehicles, congestion, speed
        fetchSystemStats(),      // App system stats
      ]);

      // Use real AQI to make mock data feel city-specific
      const aqiValue = aqi?.value ?? 60;
      const [social, anomalies, alertsResult] = await Promise.all([
        fetchSocialSignals(cityName, aqiValue),
        fetchAnomalies(cityName, aqiValue),
        fetchAlerts(cityName, aqiValue),
      ]);

      if (!cancelled) {
        // Give each alert a unique ID to prevent React bugs
        const cityPrefix = cityName.replace(/\s+/g, '').slice(0, 4).toUpperCase();
        const uniqueAlerts = alertsResult.alerts.map((a, i) => ({
          ...a,
          id: `${cityPrefix}-${i}`, // e.g. "DEHR-0", "DEHR-1"
        }));

        setData({ aqi, weather, flood, traffic, social, stats, anomalies });
        setAlerts(uniqueAlerts);
        setLoading(false);
        setRefreshing(false);
      }
    };

    loadData();                              // Fetch immediately
    const interval = setInterval(loadData, 60000); // Refresh every 60 seconds

    return () => {
      cancelled = true;          // Stop stale updates
      clearInterval(interval);  // Stop auto-refresh
    };
  }, [city.lat, city.lon]); // Only re-run when coordinates change
```

### WebSocket for Live Alerts

```jsx
  useEffect(() => {
    // Simulates a WebSocket that pushes new alerts every 15 seconds
    const socket = connectAlertSocket((newAlert) => {
      setAlerts(prev => {
        if (prev.some(a => a.id === newAlert.id)) return prev; // Skip duplicates
        return [{ ...newAlert, id: `WS-${newAlert.id}` }, ...prev.slice(0, 9)];
        // Add new alert at top, keep max 10
      });
    });
    return () => socket.close(); // Disconnect when component unmounts
  }, []);
```

### Loading Screen

```jsx
  if (loading || !data) {
    return (
      <div className="h-screen flex items-center justify-center">
        {/* Three bouncing dots — each delayed by 0.15s so they wave */}
        {[0, 1, 2].map(i => (
          <div key={i} className="w-2 h-2 rounded-full animate-bounce"
               style={{ animationDelay: `${i * 0.15}s` }} />
        ))}
        <div>Fetching live data for {city.name}…</div>
      </div>
    );
  }
```

### Main Dashboard Layout

```jsx
  return (
    <div className="h-screen w-screen flex flex-row overflow-hidden">
      <Sidebar activePage={activePage} onChangePage={setActivePage} />

      <main className="flex-1 flex flex-col overflow-hidden">
        <Topbar city={city} data={data} alerts={alerts} ... />

        {activePage === 'overview' ? (
          <div className="flex-1 overflow-y-auto p-4">

            {/* ROW 1: 4 stat cards */}
            <div className="grid grid-cols-4 gap-3 mb-4">
              <StatCard label="Live Data Streams" value={data.stats.activeSensors} />
              <StatCard label="Anomalies Today"   value={data.stats.anomaliesToday} />
              <StatCard label="API Uptime"         value={data.stats.systemUptime} unit="%" />
              <StatCard label="Active Alerts"      value={alerts.length} />
            </div>

            {/* ROW 2: 4 metric cards */}
            <div className="grid grid-cols-4 gap-3 mb-4">
              <MetricCard title="US AQI"          value={data.aqi.value} />
              <MetricCard title="Traffic Flow"    ... />
              <MetricCard title="River Discharge" ... />
              <MetricCard title="Social Signals"  ... />
            </div>

            {/* ROW 3: Charts, map, AQI gauge, alerts */}
            <div className="grid grid-cols-[1fr_320px] gap-3">
              {/* Left: anomaly chart + pipeline + map */}
              {/* Right: AQI gauge + alert feed */}
            </div>
          </div>
        ) : (
          // Any other page → "Coming Soon" screen
          <div className="flex items-center justify-center">
            <Construction size={48} />
            <h2>{activePage} Module — Coming Soon!</h2>
          </div>
        )}
      </main>

      {/* Floating AI Chatbot — always visible */}
      <ChatBot city={city} data={data} alerts={alerts} />
    </div>
  );
```

---

## 🌍 `src/hooks/useCity.js` — City & Location Manager

A **hook** is a reusable function that manages state and logic.

```js
const DEFAULT_CITY = { name: 'Dehradun', state: 'Uttarakhand', lat: 30.3165, lon: 78.0322 };

export function useCity() {
  const [city, setCity]               = useState(DEFAULT_CITY);
  const [locating, setLocating]       = useState(false);
  const [locationError, setLocationError] = useState(null);

  // FUNCTION 1: Detect user's current location via GPS
  const detectLocation = useCallback(() => {
    if (!navigator.geolocation) {
      setLocationError('Geolocation not supported');
      return;
    }
    setLocating(true);

    navigator.geolocation.getCurrentPosition(
      async ({ coords }) => {                   // SUCCESS: got GPS coords
        const { latitude, longitude } = coords;
        // Free reverse geocoding — get city name from coordinates
        const r = await axios.get('https://api.bigdatacloud.net/data/reverse-geocode-client', ...);
        const name = r.data.city || r.data.locality || 'Your Location';
        setCity({ name, lat: latitude, lon: longitude });
        setLocating(false);
      },
      (err) => {                                // FAILURE: GPS denied or timed out
        const msgs = {
          1: 'Location permission denied',
          2: 'Unable to determine position',
          3: 'Location request timed out',
        };
        setLocationError(msgs[err.code]);
        setLocating(false);
      },
      { timeout: 10000, maximumAge: 300000 }   // Timeout: 10s, cache GPS for 5 mins
    );
  }, []);

  // FUNCTION 2: Search cities by name (Open-Meteo geocoding API — free, no key needed)
  const searchCities = useCallback(async (query) => {
    if (!query || query.trim().length < 2) return []; // Don't search 1 character
    const r = await axios.get('https://geocoding-api.open-meteo.com/v1/search', {
      params: { name: query.trim(), count: 6, language: 'en' }
    });
    return r.data.results || [];
  }, []);

  // FUNCTION 3: Select a city from search results
  const selectCity = useCallback((result) => {
    setCity({
      name: result.name,
      state: result.admin1 || result.country || '',
      lat: result.latitude,
      lon: result.longitude,
    });
  }, []);

  return { city, locating, locationError, detectLocation, searchCities, selectCity };
}
```

---

## 🌐 `src/api/cityagent.js` — Data Fetching Layer

### Environment Variables (API Keys)

```js
const BACKEND_URL = import.meta.env.VITE_API_URL  || 'http://localhost:8000';
const WAQI_TOKEN  = import.meta.env.VITE_WAQI_TOKEN || 'demo';
const TOMTOM_KEY  = import.meta.env.VITE_TOMTOM_KEY || '';
// These read from a .env file. If not set, use safe defaults.
```

### `fetchAQI(lat, lon)` — Air Quality Index (LIVE)

```js
export async function fetchAQI(lat, lon) {
  const r = await axios.get('https://air-quality-api.open-meteo.com/v1/air-quality', {
    params: {
      latitude: lat, longitude: lon,
      current: 'pm10,pm2_5,nitrogen_dioxide,ozone,us_aqi', // What to fetch
      hourly: 'pm2_5', // For the trend sparkline chart
    }
  });

  const c = r.data.current; // Current air quality readings

  return {
    value: c.us_aqi,                           // The AQI number e.g. 64
    category: getUSAQICategory(c.us_aqi),      // "Good", "Moderate", "Unhealthy" etc.
    dominantPollutant: getDominantPollutant(c), // Which gas is the worst
    color: getAQIColor(c.us_aqi),              // Green / yellow / red based on value
    components: { pm25: c.pm2_5, pm10: c.pm10, no2: c.nitrogen_dioxide, ... },
    trend: last8HoursOfPM25,                   // Array of numbers for sparkline chart
    station: `Open-Meteo CAMS · ${lat}°N ${lon}°E`,
  };
  // If API fails → catch block returns safe fallback data
}
```

### `fetchWeather(lat, lon)` — Weather (LIVE)

```js
export async function fetchWeather(lat, lon) {
  const r = await axios.get('https://api.open-meteo.com/v1/forecast', { ... });
  const c = r.data.current;

  return {
    temperature:   Math.round(c.temperature_2m * 10) / 10,       // e.g. 16.9°C
    humidity:      c.relative_humidity_2m,                         // e.g. 64%
    pressure:      Math.round(c.surface_pressure * 0.750062),     // hPa → mmHg
    windSpeed:     Math.round((c.wind_speed_10m / 3.6) * 10) / 10, // km/h → m/s
    windDirection: degreesToCompass(c.wind_direction_10m),          // 225° → "SW"
    visibility:    Math.round((c.visibility / 1000) * 10) / 10,   // meters → km
    condition:     wmoToCondition(c.weather_code),                  // Code → "Overcast"
    icon:          wmoToIcon(c.weather_code),                       // Code → "☁️"
  };
}
```

### `fetchFlood(lat, lon)` — River Flood Data (LIVE)

```js
export async function fetchFlood(lat, lon) {
  const r = await axios.get('https://flood-api.open-meteo.com/v1/flood', { ... });
  const d = r.data.daily;

  const today   = d.river_discharge[7]; // Index 7 = today (7 past + today)
  const history = d.river_discharge.slice(0, 7); // Past 7 days

  return {
    floodRisk:           getFloodRisk(today, history), // "Low", "Medium", "High", "Critical"
    riverDischarge:      today,     // Cubic meters per second right now
    riverDischargeMax30d: max30d,   // 30-day max (for context)
    trend: [...history, today],     // Array for chart
  };
}
```

### `fetchTraffic(lat, lon)` — Traffic (LIVE with TomTom key, else MOCK)

```js
export async function fetchTraffic(lat, lon) {
  if (!TOMTOM_KEY) return trafficData; // No key → use mock data

  // Fetch BOTH flow data and incidents simultaneously
  const [flowRes, incRes] = await Promise.allSettled([
    axios.get('https://api.tomtom.com/traffic/services/4/flowSegmentData/...'),
    axios.get('https://api.tomtom.com/traffic/services/5/incidentDetails/...'),
  ]);

  const f = flowRes.value.data.flowSegmentData;
  const currentSpeed  = f.currentSpeed;   // Actual speed right now
  const freeFlowSpeed = f.freeFlowSpeed;  // Speed with zero traffic

  // Calculate congestion level from the speed ratio
  const ratio = currentSpeed / freeFlowSpeed;
  const congestionLevel =
    ratio >= 0.80 ? 'Free flow' :
    ratio >= 0.50 ? 'Moderate'  :
    ratio >= 0.25 ? 'Heavy'     : 'Standstill';

  return { vehiclesPerHour, congestionLevel, currentSpeed, hotspot, incidents, ... };
}
```

### Helper Functions

```js
// Convert wind degrees to compass direction
function degreesToCompass(deg) {
  const dirs = ['N','NE','E','SE','S','SW','W','NW'];
  return dirs[Math.round(((deg % 360) / 45)) % 8];
  // 0°=N, 90°=E, 180°=S, 270°=W, 225°=SW
}

// Convert AQI number to a text category
function getUSAQICategory(v) {
  if (v <= 50)  return 'Good';
  if (v <= 100) return 'Moderate';
  if (v <= 150) return 'Unhealthy for Sensitive Groups';
  if (v <= 200) return 'Unhealthy';
  if (v <= 300) return 'Very Unhealthy';
  return 'Hazardous';
}

// Find which pollutant is worst (compare each to its WHO safe limit)
function getDominantPollutant(c) {
  const ratios = {
    'PM2.5': c.pm2_5 / 15,  // WHO guideline is 15 µg/m³
    'PM10':  c.pm10  / 45,
    'NO2':   c.nitrogen_dioxide / 25,
    'O3':    c.ozone / 100,
    'SO2':   c.sulphur_dioxide / 40,
  };
  return Object.entries(ratios).sort((a, b) => b[1] - a[1])[0][0];
  // Sort by ratio (highest = most dangerous relative to safe level) → pick first
}

// Mock WebSocket — simulates real-time alerts every 15 seconds
export function connectAlertSocket(onMessage) {
  let idx = 0;
  const interval = setInterval(() => {
    onMessage(incomingAlertQueue[idx % incomingAlertQueue.length]); // Loop through queue
    idx++;
  }, 15000);
  return { close: () => clearInterval(interval) }; // Return a "close" method
}
```

---

## 🤖 `src/api/kimiChat.js` — The AI Brain

### Rate Limiter

```js
const MAX_PER_WINDOW = 10; // Max 10 messages per minute
const MIN_GAP_MS = 2000;   // Must wait 2 seconds between messages

export function checkRateLimit() {
  const now = Date.now();
  // Remove timestamps older than 1 minute
  while (_timestamps.length && now - _timestamps[0] > RATE_WINDOW_MS) _timestamps.shift();

  if (_timestamps.length >= MAX_PER_WINDOW)
    return `Rate limit reached. Wait Xs.`; // Block the request

  _timestamps.push(now); // Record this request's time
  return null;            // null = "you're allowed to send"
}
```

### System Prompt Builder

```js
export function buildCityContext(city, data, alerts, userCoords, userLocData) {
  // Builds a detailed "briefing document" for the AI, for example:
  //
  // "CityAgent AI — Dehradun live dashboard.
  //  AQI 64 (Moderate), PM2.5 32.5µg/m³, PM10 82µg/m³, dominant: PM2.5
  //  Weather: 16.9°C Overcast, humidity 64%, wind 1.3m/s SW, visibility 24.1km
  //  Flood: Low risk, river 0.34m³/s (30d max 5.93m³/s)
  //  Traffic: Moderate at Clock Tower, 42km/h (free-flow 60km/h)
  //  Alerts (3): [critical] PM2.5 spike in Rajpur Road | ...
  //  Rules: Only use the data above. Give 2-4 sentence answers."
  //
  // If user shared GPS → also include nearest zone and distance
}
```

### Streaming Chat (Word by Word)

```js
export async function* streamKimiChat(messages, systemPrompt) {
  // Send the full conversation + system briefing to Llama 3.1 AI
  const response = await fetch('/api/kimi', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${KIMI_API_KEY}`,
      'Accept': 'text/event-stream', // Tell server to stream the response
    },
    body: JSON.stringify({
      model: 'meta/llama-3.1-8b-instruct',
      messages: [{ role: 'system', content: systemPrompt }, ...messages],
      stream: true,      // ← "Send me words one at a time as they're generated"
      temperature: 0.3,  // Low = focused/factual (0 = robotic, 1 = creative)
      max_tokens: 400,
    }),
  });

  // Read the response stream chunk by chunk
  const reader = response.body.getReader();
  while (true) {
    const { done, value } = await reader.read(); // Read a chunk
    if (done) break;
    // Parse Server-Sent Events (SSE) format: "data: { choices: [{delta: {content: 'word'}}] }"
    for (const line of lines) {
      const token = JSON.parse(line.slice(6)).choices[0].delta.content;
      yield token; // ← Send each word back to ChatBot one at a time
    }
  }
}
```

> **Why streaming?** Instead of waiting 5 seconds for a full answer, you see words appear live — just like ChatGPT. Each `yield token` sends one word at a time to the UI.

---

## 🗂️ `src/components/Sidebar.jsx` — Left Navigation Bar

```jsx
export default function Sidebar({ activePage, onChangePage }) {
  const navItems = [
    { id: 'overview', icon: <LayoutDashboard size={18} /> }, // Dashboard grid icon
    { id: 'air',      icon: <Wind size={18} /> },            // Wind icon
    { id: 'traffic',  icon: <Car size={18} /> },             // Car icon
    { id: 'alerts',   icon: <Bell size={18} /> },            // Bell icon
    { id: 'pipeline', icon: <Activity size={18} /> },        // Heartbeat icon
  ];

  return (
    <aside className="w-[56px] h-full bg-bg-card border-r">
      {/* Logo at top — a bar chart SVG icon */}
      <div className="w-8 h-8 rounded bg-semantic-blue">
        <svg>...</svg>
      </div>

      <nav>
        {navItems.map((item) => (
          <div
            key={item.id}
            onClick={() => onChangePage(item.id)} // Click → change active page
            className={
              activePage === item.id
                ? 'bg-blue/15 text-blue'       // Active: highlighted blue
                : 'text-muted hover:bg-inner'  // Inactive: subtle hover
            }
          >
            {item.icon}
          </div>
        ))}
      </nav>

      {/* Settings gear at the bottom */}
      <div className="mt-auto"><Settings size={18} /></div>
    </aside>
  );
}
```

---

## 🔝 `src/components/Topbar.jsx` — Top Navigation Bar

### Key Features

```jsx
// 1. LIVE CLOCK — updates every second
useEffect(() => {
  const tick = () => setClock(new Date().toLocaleTimeString('en-IN', {...}));
  tick();
  const t = setInterval(tick, 1000); // Repeat every 1000ms = 1 second
  return () => clearInterval(t);     // Stop when component unmounts
}, []);

// 2. DEBOUNCED SEARCH — waits 350ms after you stop typing before fetching
const handleQueryChange = (e) => {
  const val = e.target.value;
  setQuery(val);
  clearTimeout(debounceRef.current); // Cancel any previous timer
  debounceRef.current = setTimeout(async () => {
    const res = await onSearchCities(val); // Fetch after 350ms of no typing
    setResults(res);
    setShowDropdown(res.length > 0);
  }, 350);
};

// 3. CLOSE DROPDOWN when clicking outside the search box
useEffect(() => {
  const handler = (e) => {
    if (searchRef.current && !searchRef.current.contains(e.target)) {
      setShowDropdown(false); // Click was outside → close dropdown
    }
  };
  document.addEventListener('mousedown', handler);
  return () => document.removeEventListener('mousedown', handler);
}, []);
```

---

## 💬 `src/components/ChatBot.jsx` — The Floating AI Assistant

### State

```jsx
const [open, setOpen]           = useState(false);  // Is chatbot window open?
const [isExpanded, setIsExpanded] = useState(false); // Is it full-screen?
const [messages, setMessages]   = useState([]);      // Full chat history
const [input, setInput]         = useState('');      // Text in the input box
const [streaming, setStreaming] = useState(false);   // Is AI currently typing?
const [userCoords, setUserCoords] = useState(null);  // User's GPS coordinates
```

### Location Detection Intent

```jsx
// Keywords that signal "user wants location-specific answer"
const LOCATION_KEYWORDS = ['my area', 'near me', 'my location', 'nearby', 'here', ...];

function hasLocationIntent(text) {
  const lower = text.toLowerCase();
  return LOCATION_KEYWORDS.some(kw => lower.includes(kw));
  // Returns true if any keyword is found in the message
}
```

### Sending a Message

```jsx
const sendMessage = useCallback(async (text) => {
  if (!text.trim() || streaming) return; // Don't send empty or while AI is typing

  const rateLimitMsg = checkRateLimit();
  if (rateLimitMsg) { setError(rateLimitMsg); return; } // Rate limit hit

  const userMsg = { role: 'user', content: text };
  const history = [...messages, userMsg]; // Add to conversation history

  // If user asked about "my area" and GPS not granted yet → show location card
  if (hasLocationIntent(text) && !userCoords) {
    setMessages([...history, { role: 'location-prompt', content: text }]);
    setPendingQuestion(text);
    return;
  }

  setMessages(history);
  await streamReply(history, userCoords, userLocData); // Send to AI
}, [...]);
```

### Streaming Reply (Word by Word)

```jsx
const streamReply = useCallback(async (history, coords, locData) => {
  setStreaming(true);
  setMessages(prev => [...prev, { role: 'assistant', content: '' }]); // Empty AI bubble

  const systemPrompt = buildCityContext(city, data, alerts, coords, locData);
  const gen = streamKimiChat(history, systemPrompt); // Start generator

  for await (const token of gen) {          // For each word received:
    setMessages(prev => {
      const updated = [...prev];
      updated[updated.length - 1] = {
        role: 'assistant',
        content: updated[updated.length - 1].content + token, // Append word
      };
      return updated;
    });
  }
  setStreaming(false);
}, [city, data, alerts]);
```

### Markdown Renderer

```jsx
function renderText(text) {
  // Splits text at **bold** and `code` markers
  const parts = text.split(/(\*\*[^*]+\*\*|`[^`]+`)/g);
  return parts.map((part, i) => {
    if (part.startsWith('**') && part.endsWith('**'))
      return <strong>{part.slice(2, -2)}</strong>;  // **text** → bold
    if (part.startsWith('`') && part.endsWith('`'))
      return <code>{part.slice(1, -1)}</code>;       // `code` → code block
    return part; // Normal text — render as-is
  });
}
```

---

## 🔄 How Everything Connects

```
index.html
    └── main.jsx  (starts React)
            └── App.jsx  (root component — the brain)
                    ├── useCity.js       → manages city selection & GPS
                    ├── cityagent.js     → fetches AQI, weather, flood, traffic
                    ├── Sidebar.jsx      → left nav, page switching
                    ├── Topbar.jsx       → search, clock, GPS button
                    ├── StatCard × 4    → summary numbers at the top
                    ├── MetricCard × 4  → detailed metrics with sparklines
                    ├── AnomalyChart    → timeline anomaly chart
                    ├── AQIGauge        → circular air quality meter
                    ├── AlertFeed       → live scrolling alert list
                    ├── MapPanel        → interactive Leaflet map
                    ├── Pipeline        → data pipeline status board
                    └── ChatBot.jsx
                            └── kimiChat.js → Llama 3.1 AI (NVIDIA NIM)
```

---

## ✅ Key Concepts Cheat Sheet

| Concept | What It Means |
|---|---|
| **useState** | A variable that triggers a re-render when changed |
| **useEffect** | Code that runs after render, or when dependencies change |
| **useCallback** | A function that is not re-created on every render |
| **Promise.all** | Run multiple async operations simultaneously |
| **async/await** | Write asynchronous code that reads like normal code |
| **yield** | Send a value from a generator one at a time (used for streaming AI) |
| **Tailwind CSS** | Utility classes like `flex`, `p-4`, `text-blue` applied in JSX |
| **props** | Data passed from parent to child component (like function arguments) |
| **component** | A reusable piece of UI — like a Lego block |
| **hook** | A reusable function for state/logic that starts with `use` |
| **API** | A URL you fetch data from (Open-Meteo, TomTom, etc.) |
| **SSE/Streaming** | AI sends words one at a time instead of waiting for the full answer |
| **debounce** | Wait Xms after last keypress before firing an action |
| **mock data** | Fake but realistic data used when a real API isn't available |
| **WebSocket** | A live connection that pushes data to you without you asking |
