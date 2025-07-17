# Smart Port Command Center Development Guide

A comprehensive step-by-step guide for building a production-ready Smart Port Command Center dashboard on Ubuntu 24.04 with React 18+, TypeScript 5.x, and modern maritime integration patterns.

## Complete Development Environment Setup

### System Requirements and Initial Setup

**Install Node.js v20.17.0 using Node Version Manager:**
```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc

# Install and use Node.js v20.17.0
nvm install 20.17.0
nvm use 20.17.0
nvm alias default 20.17.0

# Verify installation
node --version  # Should output v20.17.0
npm --version   # Should output 10.8.2 or later
```

**Install essential Ubuntu packages:**
```bash
sudo apt update
sudo apt install -y git curl wget build-essential python3 python3-pip postgresql postgresql-contrib redis-server
```

### Project Initialization with Latest Vite

**Create the Smart Port Command Center project:**
```bash
# Create project with Vite and React-SWC
npm create vite@latest smart-port-command-center -- --template react-swc-ts
cd smart-port-command-center

# Install core dependencies
npm install react@^18.2.0 react-dom@^18.2.0 react-router-dom@^6.24.1

# Install UI and visualization libraries
npm install @mui/material @emotion/react @emotion/styled @mui/icons-material
npm install recharts react-window react-virtualized-auto-sizer react-leaflet leaflet
npm install socket.io-client @reduxjs/toolkit react-redux zustand

# Install development dependencies
npm install -D @types/react@^18.2.66 @types/react-dom@^18.2.22 @types/leaflet
npm install -D @vitejs/plugin-react-swc@^3.7.0 vite@^5.3.4
npm install -D typescript@^5.5.3 @typescript-eslint/eslint-plugin@^8.0.0 @typescript-eslint/parser@^8.0.0
npm install -D eslint@^9.7.0 eslint-plugin-react@^7.34.3 eslint-plugin-react-hooks@^4.6.2
npm install -D vitest@^2.0.5 @vitest/ui@^2.0.5 @testing-library/react@^16.0.0 @testing-library/jest-dom@^6.4.6
npm install -D @playwright/test@^1.45.0 jsdom@^24.1.1
```

### Production-Ready Vite Configuration

**Create `vite.config.ts`:**
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import { resolve } from 'path'

export default defineConfig({
  plugins: [
    react({
      // Remove styled-components plugin - not needed for Material-UI
      devTarget: 'es2022',
    }),
  ],
  
  build: {
    target: 'es2022',
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
          ui: ['@mui/material', '@emotion/react', '@emotion/styled'],
          charts: ['recharts', 'react-window'],
          maps: ['react-leaflet', 'leaflet'],
        },
      },
    },
    sourcemap: process.env.NODE_ENV === 'development',
  },
  
  server: {
    port: 3000,
    host: true,
    hmr: { overlay: false },
  },
  
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@hooks': resolve(__dirname, 'src/hooks'),
      '@utils': resolve(__dirname, 'src/utils'),
      '@types': resolve(__dirname, 'src/types'),
      '@data': resolve(__dirname, 'src/data'),
    },
  },
  
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    css: true,
  },
})
```

### TypeScript 5.x Strict Configuration

**Create `tsconfig.json`:**
```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2023", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "useDefineForClassFields": true,
    
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noPropertyAccessFromIndexSignature": true,
    "allowUnreachableCode": false,
    "allowUnusedLabels": false,
    
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "verbatimModuleSyntax": true,
    "skipLibCheck": true,
    
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"],
      "@utils/*": ["src/utils/*"],
      "@types/*": ["src/types/*"],
      "@data/*": ["src/data/*"]
    }
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "vite.config.ts"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.*",
    "**/*.spec.*"
  ]
}
```

### ESLint 9+ Flat Configuration

**Create `eslint.config.js`:**
```javascript
import js from '@eslint/js'
import tseslint from 'typescript-eslint'
import reactPlugin from 'eslint-plugin-react'
import reactHooksPlugin from 'eslint-plugin-react-hooks'
import reactRefreshPlugin from 'eslint-plugin-react-refresh'
import globals from 'globals'

export default tseslint.config(
  {
    ignores: ['dist/**', 'build/**', 'node_modules/**', 'coverage/**'],
  },
  
  js.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  reactPlugin.configs.flat.recommended,
  reactPlugin.configs.flat['jsx-runtime'],
  
  {
    files: ['**/*.{js,jsx,ts,tsx}'],
    languageOptions: {
      ecmaVersion: 'latest',
      sourceType: 'module',
      globals: {
        ...globals.browser,
        ...globals.es2022,
      },
      parserOptions: {
        ecmaFeatures: { jsx: true },
        project: './tsconfig.json',
      },
    },
    
    plugins: {
      'react-hooks': reactHooksPlugin,
      'react-refresh': reactRefreshPlugin,
    },
    
    rules: {
      ...reactHooksPlugin.configs.recommended.rules,
      'react-refresh/only-export-components': ['warn', { allowConstantExport: true }],
      'react/react-in-jsx-scope': 'off',
      'react/jsx-uses-react': 'off',
      'react/prop-types': 'off',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/prefer-nullish-coalescing': 'error',
      '@typescript-eslint/prefer-optional-chain': 'error',
    },
    
    settings: {
      react: { version: 'detect' },
    },
  }
)
```

## React 18+ Implementation with Modern Patterns

### Advanced Type Definitions for Maritime Systems

**Create `src/types/maritime.ts`:**
```typescript
// AIS Vessel Data Types
export interface AISVesselData {
  mmsi: number
  timestamp: string
  position: {
    longitude: number
    latitude: number
  }
  navigation: {
    courseOverGround: number
    speedOverGround: number
    heading: number
    rateOfTurn: number
    navigationStatus: NavigationStatus
  }
  vesselInfo: {
    imo: number
    name: string
    callsign: string
    type: VesselType
    dimensions: {
      length: number
      width: number
      draft: number
    }
  }
  destination: string
  eta: string
}

