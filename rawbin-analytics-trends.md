# Rawbin IoT Dashboard — Analytics & Trends

Advanced analytics with line charts, date filters, comparison mode, insights, and activity logs.

---

## Main Analytics Page

### `app/dashboard/analytics/page.tsx`
```typescript
'use client';

import { useState } from 'react';
import { DateRangeFilter } from '@/components/analytics/DateRangeFilter';
import { MoistureTemperatureChart } from '@/components/analytics/MoistureTemperatureChart';
import { InsightsCards } from '@/components/analytics/InsightsCards';
import { ActivityLogDetailed } from '@/components/analytics/ActivityLogDetailed';
import { ComparisonToggle } from '@/components/analytics/ComparisonToggle';
import { Button } from '@/components/ui/Button';

type DateRange = 'week' | 'month' | 'quarter' | 'year';

interface Filters {
  dateRange: DateRange;
  comparisonMode: boolean;
  selectedMetrics: string[];
}

const mockChartData = [
  { time: '00:00', moisture: 42, temperature: 58 },
  { time: '04:00', moisture: 45, temperature: 56 },
  { time: '08:00', moisture: 48, temperature: 62 },
  { time: '12:00', moisture: 50, temperature: 68 },
  { time: '16:00', moisture: 52, temperature: 70 },
  { time: '20:00', moisture: 48, temperature: 64 },
  { time: '24:00', moisture: 45, temperature: 60 },
];

const mockInsights = [
  {
    id: '1',
    title: 'Optimal moisture reached',
    description: 'Your device hit the ideal moisture level 3 times this week.',
    icon: '💧',
    trend: 'up',
  },
  {
    id: '2',
    title: 'Temperature stable',
    description: 'Temperature variance is only 2°C. Great thermal regulation!',
    icon: '🌡️',
    trend: 'stable',
  },
  {
    id: '3',
    title: 'Activity increasing',
    description: '45% more waste processed than last week.',
    icon: '📈',
    trend: 'up',
  },
];

const mockActivityLog = [
  {
    id: '1',
    timestamp: new Date(Date.now() - 2 * 3600000),
    action: 'Waste added',
    quantity: '2.5 kg',
    moisture: '46%',
    temperature: '62°C',
  },
  {
    id: '2',
    timestamp: new Date(Date.now() - 6 * 3600000),
    action: 'System check',
    quantity: 'All systems nominal',
    moisture: '45%',
    temperature: '60°C',
  },
  {
    id: '3',
    timestamp: new Date(Date.now() - 12 * 3600000),
    action: 'Moisture adjusted',
    quantity: 'Added water',
    moisture: '44%',
    temperature: '58°C',
  },
  {
    id: '4',
    timestamp: new Date(Date.now() - 24 * 3600000),
    action: 'Cycle progress',
    quantity: '65% complete',
    moisture: '48%',
    temperature: '65°C',
  },
];

export default function AnalyticsPage() {
  const [filters, setFilters] = useState<Filters>({
    dateRange: 'week',
    comparisonMode: false,
    selectedMetrics: ['moisture', 'temperature'],
  });

  const handleDateRangeChange = (range: DateRange) => {
    setFilters((prev) => ({ ...prev, dateRange: range }));
  };

  const handleExport = () => {
    const csv = 'Time,Moisture,Temperature\n' +
      mockChartData
        .map((row) => `${row.time},${row.moisture},${row.temperature}`)
        .join('\n');
    
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `rawbin-analytics-${new Date().toISOString()}.csv`;
    a.click();
  };

  return (
    <div className="p-6 space-y-6">
      <div className="flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
        <div>
          <h1 className="text-3xl font-bold text-gray-900 dark:text-white">
            Analytics & trends
          </h1>
          <p className="text-gray-600 dark:text-gray-400 mt-1">
            Deep insights into your composting patterns
          </p>
        </div>
        <Button
          variant="outline"
          onClick={handleExport}
        >
          ⬇️ Export data
        </Button>
      </div>

      <div className="flex flex-col md:flex-row gap-4 items-start md:items-center">
        <DateRangeFilter
          selectedRange={filters.dateRange}
          onRangeChange={handleDateRangeChange}
        />
        <ComparisonToggle />
      </div>

      <MoistureTemperatureChart data={mockChartData} />

      <InsightsCards insights={mockInsights} />

      <ActivityLogDetailed activities={mockActivityLog} />
    </div>
  );
}
```

