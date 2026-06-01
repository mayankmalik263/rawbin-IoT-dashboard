# Rawbin IoT Dashboard — Main Dashboard

Complete dashboard layout with sidebar, navbar, real metrics, charts, and activity tracking.

---

## Layout Structure

### `app/dashboard/layout.tsx`
```typescript
'use client';

import { useAuth } from '@/hooks/useAuth';
import { useRouter } from 'next/navigation';
import { useEffect } from 'react';
import { Sidebar } from '@/components/layout/Sidebar';
import { Navbar } from '@/components/layout/Navbar';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const { isAuthenticated } = useAuth();
  const router = useRouter();

  useEffect(() => {
    if (!isAuthenticated) {
      router.push('/');
    }
  }, [isAuthenticated, router]);

  if (!isAuthenticated) {
    return null;
  }

  return (
    <div className="flex h-screen bg-gray-50 dark:bg-gray-950">
      <Sidebar />
      <div className="flex-1 flex flex-col overflow-hidden">
        <Navbar />
        <main className="flex-1 overflow-y-auto">
          {children}
        </main>
      </div>
    </div>
  );
}
```

---

## Layout Components

### `components/layout/Sidebar.tsx`
```typescript
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { useState } from 'react';

const navItems = [
  { href: '/dashboard', label: 'Dashboard', icon: '📊' },
  { href: '/devices', label: 'Devices', icon: '🔌' },
  { href: '/analytics', label: 'Analytics', icon: '📈' },
  { href: '/alerts', label: 'Alerts', icon: '🔔' },
  { href: '/admin', label: 'Admin', icon: '⚙️' },
];

export function Sidebar() {
  const pathname = usePathname();
  const [isMobileOpen, setIsMobileOpen] = useState(false);

  return (
    <>
      <aside
        className={`w-64 bg-white dark:bg-gray-900 border-r border-gray-200 dark:border-gray-800 flex flex-col transition-all duration-300 fixed md:relative h-screen z-40 ${
          isMobileOpen ? 'translate-x-0' : '-translate-x-full md:translate-x-0'
        }`}
      >
        <div className="p-6 border-b border-gray-200 dark:border-gray-800">
          <h1 className="text-2xl font-bold text-emerald-600 dark:text-emerald-400">
            Rawbin
          </h1>
          <p className="text-xs text-gray-500 dark:text-gray-400 mt-1">
            Composting Intelligence
          </p>
        </div>

        <nav className="flex-1 p-4 space-y-2">
          {navItems.map((item) => {
            const isActive = pathname.startsWith(item.href);
            return (
              <Link
                key={item.href}
                href={item.href}
                className={`flex items-center gap-3 px-4 py-3 rounded-lg transition-all min-h-[44px] ${
                  isActive
                    ? 'bg-emerald-50 dark:bg-emerald-900/20 text-emerald-700 dark:text-emerald-300 border-l-4 border-emerald-600 pl-3'
                    : 'text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-800'
                }`}
              >
                <span className="text-lg">{item.icon}</span>
                <span className="font-medium">{item.label}</span>
              </Link>
            );
          })}
        </nav>

        <div className="p-4 border-t border-gray-200 dark:border-gray-800">
          <div className="p-3 bg-blue-50 dark:bg-blue-900/20 rounded-lg text-xs text-blue-800 dark:text-blue-200">
            📚 Tip: Check Analytics to understand composting trends over time.
          </div>
        </div>
      </aside>

      <button
        onClick={() => setIsMobileOpen(!isMobileOpen)}
        className="md:hidden fixed bottom-6 right-6 w-14 h-14 bg-emerald-600 text-white rounded-full flex items-center justify-center shadow-lg z-50"
      >
        ☰
      </button>

      {isMobileOpen && (
        <div
          className="fixed inset-0 bg-black/50 md:hidden z-30"
          onClick={() => setIsMobileOpen(false)}
        />
      )}
    </>
  );
}
```