export enum NavigationStatus {
  UnderWay = 0,
  AtAnchor = 1,
  NotUnderCommand = 2,
  RestrictedManeuverability = 3,
  ConstrainedByDraft = 4,
  Moored = 5,
  Aground = 6,
  EngagedInFishing = 7,
  UnderWayUsingSail = 8,
  Reserved = 9,
  Reserved2 = 10,
  Reserved3 = 11,
  Reserved4 = 12,
  Reserved5 = 13,
  AISSearchAndRescue = 14,
  Undefined = 15,
}

export enum VesselType {
  Fishing = 30,
  Tug = 52,
  Passenger = 60,
  Cargo = 70,
  Tanker = 80,
  Pleasure = 37,
}

// Port Operations Types
export interface PortOperationMetrics {
  timestamp: string
  terminalId: string
  vesselProductivity: number
  berthUtilization: number
  yardUtilization: number
  craneUtilization: number
  truckTurnTime: number
  gateMoves: number
  dwellTime: number
  vesselWaitingTime: number
}

// IoT Sensor Data Types
export interface CraneMonitoringData {
  craneId: string
  timestamp: string
  location: {
    latitude: number
    longitude: number
    berth: string
  }
  operational: {
    loadWeight: number
    boomAngle: number
    hookHeight: number
    trolleyPosition: number
    spreaderStatus: 'engaged' | 'disengaged'
  }
  sensors: {
    motorTemperature: number
    hydraulicPressure: number
    vibrationLevel: number
    powerConsumption: number
    windSpeed: number
  }
  maintenance: {
    operatingHours: number
    lastMaintenance: string
    nextMaintenance: string
    healthScore: number
  }
  alerts: Array<{
    type: 'warning' | 'critical' | 'info'
    message: string
    threshold: number
    currentValue: number
  }>
}

// Weather Data Types
export interface WeatherData {
  location: {
    latitude: number
    longitude: number
  }
  timestamp: string
  marineConditions: {
    waveHeight: number
    waveDirection: number
    wavePeriod: number
    swellHeight: number
    swellDirection: number
    swellPeriod: number
    windWaveHeight: number
    windWaveDirection: number
    windWavePeriod: number
  }
  oceanData: {
    seaSurfaceTemperature: number
    seaLevelHeight: number
    oceanCurrentVelocity: number
    oceanCurrentDirection: number
  }
  atmospheric: {
    windSpeed: number
    windDirection: number
    airPressure: number
    visibility: number
    precipitation: number
  }
}

// Dashboard State Types
export interface DashboardState {
  vessels: AISVesselData[]
  portMetrics: PortOperationMetrics
  equipment: CraneMonitoringData[]
  weather: WeatherData
  alerts: Alert[]
  isLoading: boolean
  error: string | null
  lastUpdate: string
}

export interface Alert {
  id: string
  type: 'info' | 'warning' | 'critical'
  title: string
  message: string
  timestamp: string
  acknowledged: boolean
  source: string
}
```

### Concurrent Features with React 18+

**Create `src/hooks/useConcurrentFeatures.ts`:**
```typescript
import { 
  useTransition, 
  useDeferredValue, 
  useState, 
  useCallback, 
  useMemo,
  startTransition 
} from 'react'

export function useOptimizedSearch<T>(
  data: T[],
  searchFn: (item: T, query: string) => boolean,
  debounceMs = 300
) {
  const [query, setQuery] = useState('')
  const [isPending, startTransition] = useTransition()
  const deferredQuery = useDeferredValue(query)
  
  const filteredData = useMemo(() => {
    if (!deferredQuery) return data
    return data.filter(item => searchFn(item, deferredQuery))
  }, [data, deferredQuery, searchFn])
  
  const handleSearch = useCallback((value: string) => {
    startTransition(() => {
      setQuery(value)
    })
  }, [])
  
  return {
    query,
    filteredData,
    handleSearch,
    isPending,
    isStale: query !== deferredQuery
  }
}

export function useOptimizedDataUpdates<T>(
  initialData: T[],
  maxRetentionTime = 300000 // 5 minutes
) {
  const [data, setData] = useState<T[]>(initialData)
  const [, startTransition] = useTransition()
  
  const addData = useCallback((newData: T & { timestamp: number }) => {
    startTransition(() => {
      const now = Date.now()
      const cutoffTime = now - maxRetentionTime
      
      setData(prev => {
        const updatedData = [...prev, newData]
        return updatedData.filter(item => 
          (item as any).timestamp > cutoffTime
        )
      })
    })
  }, [maxRetentionTime])
  
  const deferredData = useDeferredValue(data)
  
  return {
    data: deferredData,
    addData,
    count: data.length
  }
}
```

### Real-time WebSocket Integration

**Create `src/hooks/useWebSocket.ts`:**
```typescript
import { useState, useEffect, useCallback, useRef } from 'react'
import { AISVesselData, PortOperationMetrics, CraneMonitoringData, WeatherData } from '@types/maritime'

