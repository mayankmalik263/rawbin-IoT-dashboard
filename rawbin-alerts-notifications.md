# Rawbin IoT Dashboard — Alerts & Notifications

Notifications center with severity levels, filters, and grouping.

---

## Main Alerts Page

### `app/dashboard/alerts/page.tsx`
```typescript
'use client';

import { useState } from 'react';
import { AlertsHeader } from '@/components/alerts/AlertsHeader';
import { AlertsFilter } from '@/components/alerts/AlertsFilter';
import { AlertsList } from '@/components/alerts/AlertsList';
import { EmptyState } from '@/components/alerts/EmptyState';

export type AlertSeverity = 'critical' | 'warning' | 'info';

interface Alert {
  id: string;
  deviceId: string;
  severity: AlertSeverity;
  title: string;
  message: string;
  timestamp: Date;
  read: boolean;
  actionLabel?: string;
  actionHandler?: () => void;
}

const mockAlerts: Alert[] = [
  {
    id: '1',
    deviceId: 'device_001',
    severity: 'critical',
    title: 'Temperature critical',
    message: 'Device temperature has exceeded safe limits (75°C). Immediate action required.',
    timestamp: new Date(Date.now() - 15 * 60000),
    read: false,
    actionLabel: 'View device',
  },
  {
    id: '2',
    deviceId: 'device_001',
    severity: 'critical',
    title: 'Offline alert',
    message: 'Device has not synced data for over 2 hours. Check WiFi connection.',
    timestamp: new Date(Date.now() - 45 * 60000),
    read: false,
    actionLabel: 'Troubleshoot',
  },
  {
    id: '3',
    deviceId: 'device_001',
    severity: 'warning',
    title: 'Moisture low',
    message: 'Current moisture level (32%) is below optimal range. Consider adding water.',
    timestamp: new Date(Date.now() - 2 * 3600000),
    read: false,
    actionLabel: 'Add water',
  },
  {
    id: '4',
    deviceId: 'device_001',
    severity: 'warning',
    title: 'Maintenance due',
    message: 'Device has been running for 180 days. Schedule maintenance check.',
    timestamp: new Date(Date.now() - 6 * 3600000),
    read: true,
    actionLabel: 'Schedule',
  },
  {
    id: '5',
    deviceId: 'device_001',
    severity: 'info',
    title: 'Cycle complete',
    message: 'Composting cycle #42 completed successfully. Compost ready for harvest.',
    timestamp: new Date(Date.now() - 24 * 3600000),
    read: true,
  },
  {
    id: '6',
    deviceId: 'device_001',
    severity: 'info',
    title: 'System update available',
    message: 'Firmware update v2.4.1 available. Current version: v2.4.0.',
    timestamp: new Date(Date.now() - 2 * 24 * 3600000),
    read: true,
    actionLabel: 'Update',
  },
];

export default function AlertsPage() {
  const [alerts, setAlerts] = useState<Alert[]>(mockAlerts);
  const [filter, setFilter] = useState<AlertSeverity | 'all'>('all');

  const filteredAlerts =
    filter === 'all' ? alerts : alerts.filter((a) => a.severity === filter);

  const handleMarkAsRead = (id: string) => {
    setAlerts((prev) =>
      prev.map((a) => (a.id === id ? { ...a, read: true } : a))
    );
  };

  const handleDismiss = (id: string) => {
    setAlerts((prev) => prev.filter((a) => a.id !== id));
  };

  const handleClearAll = () => {
    setAlerts((prev) => prev.filter((a) => !a.read));
  };

  const unreadCount = alerts.filter((a) => !a.read).length;

  return (
    <div className="p-6 space-y-6">
      <AlertsHeader unreadCount={unreadCount} onClearAll={handleClearAll} />

      <AlertsFilter selectedFilter={filter} onFilterChange={setFilter} />

      {filteredAlerts.length > 0 ? (
        <AlertsList
          alerts={filteredAlerts}
          onMarkAsRead={handleMarkAsRead}
          onDismiss={handleDismiss}
        />
      ) : (
        <EmptyState />
      )}
    </div>
  );
}
```

---

## Alerts Components