---

## Analytics Components

### `components/analytics/DateRangeFilter.tsx`
```typescript
'use client';

import { useState } from 'react';

type DateRange = 'week' | 'month' | 'quarter' | 'year';

interface DateRangeFilterProps {
  selectedRange: DateRange;
  onRangeChange: (range: DateRange) => void;
}

const rangeOptions: { label: string; value: DateRange }[] = [
  { label: 'Last 7 days', value: 'week' },
  { label: 'Last 30 days', value: 'month' },
  { label: 'Last 90 days', value: 'quarter' },
  { label: 'Last year', value: 'year' },
];

export function DateRangeFilter({
  selectedRange,
  onRangeChange,
}: DateRangeFilterProps) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="relative">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="flex items-center gap-2 px-4 py-2.5 bg-white dark:bg-gray-900 border-2 border-gray-200 dark:border-gray-700 rounded-lg hover:border-emerald-500 dark:hover:border-emerald-400 transition-colors min-h-[44px]"
      >
        <span>📅</span>
        <span className="font-medium text-gray-900 dark:text-white">
          {rangeOptions.find((r) => r.value === selectedRange)?.label}
        </span>
        <span className="text-gray-400">▼</span>
      </button>

      {isOpen && (
        <div className="absolute top-full mt-2 bg-white dark:bg-gray-900 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg z-10 min-w-48">
          {rangeOptions.map((option) => (
            <button
              key={option.value}
              onClick={() => {
                onRangeChange(option.value);
                setIsOpen(false);
              }}
              className={`w-full text-left px-4 py-3 transition-colors ${
                selectedRange === option.value
                  ? 'bg-emerald-50 dark:bg-emerald-900/20 text-emerald-700 dark:text-emerald-300 font-medium border-l-4 border-emerald-600 pl-3'
                  : 'text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-800'
              }`}
            >
              {option.label}
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
```