interface WebSocketMessage {
  type: 'vessel_update' | 'port_metrics' | 'equipment_data' | 'weather_data' | 'alert'
  data: AISVesselData | PortOperationMetrics | CraneMonitoringData | WeatherData | any
  timestamp: string
}

export function useMaritimeWebSocket(url: string) {
  const [isConnected, setIsConnected] = useState(false)
  const [lastMessage, setLastMessage] = useState<WebSocketMessage | null>(null)
  const [connectionStatus, setConnectionStatus] = useState<'connecting' | 'connected' | 'disconnected'>('disconnected')
  const wsRef = useRef<WebSocket | null>(null)
  const reconnectTimeoutRef = useRef<NodeJS.Timeout | null>(null)
  
  const connect = useCallback(() => {
    if (wsRef.current?.readyState === WebSocket.OPEN) return
    
    setConnectionStatus('connecting')
    const ws = new WebSocket(url)
    
    ws.onopen = () => {
      setIsConnected(true)
      setConnectionStatus('connected')
      console.log('Maritime WebSocket connected')
      
      // Clear reconnect timeout
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current)
      }
    }
    
    ws.onmessage = (event) => {
      try {
        const message: WebSocketMessage = JSON.parse(event.data)
        setLastMessage(message)
      } catch (error) {
        console.error('Failed to parse WebSocket message:', error)
      }
    }
    
    ws.onclose = () => {
      setIsConnected(false)
      setConnectionStatus('disconnected')
      console.log('Maritime WebSocket disconnected')
      
      // Attempt to reconnect after 3 seconds
      reconnectTimeoutRef.current = setTimeout(() => {
        connect()
      }, 3000)
    }
    
    ws.onerror = (error) => {
      console.error('Maritime WebSocket error:', error)
    }
    
    wsRef.current = ws
  }, [url])
  
  const disconnect = useCallback(() => {
    if (reconnectTimeoutRef.current) {
      clearTimeout(reconnectTimeoutRef.current)
    }
    
    if (wsRef.current) {
      wsRef.current.close()
      wsRef.current = null
    }
    
    setIsConnected(false)
    setConnectionStatus('disconnected')
  }, [])
  
  const sendMessage = useCallback((message: any) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(message))
    }
  }, [])
  
  useEffect(() => {
    connect()
    
    return () => {
      disconnect()
    }
  }, [connect, disconnect])
  
  return {
    isConnected,
    lastMessage,
    connectionStatus,
    sendMessage,
    connect,
    disconnect
  }
}
```

## Smart Port Command Center Dashboard Components

### Interactive Port Map with Vessel Tracking

**Create `src/components/PortMap.tsx`:**
```typescript
import React, { useMemo, useCallback } from 'react'
import { MapContainer, TileLayer, Marker, Popup, useMap } from 'react-leaflet'
import { Icon, LatLngBounds } from 'leaflet'
import { AISVesselData, VesselType } from '@types/maritime'
import 'leaflet/dist/leaflet.css'

interface PortMapProps {
  vessels: AISVesselData[]
  center: [number, number]
  zoom: number
  portBounds?: LatLngBounds
}

const getVesselIcon = (vesselType: VesselType, status: number) => {
  const iconUrl = `/icons/vessel-${vesselType}-${status}.png`
  return new Icon({
    iconUrl,
    iconSize: [24, 24],
    iconAnchor: [12, 12],
    popupAnchor: [0, -12]
  })
}

const VesselMarker: React.FC<{ vessel: AISVesselData }> = ({ vessel }) => {
  const icon = useMemo(() => 
    getVesselIcon(vessel.vesselInfo.type, vessel.navigation.navigationStatus),
    [vessel.vesselInfo.type, vessel.navigation.navigationStatus]
  )
  
  return (
    <Marker 
      position={[vessel.position.latitude, vessel.position.longitude]}
      icon={icon}
    >
      <Popup>
        <div className="vessel-popup">
          <h3>{vessel.vesselInfo.name}</h3>
          <p><strong>MMSI:</strong> {vessel.mmsi}</p>
          <p><strong>Type:</strong> {VesselType[vessel.vesselInfo.type]}</p>
          <p><strong>Speed:</strong> {vessel.navigation.speedOverGround.toFixed(1)} knots</p>
          <p><strong>Course:</strong> {vessel.navigation.courseOverGround.toFixed(1)}°</p>
          <p><strong>Destination:</strong> {vessel.destination}</p>
          <p><strong>ETA:</strong> {vessel.eta}</p>
          <p><strong>Last Update:</strong> {new Date(vessel.timestamp).toLocaleString()}</p>
        </div>
      </Popup>
    </Marker>
  )
}

const PortMap: React.FC<PortMapProps> = ({ vessels, center, zoom, portBounds }) => {
  const visibleVessels = useMemo(() => {
    return vessels.filter(vessel => {
      if (!portBounds) return true
      return portBounds.contains([vessel.position.latitude, vessel.position.longitude])
    })
  }, [vessels, portBounds])
  
  return (
    <div className="port-map-container" style={{ height: '600px', width: '100%' }}>
      <MapContainer
        center={center}
        zoom={zoom}
        scrollWheelZoom={true}
        style={{ height: '100%', width: '100%' }}
      >
        <TileLayer
          attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        />
        
        {visibleVessels.map(vessel => (
          <VesselMarker key={vessel.mmsi} vessel={vessel} />
        ))}
      </MapContainer>
    </div>
  )
}