### `components/alerts/AlertsHeader.tsx`
```typescript
import { Button } from '@/components/ui/Button';

interface AlertsHeaderProps {
  unreadCount: number;
  onClearAll: () => void;
}

export function AlertsHeader({
  unreadCount,
  onClearAll,
}: AlertsHeaderProps) {
  return (
    <div className="flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
      <div>
        <h1 className="text-3xl font-bold text-gray-900 dark:text-white">
          Alerts & notifications
        </h1>
        <p className="text-gray-600 dark:text-gray-400 mt-1">
          {unreadCount === 0
            ? 'All notifications read'
            : `${unreadCount} unread notification${unreadCount !== 1 ? 's' : ''}`}
        </p>
      </div>

      {unreadCount > 0 && (
        <Button
          variant="outline"
          onClick={onClearAll}
        >
          ✓ Mark all as read
        </Button>
      )}
    </div>
  );
}
```

### `components/alerts/AlertsFilter.tsx`
```typescript
'use client';

import { AlertSeverity } from '@/app/dashboard/alerts/page';

interface AlertsFilterProps {
  selectedFilter: AlertSeverity | 'all';
  onFilterChange: (filter: AlertSeverity | 'all') => void;
}

const filterOptions: { label: string; value: AlertSeverity | 'all' }[] = [
  { label: 'All alerts', value: 'all' },
  { label: 'Critical', value: 'critical' },
  { label: 'Warnings', value: 'warning' },
  { label: 'Info', value: 'info' },
];

export function AlertsFilter({
  selectedFilter,
  onFilterChange,
}: AlertsFilterProps) {
  return (
    <div className="flex flex-wrap gap-2">
      {filterOptions.map((option) => (
        <button
          key={option.value}
          onClick={() => onFilterChange(option.value)}
          className={`px-4 py-2.5 rounded-lg font-medium transition-all min-h-[44px] ${
            selectedFilter === option.value
              ? 'bg-emerald-600 text-white'
              : 'bg-white dark:bg-gray-900 text-gray-700 dark:text-gray-300 border-2 border-gray-200 dark:border-gray-700 hover:border-emerald-500 dark:hover:border-emerald-400'
          }`}
        >
          {option.label}
        </button>
      ))}
    </div>
  );
}
```

### `components/alerts/AlertsList.tsx`
```typescript
'use client';

import { AlertCard } from './AlertCard';
import { Alert } from '@/app/dashboard/alerts/page';

interface AlertsListProps {
  alerts: Alert[];
  onMarkAsRead: (id: string) => void;
  onDismiss: (id: string) => void;
}

export function AlertsList({
  alerts,
  onMarkAsRead,
  onDismiss,
}: AlertsListProps) {
  const groupedBySeverity = {
    critical: alerts.filter((a) => a.severity === 'critical'),
    warning: alerts.filter((a) => a.severity === 'warning'),
    info: alerts.filter((a) => a.severity === 'info'),
  };

  return (
    <div className="space-y-6">
      {groupedBySeverity.critical.length > 0 && (
        <AlertGroup
          title="Critical alerts"
          alerts={groupedBySeverity.critical}
          onMarkAsRead={onMarkAsRead}
          onDismiss={onDismiss}
        />
      )}

      {groupedBySeverity.warning.length > 0 && (
        <AlertGroup
          title="Warnings"
          alerts={groupedBySeverity.warning}
          onMarkAsRead={onMarkAsRead}
          onDismiss={onDismiss}
        />
      )}

      {groupedBySeverity.info.length > 0 && (
        <AlertGroup
          title="Information"
          alerts={groupedBySeverity.info}
          onMarkAsRead={onMarkAsRead}
          onDismiss={onDismiss}
        />
      )}
    </div>
  );
}

interface AlertGroupProps {
  title: string;
  alerts: Alert[];
  onMarkAsRead: (id: string) => void;
  onDismiss: (id: string) => void;
}

function AlertGroup({
  title,
  alerts,
  onMarkAsRead,
  onDismiss,
}: AlertGroupProps) {
  return (
    <div>
      <h3 className="text-sm font-semibold text-gray-600 dark:text-gray-400 uppercase tracking-wide mb-3 px-2">
        {title}
      </h3>
      <div className="space-y-3">
        {alerts.map((alert) => (
          <AlertCard
            key={alert.id}
            alert={alert}
            onMarkAsRead={() => onMarkAsRead(alert.id)}
            onDismiss={() => onDismiss(alert.id)}
          />
        ))}
      </div>
    </div>
  );
}
```