### `components/layout/Navbar.tsx`
```typescript
'use client';

import { useAuth } from '@/hooks/useAuth';
import { useTheme } from '@/hooks/useTheme';
import { useState } from 'react';

export function Navbar() {
  const { user, logout } = useAuth();
  const { isDark, setTheme } = useTheme();
  const [isProfileOpen, setIsProfileOpen] = useState(false);

  return (
    <nav className="bg-white dark:bg-gray-900 border-b border-gray-200 dark:border-gray-800 px-6 py-4 flex items-center justify-between">
      <div className="flex items-center gap-4">
        <h2 className="text-lg font-semibold text-gray-900 dark:text-white">
          Dashboard
        </h2>
      </div>

      <div className="flex items-center gap-4">
        <button
          onClick={() => setTheme(isDark ? 'light' : 'dark')}
          className="p-2 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg transition-colors"
          aria-label="Toggle theme"
        >
          {isDark ? '☀️' : '🌙'}
        </button>

        <button
          onClick={() => setIsProfileOpen(!isProfileOpen)}
          className="flex items-center gap-3 px-4 py-2 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg transition-colors min-h-[44px]"
        >
          <div className="w-8 h-8 rounded-full bg-emerald-600 text-white flex items-center justify-center font-bold text-sm">
            {user?.name?.charAt(0) || 'U'}
          </div>
          <span className="text-sm font-medium text-gray-700 dark:text-gray-300">
            {user?.name || 'User'}
          </span>
        </button>

        {isProfileOpen && (
          <div className="absolute top-16 right-6 bg-white dark:bg-gray-800 rounded-lg shadow-lg border border-gray-200 dark:border-gray-700 z-50 min-w-48">
            <div className="p-4 border-b border-gray-200 dark:border-gray-700">
              <p className="text-sm font-medium text-gray-900 dark:text-white">
                {user?.email}
              </p>
              <p className="text-xs text-gray-500 dark:text-gray-400 mt-1">
                Role: {user?.role}
              </p>
            </div>
            <div className="p-2">
              <button
                onClick={() => {
                  logout();
                  setIsProfileOpen(false);
                }}
                className="w-full text-left px-4 py-2 text-sm text-red-600 dark:text-red-400 hover:bg-red-50 dark:hover:bg-red-900/20 rounded transition-colors"
              >
                Sign out
              </button>
            </div>
          </div>
        )}
      </div>
    </nav>
  );
}
```

---

## Main Dashboard Page

### `app/dashboard/page.tsx`
```typescript
'use client';

import { useDeviceStore } from '@/store/deviceStore';
import { useEffect, useState } from 'react';
import { SustainabilityScore } from '@/components/dashboard/SustainabilityScore';
import { MetricCard } from '@/components/dashboard/MetricCard';
import { CompositingProgress } from '@/components/dashboard/CompositingProgress';
import { UsageChart } from '@/components/dashboard/UsageChart';
import { DeviceStatus } from '@/components/dashboard/DeviceStatus';
import { ActivityLog } from '@/components/dashboard/ActivityLog';
import { StreakTracker } from '@/components/dashboard/StreakTracker';
import { QuickActions } from '@/components/dashboard/QuickActions';

const mockDevices = [
  {
    id: 'device_001',
    name: 'Rawbin Unit #1',
    status: 'active' as const,
    location: 'Kitchen',
    temperature: 62,
    moisture: 45,
    wasteProcessed: 245,
    compostGenerated: 78,
    carbonAvoided: 22.5,
    lastSync: new Date(Date.now() - 5 * 60000),
  },
];

const mockActivities = [
  {
    id: '1',
    type: 'waste_added',
    description: 'Added 2.5 kg food waste',
    timestamp: new Date(Date.now() - 30 * 60000),
    icon: '♻️',
  },
  {
    id: '2',
    type: 'cycle_complete',
    description: 'Composting cycle completed',
    timestamp: new Date(Date.now() - 2 * 3600000),
    icon: '✅',
  },
  {
    id: '3',
    type: 'maintenance',
    description: 'System check passed',
    timestamp: new Date(Date.now() - 24 * 3600000),
    icon: '⚙️',
  },
];

export default function DashboardPage() {
  const { setDevices, devices } = useDeviceStore();
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setDevices(mockDevices);
    setIsLoading(false);
  }, [setDevices]);

  const device = devices[0] || mockDevices[0];
  const sustainabilityScore = 78;
  const currentStreak = 15;
  const longestStreak = 32;

  if (isLoading) {
    return (
      <div className="p-6 flex items-center justify-center h-full">
        <div className="text-gray-500 dark:text-gray-400">Loading...</div>
      </div>
    );
  }

  return (
    <div className="p-6 space-y-6">
      <SustainabilityScore score={sustainabilityScore} />

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <MetricCard
          label="Waste processed"
          value={device.wasteProcessed}
          unit="kg"
          trend="+12%"
          target={300}
          icon="♻️"
        />
        <MetricCard
          label="Compost generated"
          value={device.compostGenerated}
          unit="kg"
          trend="+8%"
          target={100}
          icon="🌱"
        />
        <MetricCard
          label="Carbon avoided"
          value={device.carbonAvoided}
          unit="kg CO₂"
          trend="+15%"
          target={30}
          icon="🌍"
        />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <CompositingProgress device={device} />
        <UsageChart />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <StreakTracker current={currentStreak} longest={longestStreak} />
        <DeviceStatus device={device} />
        <QuickActions />
      </div>

      <ActivityLog activities={mockActivities} />
    </div>
  );
}
```