export default PortMap
```

### Real-time Analytics Dashboard

**Create `src/components/AnalyticsDashboard.tsx`:**
```typescript
import React, { useMemo } from 'react'
import { 
  LineChart, 
  Line, 
  XAxis, 
  YAxis, 
  CartesianGrid, 
  Tooltip, 
  ResponsiveContainer,
  BarChart,
  Bar,
  PieChart,
  Pie,
  Cell
} from 'recharts'
import { Grid, Card, CardContent, Typography, Box } from '@mui/material'
import { PortOperationMetrics } from '@types/maritime'

interface AnalyticsDashboardProps {
  portMetrics: PortOperationMetrics[]
  realtimeData: any[]
}

const KPICard: React.FC<{ title: string; value: number | string; unit?: string; trend?: number }> = ({ 
  title, 
  value, 
  unit, 
  trend 
}) => (
  <Card>
    <CardContent>
      <Typography variant="h6" component="h2">
        {title}
      </Typography>
      <Typography variant="h4" component="p">
        {typeof value === 'number' ? value.toFixed(1) : value}
        {unit && <span style={{ fontSize: '0.7em', marginLeft: '4px' }}>{unit}</span>}
      </Typography>
      {trend !== undefined && (
        <Typography 
          variant="body2" 
          color={trend > 0 ? 'success.main' : 'error.main'}
        >
          {trend > 0 ? '↑' : '↓'} {Math.abs(trend).toFixed(1)}%
        </Typography>
      )}
    </CardContent>
  </Card>
)

const AnalyticsDashboard: React.FC<AnalyticsDashboardProps> = ({ portMetrics, realtimeData }) => {
  const latestMetrics = useMemo(() => {
    return portMetrics[portMetrics.length - 1]
  }, [portMetrics])
  
  const chartData = useMemo(() => {
    return portMetrics.map(metric => ({
      timestamp: new Date(metric.timestamp).toLocaleTimeString(),
      vesselProductivity: metric.vesselProductivity,
      berthUtilization: metric.berthUtilization * 100,
      yardUtilization: metric.yardUtilization * 100,
      craneUtilization: metric.craneUtilization * 100,
      truckTurnTime: metric.truckTurnTime,
      gateMoves: metric.gateMoves,
      dwellTime: metric.dwellTime,
    }))
  }, [portMetrics])
  
  const utilizationData = useMemo(() => {
    if (!latestMetrics) return []
    
    return [
      { name: 'Berth', value: latestMetrics.berthUtilization * 100 },
      { name: 'Yard', value: latestMetrics.yardUtilization * 100 },
      { name: 'Crane', value: latestMetrics.craneUtilization * 100 },
    ]
  }, [latestMetrics])
  
  const COLORS = ['#0088FE', '#00C49F', '#FFBB28']
  
  if (!latestMetrics) {
    return <div>Loading analytics...</div>
  }
  
  return (
    <Box sx={{ flexGrow: 1 }}>
      <Grid container spacing={3}>
        {/* KPI Cards */}
        <Grid item xs={12} sm={6} md={3}>
          <KPICard 
            title="Vessel Productivity" 
            value={latestMetrics.vesselProductivity} 
            unit="moves/hr"
            trend={2.5}
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <KPICard 
            title="Berth Utilization" 
            value={latestMetrics.berthUtilization * 100} 
            unit="%"
            trend={-1.2}
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <KPICard 
            title="Truck Turn Time" 
            value={latestMetrics.truckTurnTime} 
            unit="min"
            trend={-3.8}
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <KPICard 
            title="Gate Moves" 
            value={latestMetrics.gateMoves} 
            unit="daily"
            trend={5.2}
          />
        </Grid>
        
        {/* Time Series Chart */}
        <Grid item xs={12} md={8}>
          <Card>
            <CardContent>
              <Typography variant="h6" gutterBottom>
                Port Performance Trends
              </Typography>
              <ResponsiveContainer width="100%" height={300}>
                <LineChart data={chartData}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="timestamp" />
                  <YAxis />
                  <Tooltip />
                  <Line 
                    type="monotone" 
                    dataKey="vesselProductivity" 
                    stroke="#8884d8" 
                    name="Vessel Productivity"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="berthUtilization" 
                    stroke="#82ca9d" 
                    name="Berth Utilization %"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="truckTurnTime" 
                    stroke="#ffc658" 
                    name="Truck Turn Time"
                  />
                </LineChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </Grid>
        
        {/* Utilization Pie Chart */}
        <Grid item xs={12} md={4}>
          <Card>
            <CardContent>
              <Typography variant="h6" gutterBottom>
                Current Utilization
              </Typography>
              <ResponsiveContainer width="100%" height={300}>
                <PieChart>
                  <Pie
                    data={utilizationData}
                    cx="50%"
                    cy="50%"
                    labelLine={false}
                    label={({ name, value }) => `${name}: ${value.toFixed(1)}%`}
                    outerRadius={80}
                    fill="#8884d8"
                    dataKey="value"
                  >
                    {utilizationData.map((entry, index) => (
                      <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                    ))}
                  </Pie>
                  <Tooltip />
                </PieChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </Grid>
        
        {/* Equipment Status Bar Chart */}
        <Grid item xs={12}>
          <Card>
            <CardContent>
              <Typography variant="h6" gutterBottom>
                Equipment Performance
              </Typography>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={chartData.slice(-10)}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="timestamp" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="craneUtilization" fill="#8884d8" name="Crane Utilization %" />
                  <Bar dataKey="gateMoves" fill="#82ca9d" name="Gate Moves" />
                </BarChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </Grid>
      </Grid>
    </Box>
  )
}