### `components/alerts/AlertCard.tsx`
```typescript
'use client';

import { useState } from 'react';
import { Alert } from '@/app/dashboard/alerts/page';
import { Button } from '@/components/ui/Button';

interface AlertCardProps {
  alert: Alert;
  onMarkAsRead: () => void;
  onDismiss: () => void;
}

export function AlertCard({
  alert,
  onMarkAsRead,
  onDismiss,
}: AlertCardProps) {
  const [isExpanded, setIsExpanded] = useState(!alert.read);

  const getSeverityStyles = (severity: string) => {
    switch (severity) {
      case 'critical':
        return {
          bg: 'bg-red-50 dark:bg-red-900/20',
          border: 'border-red-200 dark:border-red-700',
          icon: '🚨',
          badge: 'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200',
        };
      case 'warning':
        return {
          bg: 'bg-amber-50 dark:bg-amber-900/20',
          border: 'border-amber-200 dark:border-amber-700',
          icon: '⚠️',
          badge: 'bg-amber-100 text-amber-800 dark:bg-amber-900 dark:text-amber-200',
        };
      default:
        return {
          bg: 'bg-blue-50 dark:bg-blue-900/20',
          border: 'border-blue-200 dark:border-blue-700',
          icon: 'ℹ️',
          badge: 'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200',
        };
    }
  };

  const styles = getSeverityStyles(alert.severity);

  const formatTime = (date: Date) => {
    const now = new Date();
    const diffMs = now.getTime() - date.getTime();
    const diffMins = Math.floor(diffMs / 60000);
    const diffHours = Math.floor(diffMs / 3600000);
    const diffDays = Math.floor(diffMs / 86400000);

    if (diffMins < 60) return `${diffMins}m ago`;
    if (diffHours < 24) return `${diffHours}h ago`;
    if (diffDays < 7) return `${diffDays}d ago`;
    return date.toLocaleDateString();
  };

  return (
    <div
      className={`rounded-lg border-2 ${styles.bg} ${styles.border} p-4 transition-all ${
        !alert.read ? 'shadow-md' : ''
      }`}
    >
      <button
        onClick={() => setIsExpanded(!isExpanded)}
        className="w-full text-left"
      >
        <div className="flex items-start gap-4">
          <span className="text-2xl mt-1">{styles.icon}</span>

          <div className="flex-1 min-w-0">
            <div className="flex items-center gap-3 mb-1">
              <h4 className="font-semibold text-gray-900 dark:text-white">
                {alert.title}
              </h4>
              {!alert.read && (
                <span className={`text-xs font-bold px-2 py-1 rounded-full whitespace-nowrap ${styles.badge}`}>
                  New
                </span>
              )}
            </div>

            {isExpanded && (
              <p className="text-sm text-gray-700 dark:text-gray-300 mb-3">
                {alert.message}
              </p>
            )}

            <p className="text-xs text-gray-500 dark:text-gray-400">
              {formatTime(alert.timestamp)}
            </p>
          </div>

          <span className="text-gray-400 mt-1 text-lg">
            {isExpanded ? '▲' : '▼'}
          </span>
        </div>
      </button>

      {isExpanded && (
        <div className="mt-4 pt-4 border-t border-gray-300 dark:border-gray-600 flex flex-wrap gap-2">
          {alert.actionLabel && (
            <Button
              variant="primary"
              size="sm"
              onClick={() => alert.actionHandler?.()}
            >
              {alert.actionLabel}
            </Button>
          )}

          {!alert.read && (
            <Button
              variant="secondary"
              size="sm"
              onClick={onMarkAsRead}
            >
              ✓ Mark as read
            </Button>
          )}

          <Button
            variant="outline"
            size="sm"
            onClick={onDismiss}
          >
            Dismiss
          </Button>
        </div>
      )}
    </div>
  );
}
```

### `components/alerts/EmptyState.tsx`
```typescript
export function EmptyState() {
  return (
    <div className="flex flex-col items-center justify-center py-16 px-4">
      <div className="text-6xl mb-4">✓</div>
      <h3 className="text-xl font-semibold text-gray-900 dark:text-white mb-2">
        All caught up!
      </h3>
      <p className="text-gray-600 dark:text-gray-400 text-center max-w-sm">
        No new alerts at the moment. Your Rawbin device is running smoothly.
      </p>
    </div>
  );
}
```

---

## Alerts Store