---

## Dashboard Components

### `components/dashboard/SustainabilityScore.tsx`
```typescript
interface SustainabilityScoreProps {
  score: number;
}

export function SustainabilityScore({ score }: SustainabilityScoreProps) {
  const getScoreColor = (s: number) => {
    if (s >= 80) return 'text-emerald-600 dark:text-emerald-400';
    if (s >= 60) return 'text-blue-600 dark:text-blue-400';
    return 'text-amber-600 dark:text-amber-400';
  };

  const getScoreLabel = (s: number) => {
    if (s >= 80) return 'Excellent';
    if (s >= 60) return 'Good';
    return 'Fair';
  };

  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-lg font-semibold text-gray-900 dark:text-white mb-2">
            Sustainability score
          </h2>
          <p className="text-sm text-gray-600 dark:text-gray-400">
            Based on your composting habits and environmental impact
          </p>
        </div>
        <div className="text-right">
          <div className={`text-5xl font-bold ${getScoreColor(score)}`}>
            {score}
          </div>
          <p className={`text-sm font-medium mt-2 ${getScoreColor(score)}`}>
            {getScoreLabel(score)}
          </p>
        </div>
      </div>

      <div className="mt-6 w-full bg-gray-200 dark:bg-gray-700 rounded-full h-2 overflow-hidden">
        <div
          className="bg-emerald-600 h-full rounded-full transition-all duration-500"
          style={{ width: `${score}%` }}
        />
      </div>
    </div>
  );
}
```

### `components/dashboard/MetricCard.tsx`
```typescript
interface MetricCardProps {
  label: string;
  value: number;
  unit: string;
  trend: string;
  target: number;
  icon: string;
}

export function MetricCard({
  label,
  value,
  unit,
  trend,
  target,
  icon,
}: MetricCardProps) {
  const progress = (value / target) * 100;

  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm hover:shadow-md hover:border-gray-300 dark:hover:border-gray-700 transition-all">
      <div className="flex items-start justify-between mb-4">
        <div>
          <p className="text-sm text-gray-600 dark:text-gray-400 font-medium">
            {label}
          </p>
          <h3 className="text-3xl font-bold text-gray-900 dark:text-white mt-1">
            {value}
            <span className="text-lg text-gray-600 dark:text-gray-400 ml-2">
              {unit}
            </span>
          </h3>
        </div>
        <span className="text-3xl">{icon}</span>
      </div>

      <div className="flex items-center justify-between mb-3">
        <span className="text-xs text-green-600 dark:text-green-400 font-medium">
          {trend}
        </span>
        <span className="text-xs text-gray-500 dark:text-gray-400">
          Target: {target}
          {unit}
        </span>
      </div>

      <div className="w-full bg-gray-200 dark:bg-gray-700 rounded-full h-1.5 overflow-hidden">
        <div
          className="bg-emerald-600 h-full rounded-full transition-all duration-500"
          style={{ width: `${Math.min(progress, 100)}%` }}
        />
      </div>
    </div>
  );
}
```

### `components/dashboard/CompositingProgress.tsx`
```typescript
'use client';

import { PieChart, Pie, Cell, ResponsiveContainer, Legend } from 'recharts';
import { Device } from '@/types';

interface CompositingProgressProps {
  device: Device;
}

export function CompositingProgress({ device }: CompositingProgressProps) {
  const progress = 65;
  const data = [
    { name: 'Progress', value: progress },
    { name: 'Remaining', value: 100 - progress },
  ];

  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">
        Composting cycle progress
      </h3>

      <div className="flex items-center justify-center">
        <ResponsiveContainer width="100%" height={250}>
          <PieChart>
            <Pie
              data={data}
              cx="50%"
              cy="50%"
              innerRadius={60}
              outerRadius={90}
              dataKey="value"
            >
              <Cell fill="#10b981" />
              <Cell fill="#e5e7eb" />
            </Pie>
          </PieChart>
        </ResponsiveContainer>
      </div>

      <div className="text-center">
        <p className="text-4xl font-bold text-emerald-600 dark:text-emerald-400">
          {progress}%
        </p>
        <p className="text-sm text-gray-600 dark:text-gray-400 mt-1">
          Est. completion in 8 days
        </p>
      </div>

      <div className="mt-6 p-3 bg-gray-50 dark:bg-gray-800 rounded-lg">
        <div className="text-sm">
          <div className="flex justify-between mb-2">
            <span className="text-gray-600 dark:text-gray-400">Temperature</span>
            <span className="font-medium text-gray-900 dark:text-white">
              {device.temperature}°C
            </span>
          </div>
          <div className="flex justify-between">
            <span className="text-gray-600 dark:text-gray-400">Moisture</span>
            <span className="font-medium text-gray-900 dark:text-white">
              {device.moisture}%
            </span>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### `components/dashboard/UsageChart.tsx`
```typescript
'use client';