export default AnalyticsDashboard
```

### Equipment Monitoring Dashboard

**Create `src/components/EquipmentMonitoring.tsx`:**
```typescript
import React, { useMemo } from 'react'
import { 
  Grid, 
  Card, 
  CardContent, 
  Typography, 
  Box, 
  Alert, 
  Chip, 
  LinearProgress,
  List,
  ListItem,
  ListItemText,
  ListItemIcon
} from '@mui/material'
import { 
  Warning, 
  CheckCircle, 
  Error, 
  Build, 
  Timeline 
} from '@mui/icons-material'
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts'
import { CraneMonitoringData } from '@types/maritime'

interface EquipmentMonitoringProps {
  equipmentData: CraneMonitoringData[]
  historicalData: any[]
}

const HealthScoreIndicator: React.FC<{ score: number }> = ({ score }) => {
  const getColor = (score: number) => {
    if (score >= 0.8) return 'success'
    if (score >= 0.6) return 'warning'
    return 'error'
  }
  
  return (
    <Box sx={{ width: '100%' }}>
      <Box sx={{ display: 'flex', alignItems: 'center' }}>
        <Box sx={{ width: '100%', mr: 1 }}>
          <LinearProgress 
            variant="determinate" 
            value={score * 100} 
            color={getColor(score)}
          />
        </Box>
        <Box sx={{ minWidth: 35 }}>
          <Typography variant="body2" color="text.secondary">
            {`${Math.round(score * 100)}%`}
          </Typography>
        </Box>
      </Box>
    </Box>
  )
}

const EquipmentCard: React.FC<{ equipment: CraneMonitoringData }> = ({ equipment }) => {
  const statusColor = useMemo(() => {
    if (equipment.alerts.some(alert => alert.type === 'critical')) return 'error'
    if (equipment.alerts.some(alert => alert.type === 'warning')) return 'warning'
    return 'success'
  }, [equipment.alerts])
  
  const getStatusIcon = () => {
    switch (statusColor) {
      case 'error': return <Error color="error" />
      case 'warning': return <Warning color="warning" />
      default: return <CheckCircle color="success" />
    }
  }
  
  return (
    <Card>
      <CardContent>
        <Box sx={{ display: 'flex', alignItems: 'center', mb: 2 }}>
          {getStatusIcon()}
          <Typography variant="h6" sx={{ ml: 1 }}>
            {equipment.craneId}
          </Typography>
          <Chip 
            label={equipment.location.berth}
            size="small"
            sx={{ ml: 'auto' }}
          />
        </Box>
        
        <Typography variant="body2" color="text.secondary" gutterBottom>
          Health Score
        </Typography>
        <HealthScoreIndicator score={equipment.maintenance.healthScore} />
        
        <Grid container spacing={2} sx={{ mt: 2 }}>
          <Grid item xs={6}>
            <Typography variant="caption">Load Weight</Typography>
            <Typography variant="h6">{equipment.operational.loadWeight.toFixed(1)} tons</Typography>
          </Grid>
          <Grid item xs={6}>
            <Typography variant="caption">Power Consumption</Typography>
            <Typography variant="h6">{equipment.sensors.powerConsumption.toFixed(1)} kW</Typography>
          </Grid>
          <Grid item xs={6}>
            <Typography variant="caption">Motor Temperature</Typography>
            <Typography variant="h6">{equipment.sensors.motorTemperature.toFixed(1)}°C</Typography>
          </Grid>
          <Grid item xs={6}>
            <Typography variant="caption">Operating Hours</Typography>
            <Typography variant="h6">{equipment.maintenance.operatingHours.toFixed(1)} hrs</Typography>
          </Grid>
        </Grid>
        
        {equipment.alerts.length > 0 && (
          <Box sx={{ mt: 2 }}>
            <Typography variant="subtitle2" gutterBottom>
              Active Alerts
            </Typography>
            {equipment.alerts.map((alert, index) => (
              <Alert 
                key={index} 
                severity={alert.type === 'critical' ? 'error' : alert.type}
                sx={{ mb: 1 }}
              >
                {alert.message}
              </Alert>
            ))}
          </Box>
        )}
      </CardContent>
    </Card>
  )
}

const EquipmentMonitoring: React.FC<EquipmentMonitoringProps> = ({ 
  equipmentData, 
  historicalData 
}) => {
  const summaryStats = useMemo(() => {
    const totalEquipment = equipmentData.length
    const healthyEquipment = equipmentData.filter(eq => eq.maintenance.healthScore > 0.8).length
    const warningEquipment = equipmentData.filter(eq => 
      eq.maintenance.healthScore <= 0.8 && eq.maintenance.healthScore > 0.6
    ).length
    const criticalEquipment = equipmentData.filter(eq => eq.maintenance.healthScore <= 0.6).length
    
    return {
      total: totalEquipment,
      healthy: healthyEquipment,
      warning: warningEquipment,
      critical: criticalEquipment
    }
  }, [equipmentData])
  
  return (
    <Box sx={{ flexGrow: 1 }}>
      <Grid container spacing={3}>
        {/* Summary Cards */}
        <Grid item xs={12} sm={6} md={3}>
          <Card>
            <CardContent>
              <Typography variant="h6">Total Equipment</Typography>
              <Typography variant="h4">{summaryStats.total}</Typography>
            </CardContent>
          </Card>
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <Card>
            <CardContent>
              <Typography variant="h6">Healthy</Typography>
              <Typography variant="h4" color="success.main">
                {summaryStats.healthy}
              </Typography>
            </CardContent>
          </Card>
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <Card>
            <CardContent>
              <Typography variant="h6">Warning</Typography>
              <Typography variant="h4" color="warning.main">
                {summaryStats.warning}
              </Typography>
            </CardContent>
          </Card>
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <Card>
            <CardContent>
              <Typography variant="h6">Critical</Typography>
              <Typography variant="h4" color="error.main">
                {summaryStats.critical}
              </Typography>
            </CardContent>
          </Card>
        </Grid>
        
        {/* Equipment Cards */}
        {equipmentData.map(equipment => (
          <Grid item xs={12} md={6} lg={4} key={equipment.craneId}>
            <EquipmentCard equipment={equipment} />
          </Grid>
        ))}
        
        {/* Historical Performance Chart */}
        <Grid item xs={12}>
          <Card>
            <CardContent>
              <Typography variant="h6" gutterBottom>
                Equipment Performance History
              </Typography>
              <ResponsiveContainer width="100%" height={300}>
                <LineChart data={historicalData}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="timestamp" />
                  <YAxis />
                  <Tooltip />
                  <Line 
                    type="monotone" 
                    dataKey="averageHealthScore" 
                    stroke="#8884d8" 
                    name="Average Health Score"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="powerConsumption" 
                    stroke="#82ca9d" 
                    name="Power Consumption"
                  />
                </LineChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </Grid>
      </Grid>
    </Box>
  )
}