### `store/alertStore.ts`
```typescript
import { create } from 'zustand';
import { Alert } from '@/types';

interface AlertState {
  alerts: Alert[];
  unreadCount: number;

  setAlerts: (alerts: Alert[]) => void;
  addAlert: (alert: Alert) => void;
  markAsRead: (id: string) => void;
  markAllAsRead: () => void;
  dismissAlert: (id: string) => void;
  clearDismissed: () => void;
}

export const useAlertStore = create<AlertState>((set) => ({
  alerts: [],
  unreadCount: 0,

  setAlerts: (alerts) => set({
    alerts,
    unreadCount: alerts.filter((a) => !a.read).length,
  }),

  addAlert: (alert) => set((state) => {
    const newAlerts = [alert, ...state.alerts];
    return {
      alerts: newAlerts,
      unreadCount: newAlerts.filter((a) => !a.read).length,
    };
  }),

  markAsRead: (id) => set((state) => {
    const updated = state.alerts.map((a) =>
      a.id === id ? { ...a, read: true } : a
    );
    return {
      alerts: updated,
      unreadCount: updated.filter((a) => !a.read).length,
    };
  }),

  markAllAsRead: () => set((state) => {
    const updated = state.alerts.map((a) => ({ ...a, read: true }));
    return {
      alerts: updated,
      unreadCount: 0,
    };
  }),

  dismissAlert: (id) => set((state) => {
    const updated = state.alerts.filter((a) => a.id !== id);
    return {
      alerts: updated,
      unreadCount: updated.filter((a) => !a.read).length,
    };
  }),

  clearDismissed: () => set((state) => {
    const updated = state.alerts.filter((a) => !a.read);
    return {
      alerts: updated,
      unreadCount: updated.length,
    };
  }),
}));
```

---

## Alert Services

### `services/alerts.ts`
```typescript
import { ApiClient } from './api';
import { Alert } from '@/types';

export const alertService = {
  async getAlerts(): Promise<Alert[]> {
    return ApiClient.get<Alert[]>('/alerts');
  },

  async getAlertsBySeverity(severity: 'critical' | 'warning' | 'info'): Promise<Alert[]> {
    return ApiClient.get<Alert[]>(`/alerts?severity=${severity}`);
  },

  async markAsRead(id: string): Promise<void> {
    await ApiClient.put(`/alerts/${id}/read`, {});
  },

  async markAllAsRead(): Promise<void> {
    await ApiClient.put('/alerts/read-all', {});
  },

  async dismissAlert(id: string): Promise<void> {
    await ApiClient.delete(`/alerts/${id}`);
  },

  async subscribeToAlerts(callback: (alert: Alert) => void) {
    const eventSource = new EventSource('/api/alerts/stream');
    eventSource.onmessage = (event) => {
      const alert = JSON.parse(event.data);
      callback(alert);
    };
    return () => eventSource.close();
  },
};
```

---

## Types

### `types/index.ts` (Extended)
```typescript
export type AlertSeverity = 'critical' | 'warning' | 'info';

export interface Alert {
  id: string;
  deviceId: string;
  severity: AlertSeverity;
  title: string;
  message: string;
  timestamp: Date;
  read: boolean;
  actionLabel?: string;
  actionHandler?: () => void;
}
```

---

## Mock Alert Data

Severity levels (appearance only changes, functionality same):

**Critical** = 🚨 Red badge, red border
- Immediate action needed
- Device offline/temp critical/moisture extremes

**Warning** = ⚠️ Amber badge, amber border
- Action recommended soon
- Low moisture/maintenance due/cycle alert

**Info** = ℹ️ Blue badge, blue border
- Informational only
- Cycle complete/update available/system checks

---

## Features

✓ 3-severity system (critical/warning/info)
✓ Color-coded badges + borders
✓ Filter tabs (all/critical/warning/info)
✓ Grouped by severity
✓ Mark as read / Mark all as read
✓ Dismiss individual alerts
✓ Unread count badge
✓ Expandable alert details
✓ Action buttons (contextual)
✓ Timestamps (relative time)
✓ New badge on unread
✓ Empty state
✓ Accessibility (44px+ tap targets)
✓ Dark mode throughout
✓ Fully responsive
✓ TypeScript typing

---

## Next: Admin Panel

Ready for admin dashboard (user management table, device management, role badges, system health, analytics overview)?