### `components/analytics/MoistureTemperatureChart.tsx`
```typescript
'use client';

import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer,
  ComposedChart,
} from 'recharts';

interface ChartDataPoint {
  time: string;
  moisture: number;
  temperature: number;
}

interface MoistureTemperatureChartProps {
  data: ChartDataPoint[];
}

export function MoistureTemperatureChart({
  data,
}: MoistureTemperatureChartProps) {
  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">
        Moisture vs temperature
      </h3>

      <ResponsiveContainer width="100%" height={400}>
        <ComposedChart
          data={data}
          margin={{ top: 5, right: 30, left: 0, bottom: 5 }}
        >
          <CartesianGrid
            strokeDasharray="3 3"
            stroke="#e5e7eb"
            vertical={false}
          />
          <XAxis
            dataKey="time"
            tick={{ fontSize: 12 }}
            tickLine={false}
            stroke="#999"
          />
          <YAxis
            yAxisId="left"
            tick={{ fontSize: 12 }}
            tickLine={false}
            stroke="#999"
            label={{ value: 'Moisture (%)', angle: -90, position: 'insideLeft' }}
          />
          <YAxis
            yAxisId="right"
            orientation="right"
            tick={{ fontSize: 12 }}
            tickLine={false}
            stroke="#999"
            label={{ value: 'Temperature (°C)', angle: 90, position: 'insideRight' }}
          />
          <Tooltip
            contentStyle={{
              backgroundColor: '#1f2937',
              border: 'none',
              borderRadius: '8px',
              color: '#fff',
              padding: '12px',
            }}
            labelStyle={{ color: '#fff' }}
          />
          <Legend
            wrapperStyle={{ paddingTop: '16px' }}
            verticalAlign="top"
            height={36}
          />
          <Line
            yAxisId="left"
            type="monotone"
            dataKey="moisture"
            stroke="#10b981"
            strokeWidth={3}
            dot={false}
            name="Moisture (%)"
            isAnimationActive={true}
          />
          <Line
            yAxisId="right"
            type="monotone"
            dataKey="temperature"
            stroke="#f59e0b"
            strokeWidth={3}
            strokeDasharray="5 5"
            dot={false}
            name="Temperature (°C)"
            isAnimationActive={true}
          />
        </ComposedChart>
      </ResponsiveContainer>

      <div className="mt-6 grid grid-cols-2 md:grid-cols-4 gap-4">
        <StatBox label="Avg moisture" value="47.2%" color="text-emerald-600 dark:text-emerald-400" />
        <StatBox label="Avg temperature" value="62.7°C" color="text-amber-600 dark:text-amber-400" />
        <StatBox label="Peak moisture" value="52%" color="text-emerald-600 dark:text-emerald-400" />
        <StatBox label="Peak temperature" value="70°C" color="text-amber-600 dark:text-amber-400" />
      </div>
    </div>
  );
}

interface StatBoxProps {
  label: string;
  value: string;
  color: string;
}

function StatBox({ label, value, color }: StatBoxProps) {
  return (
    <div className="p-3 bg-gray-50 dark:bg-gray-800 rounded-lg text-center">
      <p className="text-xs text-gray-600 dark:text-gray-400 mb-1">{label}</p>
      <p className={`text-xl font-bold ${color}`}>{value}</p>
    </div>
  );
}
```

### `components/analytics/InsightsCards.tsx`
```typescript
interface Insight {
  id: string;
  title: string;
  description: string;
  icon: string;
  trend: 'up' | 'down' | 'stable';
}

interface InsightsCardsProps {
  insights: Insight[];
}

export function InsightsCards({ insights }: InsightsCardsProps) {
  return (
    <div className="space-y-4">
      <h3 className="text-lg font-semibold text-gray-900 dark:text-white">
        Key insights
      </h3>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {insights.map((insight) => (
          <div
            key={insight.id}
            className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-4 shadow-sm hover:shadow-md transition-shadow"
          >
            <div className="flex items-start gap-3">
              <span className="text-2xl">{insight.icon}</span>
              <div className="flex-1">
                <h4 className="font-semibold text-gray-900 dark:text-white">
                  {insight.title}
                </h4>
                <p className="text-sm text-gray-600 dark:text-gray-400 mt-1">
                  {insight.description}
                </p>
                <div className="mt-3">
                  <span
                    className={`text-xs font-medium px-2 py-1 rounded-full ${
                      insight.trend === 'up'
                        ? 'bg-green-100 text-green-800 dark:bg-green-900/30 dark:text-green-300'
                        : insight.trend === 'down'
                        ? 'bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-300'
                        : 'bg-blue-100 text-blue-800 dark:bg-blue-900/30 dark:text-blue-300'
                    }`}
                  >
                    {insight.trend === 'up'
                      ? '↑ Trending up'
                      : insight.trend === 'down'
                      ? '↓ Trending down'
                      : '→ Stable'}
                  </span>
                </div>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### `components/analytics/ActivityLogDetailed.tsx`
```typescript
interface ActivityEntry {
  id: string;
  timestamp: Date;
  action: string;
  quantity: string;
  moisture: string;
  temperature: string;
}

interface ActivityLogDetailedProps {
  activities: ActivityEntry[];
}