export default EquipmentMonitoring
```

## Maritime Data Sources and Sample Data Generation

### Comprehensive Data Generation System

**Create `src/data/maritimeDataGenerator.ts`:**
```typescript
import { AISVesselData, VesselType, NavigationStatus, PortOperationMetrics, CraneMonitoringData, WeatherData } from '@types/maritime'

export class MaritimeDataGenerator {
  private vesselNames = [
    'MAERSK EDINBURGH', 'EVER GIVEN', 'MSC OSCAR', 'OOCL HONG KONG',
    'COSCO SHIPPING UNIVERSE', 'MADRID MAERSK', 'HAPAG EXPRESS',
    'CMA CGM MARCO POLO', 'ATLANTIC SAIL', 'PACIFIC PIONEER'
  ]
  
  private portCoordinates = {
    'LOS_ANGELES': { lat: 33.7362, lng: -118.2632 },
    'SINGAPORE': { lat: 1.2897, lng: 103.8517 },
    'HAMBURG': { lat: 53.5511, lng: 9.9937 },
    'ROTTERDAM': { lat: 51.8985, lng: 4.1250 },
    'SHANGHAI': { lat: 31.2304, lng: 121.4737 }
  }
  
  generateAISData(count: number = 50): AISVesselData[] {
    const vessels: AISVesselData[] = []
    
    for (let i = 0; i < count; i++) {
      const port = Object.values(this.portCoordinates)[i % Object.values(this.portCoordinates).length]
      
      const vessel: AISVesselData = {
        mmsi: 200000000 + Math.floor(Math.random() * 99999999),
        timestamp: new Date().toISOString(),
        position: {
          longitude: port.lng + (Math.random() - 0.5) * 0.2,
          latitude: port.lat + (Math.random() - 0.5) * 0.2
        },
        navigation: {
          courseOverGround: Math.random() * 360,
          speedOverGround: Math.random() * 25,
          heading: Math.floor(Math.random() * 360),
          rateOfTurn: Math.floor(Math.random() * 255) - 128,
          navigationStatus: Math.floor(Math.random() * 16) as NavigationStatus
        },
        vesselInfo: {
          imo: 9000000 + Math.floor(Math.random() * 999999),
          name: this.vesselNames[i % this.vesselNames.length],
          callsign: `ABC${i.toString().padStart(3, '0')}`,
          type: Object.values(VesselType)[Math.floor(Math.random() * Object.values(VesselType).length)] as VesselType,
          dimensions: {
            length: 150 + Math.random() * 250,
            width: 20 + Math.random() * 30,
            draft: 5 + Math.random() * 10
          }
        },
        destination: Object.keys(this.portCoordinates)[Math.floor(Math.random() * Object.keys(this.portCoordinates).length)],
        eta: new Date(Date.now() + Math.random() * 7 * 24 * 60 * 60 * 1000).toISOString().slice(5, 16).replace('T', ' ')
      }
      
      vessels.push(vessel)
    }
    
    return vessels
  }
  
  generatePortMetrics(): PortOperationMetrics {
    return {
      timestamp: new Date().toISOString(),
      terminalId: 'TERM-001',
      vesselProductivity: 40 + Math.random() * 20,
      berthUtilization: 0.7 + Math.random() * 0.2,
      yardUtilization: 0.6 + Math.random() * 0.2,
      craneUtilization: 0.7 + Math.random() * 0.2,
      truckTurnTime: 20 + Math.random() * 25,
      gateMoves: 2000 + Math.random() * 1000,
      dwellTime: 2 + Math.random() * 3,
      vesselWaitingTime: 1 + Math.random() * 3
    }
  }
  