import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer,
} from 'recharts';

const mockData = [
  { day: 'Mon', waste: 2.5, compost: 0.8 },
  { day: 'Tue', waste: 3.2, compost: 1.0 },
  { day: 'Wed', waste: 2.1, compost: 0.7 },
  { day: 'Thu', waste: 3.8, compost: 1.2 },
  { day: 'Fri', waste: 2.9, compost: 0.9 },
  { day: 'Sat', waste: 4.2, compost: 1.3 },
  { day: 'Sun', waste: 3.5, compost: 1.1 },
];

export function UsageChart() {
  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">
        Weekly usage
      </h3>

      <ResponsiveContainer width="100%" height={300}>
        <BarChart data={mockData}>
          <CartesianGrid strokeDasharray="3 3" stroke="#e5e7eb" />
          <XAxis dataKey="day" tick={{ fontSize: 12 }} />
          <YAxis tick={{ fontSize: 12 }} />
          <Tooltip
            contentStyle={{
              backgroundColor: '#1f2937',
              border: 'none',
              borderRadius: '8px',
              color: '#fff',
            }}
          />
          <Legend />
          <Bar dataKey="waste" fill="#10b981" radius={[8, 8, 0, 0]} />
          <Bar dataKey="compost" fill="#6366f1" radius={[8, 8, 0, 0]} />
        </BarChart>
      </ResponsiveContainer>
    </div>
  );
}
```

### `components/dashboard/DeviceStatus.tsx`
```typescript
import { Device } from '@/types';
import { Badge } from '@/components/ui/Badge';

interface DeviceStatusProps {
  device: Device;
}