export function ActivityLogDetailed({
  activities,
}: ActivityLogDetailedProps) {
  const formatDate = (date: Date) => {
    return new Intl.DateTimeFormat('en-US', {
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit',
    }).format(date);
  };

  const getActionIcon = (action: string) => {
    switch (action) {
      case 'Waste added':
        return '♻️';
      case 'System check':
        return '✓';
      case 'Moisture adjusted':
        return '💧';
      case 'Cycle progress':
        return '📊';
      default:
        return '•';
    }
  };

  return (
    <div className="bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 shadow-sm overflow-hidden">
      <div className="p-6 border-b border-gray-200 dark:border-gray-800">
        <h3 className="text-lg font-semibold text-gray-900 dark:text-white">
          Activity log
        </h3>
      </div>

      <div className="overflow-x-auto">
        <table className="w-full text-sm">
          <thead>
            <tr className="border-b border-gray-200 dark:border-gray-800 bg-gray-50 dark:bg-gray-800/50">
              <th className="px-6 py-3 text-left font-semibold text-gray-900 dark:text-white">
                Timestamp
              </th>
              <th className="px-6 py-3 text-left font-semibold text-gray-900 dark:text-white">
                Action
              </th>
              <th className="px-6 py-3 text-left font-semibold text-gray-900 dark:text-white">
                Details
              </th>
              <th className="px-6 py-3 text-left font-semibold text-gray-900 dark:text-white">
                Conditions
              </th>
            </tr>
          </thead>
          <tbody className="divide-y divide-gray-200 dark:divide-gray-800">
            {activities.map((activity, idx) => (
              <tr
                key={activity.id}
                className={`${
                  idx % 2 === 0
                    ? 'bg-white dark:bg-gray-900'
                    : 'bg-gray-50 dark:bg-gray-800/50'
                } hover:bg-gray-100 dark:hover:bg-gray-800 transition-colors`}
              >
                <td className="px-6 py-4 text-gray-600 dark:text-gray-300 font-mono text-xs">
                  {formatDate(activity.timestamp)}
                </td>
                <td className="px-6 py-4">
                  <div className="flex items-center gap-2">
                    <span className="text-lg">
                      {getActionIcon(activity.action)}
                    </span>
                    <span className="font-medium text-gray-900 dark:text-white">
                      {activity.action}
                    </span>
                  </div>
                </td>
                <td className="px-6 py-4 text-gray-700 dark:text-gray-300">
                  {activity.quantity}
                </td>
                <td className="px-6 py-4 text-gray-600 dark:text-gray-400">
                  <div className="flex items-center gap-2 text-xs">
                    <span>{activity.moisture}</span>
                    <span className="text-gray-400">•</span>
                    <span>{activity.temperature}</span>
                  </div>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      <div className="p-4 border-t border-gray-200 dark:border-gray-800 flex justify-center">
        <button className="px-4 py-2 text-sm text-emerald-600 dark:text-emerald-400 hover:bg-emerald-50 dark:hover:bg-emerald-900/20 rounded-lg transition-colors font-medium">
          Load more
        </button>
      </div>
    </div>
  );
}
```

### `components/analytics/ComparisonToggle.tsx`
```typescript
'use client';

import { useState } from 'react';

export function ComparisonToggle() {
  const [comparisonMode, setComparisonMode] = useState(false);

  return (
    <div className="flex items-center gap-3 px-4 py-2.5 bg-white dark:bg-gray-900 border-2 border-gray-200 dark:border-gray-700 rounded-lg hover:border-emerald-500 dark:hover:border-emerald-400 transition-colors min-h-[44px]">
      <label className="flex items-center gap-2 cursor-pointer">
        <input
          type="checkbox"
          checked={comparisonMode}
          onChange={(e) => setComparisonMode(e.target.checked)}
          className="w-5 h-5 rounded border-gray-300 text-emerald-600 focus:ring-emerald-500 cursor-pointer"
        />
        <span className="font-medium text-gray-700 dark:text-gray-300">
          Compare periods
        </span>
      </label>
    </div>
  );
}
```

---

## Analytics Hooks

### `hooks/useAnalyticsData.ts`
```typescript
import { useState, useEffect } from 'react';

interface AnalyticsData {
  timestamp: Date;
  moisture: number;
  temperature: number;
  waste: number;
  compost: number;
}

export function useAnalyticsData(dateRange: 'week' | 'month' | 'quarter' | 'year') {
  const [data, setData] = useState<AnalyticsData[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    
    const generateMockData = () => {
      const points = dateRange === 'week' ? 7 : dateRange === 'month' ? 30 : 90;
      const newData: AnalyticsData[] = [];

      for (let i = 0; i < points; i++) {
        newData.push({
          timestamp: new Date(Date.now() - i * 24 * 3600000),
          moisture: 40 + Math.random() * 15,
          temperature: 55 + Math.random() * 20,
          waste: 2 + Math.random() * 3,
          compost: 0.5 + Math.random() * 1.5,
        });
      }

      return newData.reverse();
    };

    const timer = setTimeout(() => {
      setData(generateMockData());
      setIsLoading(false);
    }, 500);

    return () => clearTimeout(timer);
  }, [dateRange]);

  return { data, isLoading };
}
```

---

## Types

### `types/analytics.ts`
```typescript
export interface AnalyticsMetric {
  timestamp: Date;
  value: number;
  unit: string;
}

export interface Insight {
  id: string;
  title: string;
  description: string;
  metric: string;
  trend: 'up' | 'down' | 'stable';
  percentageChange: number;
}

export interface ComparisonPeriod {
  label: string;
  startDate: Date;
  endDate: Date;
  data: AnalyticsMetric[];
}
```

---

## API Services

### `services/analytics.ts`
```typescript
import { ApiClient } from './api';

export const analyticsService = {
  async getMetrics(
    deviceId: string,
    startDate: Date,
    endDate: Date
  ) {
    return ApiClient.get(
      `/analytics/metrics?deviceId=${deviceId}&start=${startDate.toISOString()}&end=${endDate.toISOString()}`
    );
  },

  async getInsights(deviceId: string) {
    return ApiClient.get(`/analytics/insights?deviceId=${deviceId}`);
  },

  async getActivityLog(deviceId: string, limit: number = 50) {
    return ApiClient.get(`/analytics/activity?deviceId=${deviceId}&limit=${limit}`);
  },

  async exportAnalytics(deviceId: string, format: 'csv' | 'json') {
    return ApiClient.get(`/analytics/export?deviceId=${deviceId}&format=${format}`);
  },
};
```

---

## Features

✓ Date range filters (week/month/quarter/year)
✓ Dual-axis line chart (moisture + temperature)
✓ Solid + dashed line styles
✓ Responsive tooltips on hover
✓ Legends + axis labels
✓ Summary stat boxes (avg, peak)
✓ Insights cards with trend indicators
✓ Detailed activity log table (timestamps, actions, conditions)
✓ Export data as CSV
✓ Comparison mode toggle (ready for multi-period)
✓ Alternating row backgrounds (accessibility)
✓ Dark mode support
✓ Fully responsive (mobile → desktop)
✓ TypeScript throughout

---

## Mock API Responses

**GET /api/analytics/metrics**
```json
[
  {
    "timestamp": "2026-05-28T10:30:00Z",
    "moisture": 42,
    "temperature": 62,
    "waste": 2.5,
    "compost": 0.8
  },
  {
    "timestamp": "2026-05-29T10:30:00Z",
    "moisture": 45,
    "temperature": 65,
    "waste": 3.2,
    "compost": 1.0
  }
]
```

**GET /api/analytics/insights**
```json
[
  {
    "id": "1",
    "title": "Optimal conditions",
    "description": "Conditions hit ideal range 3 times",
    "metric": "efficiency",
    "trend": "up",
    "percentageChange": 12
  }
]
```

---

## Next: Alerts & Notifications

Ready for alerts center (critical/warning/info severity, clear all, dismiss, filters)?