  generateEquipmentData(count: number = 10): CraneMonitoringData[] {
    const equipment: CraneMonitoringData[] = []
    
    for (let i = 0; i < count; i++) {
      const healthScore = 0.5 + Math.random() * 0.5
      const alerts = []
      
      // Generate alerts based on health score
      if (healthScore < 0.7) {
        alerts.push({
          type: 'critical' as const,
          message: 'Equipment requires immediate maintenance',
          threshold: 0.7,
          currentValue: healthScore
        })
      } else if (healthScore < 0.85) {
        alerts.push({
          type: 'warning' as const,
          message: 'Schedule maintenance soon',
          threshold: 0.85,
          currentValue: healthScore
        })
      }
      
      equipment.push({
        craneId: `QC-${(i + 1).toString().padStart(3, '0')}`,
        timestamp: new Date().toISOString(),
        location: {
          latitude: 33.7362 + (Math.random() - 0.5) * 0.01,
          longitude: -118.2632 + (Math.random() - 0.5) * 0.01,
          berth: `B-${i + 1}`
        },
        operational: {
          loadWeight: Math.random() * 50,
          boomAngle: 20 + Math.random() * 40,
          hookHeight: 10 + Math.random() * 40,
          trolleyPosition: Math.random() * 30,
          spreaderStatus: Math.random() > 0.5 ? 'engaged' : 'disengaged'
        },
        sensors: {
          motorTemperature: 60 + Math.random() * 30,
          hydraulicPressure: 150 + Math.random() * 50,
          vibrationLevel: Math.random() * 5,
          powerConsumption: 100 + Math.random() * 100,
          windSpeed: Math.random() * 20
        },
        maintenance: {
          operatingHours: Math.random() * 2000,
          lastMaintenance: new Date(Date.now() - Math.random() * 30 * 24 * 60 * 60 * 1000).toISOString(),
          nextMaintenance: new Date(Date.now() + Math.random() * 30 * 24 * 60 * 60 * 1000).toISOString(),
          healthScore
        },
        alerts
      })
    }
    
    return equipment
  }
  
  generateWeatherData(): WeatherData {
    return {
      location: {
        latitude: 33.7362,
        longitude: -118.2632
      },
      timestamp: new Date().toISOString(),
      marineConditions: {
        waveHeight: 0.5 + Math.random() * 2.5,
        waveDirection: Math.random() * 360,
        wavePeriod: 5 + Math.random() * 10,
        swellHeight: 0.3 + Math.random() * 1.7,
        swellDirection: Math.random() * 360,
        swellPeriod: 8 + Math.random() * 12,
        windWaveHeight: 0.2 + Math.random() * 1.3,
        windWaveDirection: Math.random() * 360,
        windWavePeriod: 4 + Math.random() * 8
      },
      oceanData: {
        seaSurfaceTemperature: 15 + Math.random() * 10,
        seaLevelHeight: -0.5 + Math.random(),
        oceanCurrentVelocity: 0.1 + Math.random() * 1.9,
        oceanCurrentDirection: Math.random() * 360
      },
      atmospheric: {
        windSpeed: Math.random() * 25,
        windDirection: Math.random() * 360,
        airPressure: 1000 + Math.random() * 30,
        visibility: 5000 + Math.random() * 10000,
        precipitation: Math.random() * 10
      }
    }
  }
  
  generateHistoricalData(days: number = 30): any[] {
    const data = []
    const now = new Date()
    
    for (let i = 0; i < days * 24; i++) {
      const timestamp = new Date(now.getTime() - i * 60 * 60 * 1000)
      
      data.push({
        timestamp: timestamp.toISOString(),
        averageHealthScore: 0.7 + Math.random() * 0.3,
        powerConsumption: 120 + Math.random() * 40,
        vesselProductivity: 45 + Math.random() * 15,
        berthUtilization: 0.75 + Math.random() * 0.2,
        weather: {
          windSpeed: Math.random() * 20,
          waveHeight: 0.5 + Math.random() * 2,
          temperature: 18 + Math.random() * 8
        }
      })
    }
    
    return data.reverse()
  }
}

// Export singleton instance
export const maritimeDataGenerator = new MaritimeDataGenerator()
```

## Advanced Testing and Performance Configuration

### Comprehensive Test Setup

**Create `src/test/setup.ts`:**
```typescript
import '@testing-library/jest-dom'
import { cleanup } from '@testing-library/react'
import { afterEach, beforeEach, vi } from 'vitest'

// Cleanup after each test
afterEach(() => {
  cleanup()
})

// Mock window.matchMedia for MUI components
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})

// Mock IntersectionObserver
global.IntersectionObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))

// Mock ResizeObserver
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}))

// Mock WebSocket
global.WebSocket = vi.fn().mockImplementation(() => ({
  send: vi.fn(),
  close: vi.fn(),
  addEventListener: vi.fn(),
  removeEventListener: vi.fn(),
  readyState: 1,
  CONNECTING: 0,
  OPEN: 1,
  CLOSING: 2,
  CLOSED: 3,
}))

// Mock Leaflet for map components
vi.mock('leaflet', () => ({
  Icon: vi.fn(() => ({})),
  LatLngBounds: vi.fn(() => ({})),
  Map: vi.fn(() => ({})),
  TileLayer: vi.fn(() => ({})),
  Marker: vi.fn(() => ({})),
  Popup: vi.fn(() => ({})),
}))

// Mock react-leaflet
vi.mock('react-leaflet', () => ({
  MapContainer: vi.fn(({ children }) => children),
  TileLayer: vi.fn(() => null),
  Marker: vi.fn(() => null),
  Popup: vi.fn(() => null),
  useMap: vi.fn(() => ({})),
}))
```

### Playwright Configuration

**Create `playwright.config.ts`:**
```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/junit.xml' }],
  ],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

## Main Application Integration

### Complete Application Setup