export function DeviceStatus({ device }: DeviceStatusProps) {
  const getStatusVariant = (status: string) => {
    switch (status) {
      case 'active':
        return 'success';
      case 'offline':
        return 'danger';
      case 'warning':
        return 'warning';
      default:
        return 'info';
    }
  };

  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">
        Device status
      </h3>

      <div className="space-y-4">
        <div>
          <p className="text-sm text-gray-600 dark:text-gray-400 mb-2">
            {device.name}
          </p>
          <Badge label={device.status} variant={getStatusVariant(device.status)} />
        </div>

        <div className="pt-4 border-t border-gray-200 dark:border-gray-700 space-y-3">
          <div className="flex justify-between text-sm">
            <span className="text-gray-600 dark:text-gray-400">Location</span>
            <span className="font-medium text-gray-900 dark:text-white">
              {device.location}
            </span>
          </div>
          <div className="flex justify-between text-sm">
            <span className="text-gray-600 dark:text-gray-400">Last sync</span>
            <span className="font-medium text-gray-900 dark:text-white">
              5m ago
            </span>
          </div>
          <div className="flex justify-between text-sm">
            <span className="text-gray-600 dark:text-gray-400">Uptime</span>
            <span className="font-medium text-gray-900 dark:text-white">
              98.2%
            </span>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### `components/dashboard/StreakTracker.tsx`
```typescript
interface StreakTrackerProps {
  current: number;
  longest: number;
}

export function StreakTracker({ current, longest }: StreakTrackerProps) {
  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">
        Composting streak
      </h3>

      <div className="space-y-4">
        <div className="text-center py-4 bg-gradient-to-r from-emerald-50 to-emerald-100 dark:from-emerald-900/20 dark:to-emerald-800/20 rounded-lg">
          <div className="text-4xl font-bold text-emerald-600 dark:text-emerald-400">
            {current}
          </div>
          <p className="text-sm text-emerald-700 dark:text-emerald-300 mt-1">
            Days in a row
          </p>
        </div>

        <div className="p-3 bg-gray-50 dark:bg-gray-800 rounded-lg">
          <p className="text-xs text-gray-600 dark:text-gray-400 mb-1">
            Personal best
          </p>
          <p className="text-lg font-bold text-gray-900 dark:text-white">
            {longest} days
          </p>
        </div>
      </div>

      <div className="mt-6 p-3 bg-blue-50 dark:bg-blue-900/20 rounded-lg text-xs text-blue-800 dark:text-blue-200">
        Keep it up! You're making a real difference.
      </div>
    </div>
  );
}
```

### `components/dashboard/QuickActions.tsx`
```typescript
import { Button } from '@/components/ui/Button';

export function QuickActions() {
  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">
        Quick actions
      </h3>

      <div className="space-y-2">
        <Button
          variant="outline"
          size="lg"
          className="w-full justify-start"
        >
          ➕ Log waste
        </Button>
        <Button
          variant="outline"
          size="lg"
          className="w-full justify-start"
        >
          📸 Take photo
        </Button>
        <Button
          variant="outline"
          size="lg"
          className="w-full justify-start"
        >
          ⚙️ Device settings
        </Button>
      </div>
    </div>
  );
}
```

### `components/dashboard/ActivityLog.tsx`
```typescript
interface Activity {
  id: string;
  type: string;
  description: string;
  timestamp: Date;
  icon: string;
}

interface ActivityLogProps {
  activities: Activity[];
}

export function ActivityLog({ activities }: ActivityLogProps) {
  const formatTime = (date: Date) => {
    const now = new Date();
    const diffMs = now.getTime() - date.getTime();
    const diffMins = Math.floor(diffMs / 60000);
    const diffHours = Math.floor(diffMs / 3600000);
    const diffDays = Math.floor(diffMs / 86400000);

    if (diffMins < 60) return `${diffMins}m ago`;
    if (diffHours < 24) return `${diffHours}h ago`;
    return `${diffDays}d ago`;
  };

  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">
        Recent activity
      </h3>

      <div className="space-y-0 divide-y divide-gray-200 dark:divide-gray-800">
        {activities.map((activity, idx) => (
          <div
            key={activity.id}
            className={`py-4 flex items-start gap-4 ${
              idx % 2 === 0 ? 'bg-white dark:bg-gray-900' : 'bg-gray-50 dark:bg-gray-800'
            }`}
          >
            <span className="text-2xl mt-0.5">{activity.icon}</span>
            <div className="flex-1 min-w-0">
              <p className="text-sm font-medium text-gray-900 dark:text-white">
                {activity.description}
              </p>
              <p className="text-xs text-gray-500 dark:text-gray-400 mt-1">
                {formatTime(activity.timestamp)}
              </p>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Types Update

### `types/index.ts` (Extended)
```typescript
export interface ChartDataPoint {
  timestamp: Date;
  value: number;
  label?: string;
}

export interface DashboardMetrics {
  totalWasteProcessed: number;
  totalCompostGenerated: number;
  totalCarbonAvoided: number;
  currentCompostingProgress: number;
}
```

---

## Mock Data Hook

### `hooks/useDashboardData.ts`
```typescript
import { useDeviceStore } from '@/store/deviceStore';
import { useEffect } from 'react';

export function useDashboardData() {
  const { devices, setDevices } = useDeviceStore();

  useEffect(() => {
    const mockDevices = [
      {
        id: 'device_001',
        name: 'Rawbin Unit #1',
        status: 'active' as const,
        location: 'Kitchen',
        temperature: 62,
        moisture: 45,
        wasteProcessed: 245,
        compostGenerated: 78,
        carbonAvoided: 22.5,
        lastSync: new Date(Date.now() - 5 * 60000),
      },
    ];
    setDevices(mockDevices);
  }, [setDevices]);

  return {
    devices,
    primaryDevice: devices[0],
  };
}
```

---

## Responsive Breakpoints

Mobile-first:
- `grid-cols-1` (mobile default)
- `md:grid-cols-2` (tablet up)
- `md:grid-cols-3` (3-col metrics on desktop)
- `lg:grid-cols-2` (2-col for progress + usage)
- `lg:grid-cols-3` (3-col for streaks + device + actions)

Sidebar collapsible on mobile via hamburger button.

---

## Key Features

✓ Responsive sidebar (mobile drawer, desktop fixed)
✓ Top navbar with profile menu + theme toggle
✓ Sustainability score card
✓ 3-column metric cards with progress bars
✓ Composting progress donut chart (Recharts)
✓ Weekly usage bar chart (waste vs compost)
✓ Device status with health indicators
✓ Streak tracker (current + personal best)
✓ Quick action buttons
✓ Activity log with timestamps
✓ Dark/light mode support
✓ Accessibility (44px+ tap targets)
✓ TypeScript throughout
✓ Ready for real device data via API

---

## Next: Analytics & Trends

Ready for advanced analytics dashboard (moisture/temperature trends, date filters, insights, activity logs)?