**Create `src/App.tsx`:**
```typescript
import React, { Suspense, lazy } from 'react'
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom'
import { ThemeProvider, createTheme } from '@mui/material/styles'
import { CssBaseline, Box } from '@mui/material'
import { ErrorBoundary } from 'react-error-boundary'
import { useMaritimeWebSocket } from '@hooks/useWebSocket'
import { maritimeDataGenerator } from '@data/maritimeDataGenerator'
import Sidebar from '@components/Sidebar'
import LoadingSpinner from '@components/LoadingSpinner'
import ErrorFallback from '@components/ErrorFallback'

// Lazy load dashboard components
const Dashboard = lazy(() => import('@components/Dashboard'))
const PortMap = lazy(() => import('@components/PortMap'))
const AnalyticsDashboard = lazy(() => import('@components/AnalyticsDashboard'))
const EquipmentMonitoring = lazy(() => import('@components/EquipmentMonitoring'))

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
  },
})

function App() {
  const { isConnected, lastMessage, connectionStatus } = useMaritimeWebSocket('ws://localhost:8080')
  
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <Router>
        <ErrorBoundary FallbackComponent={ErrorFallback}>
          <Box sx={{ display: 'flex' }}>
            <Sidebar connectionStatus={connectionStatus} />
            <Box component="main" sx={{ flexGrow: 1, p: 3 }}>
              <Suspense fallback={<LoadingSpinner />}>
                <Routes>
                  <Route path="/" element={<Dashboard />} />
                  <Route path="/port-map" element={<PortMap vessels={[]} center={[33.7362, -118.2632]} zoom={12} />} />
                  <Route path="/analytics" element={<AnalyticsDashboard portMetrics={[]} realtimeData={[]} />} />
                  <Route path="/equipment" element={<EquipmentMonitoring equipmentData={[]} historicalData={[]} />} />
                </Routes>
              </Suspense>
            </Box>
          </Box>
        </ErrorBoundary>
      </Router>
    </ThemeProvider>
  )
}

export default App
```

### Package.json Scripts

**Update `package.json`:**
```json
{
  "name": "smart-port-command-center",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "build:analyze": "npm run build && npx vite-bundle-analyzer dist/assets/*.js",
    "typecheck": "tsc --noEmit"
  }
}
```

## Backend Integration and WebSocket Server

### Simple Node.js WebSocket Server

**Create `server/websocket-server.js`:**
```javascript
const WebSocket = require('ws')
const { MaritimeDataGenerator } = require('./maritime-data-generator')

const wss = new WebSocket.Server({ port: 8080 })
const dataGenerator = new MaritimeDataGenerator()

console.log('Maritime WebSocket server running on ws://localhost:8080')

wss.on('connection', (ws) => {
  console.log('Client connected')
  
  // Send initial data
  ws.send(JSON.stringify({
    type: 'vessel_update',
    data: dataGenerator.generateAISData(25),
    timestamp: new Date().toISOString()
  }))
  
  // Send real-time updates every 5 seconds
  const interval = setInterval(() => {
    const messageTypes = ['vessel_update', 'port_metrics', 'equipment_data', 'weather_data']
    const randomType = messageTypes[Math.floor(Math.random() * messageTypes.length)]
    
    let data
    switch (randomType) {
      case 'vessel_update':
        data = dataGenerator.generateAISData(5)
        break
      case 'port_metrics':
        data = dataGenerator.generatePortMetrics()
        break
      case 'equipment_data':
        data = dataGenerator.generateEquipmentData(3)
        break
      case 'weather_data':
        data = dataGenerator.generateWeatherData()
        break
      default:
        data = {}
    }
    
    ws.send(JSON.stringify({
      type: randomType,
      data,
      timestamp: new Date().toISOString()
    }))
  }, 5000)
  
  ws.on('close', () => {
    console.log('Client disconnected')
    clearInterval(interval)
  })
  
  ws.on('error', (error) => {
    console.error('WebSocket error:', error)
  })
})
```

## Development Workflow and Best Practices

### Development Commands

**Start the development environment:**
```bash
# Terminal 1: Start the frontend
npm run dev

# Terminal 2: Start the WebSocket server
cd server
node websocket-server.js

# Terminal 3: Run tests in watch mode
npm run test

# Terminal 4: Type checking
npm run typecheck
```

### Production Build and Deployment

**Build and optimize for production:**
```bash
# Run full test suite
npm run test:coverage
npm run test:e2e

# Type checking
npm run typecheck

# Lint and fix
npm run lint:fix

# Build for production
npm run build

# Analyze bundle size
npm run build:analyze

# Preview production build
npm run preview
```

### Performance Optimization Checklist

1. **Code Splitting**: All major components are lazy-loaded
2. **Bundle Analysis**: Use webpack-bundle-analyzer to identify large dependencies
3. **Memory Management**: Implement cleanup in useEffect hooks
4. **Virtualization**: Use react-window for large data sets
5. **Memoization**: Strategic use of React.memo, useMemo, and useCallback
6. **WebSocket Optimization**: Implement message throttling and buffering
7. **Chart Performance**: Disable animations for real-time updates
8. **State Management**: Use Zustand for lightweight state management

### Security Considerations

1. **WebSocket Security**: Use WSS in production
2. **Input Validation**: Validate all incoming WebSocket messages
3. **Authentication**: Implement token-based authentication
4. **CORS**: Configure proper CORS policies
5. **Rate Limiting**: Implement client-side rate limiting
6. **Environment Variables**: Use .env files for sensitive configuration

This comprehensive guide provides a complete foundation for building a production-ready Smart Port Command Center dashboard with modern React patterns, real-time data visualization, and maritime domain expertise integrated throughout the application architecture.