# Rawbin IoT Dashboard — Auth + Shared Setup

## Project Structure

```
rawbin-dashboard/
├── app/
│   ├── layout.tsx
│   ├── page.tsx (Auth screen)
│   ├── dashboard/
│   │   └── page.tsx (Main dashboard - future)
│   ├── analytics/
│   │   └── page.tsx (Analytics - future)
│   ├── alerts/
│   │   └── page.tsx (Alerts - future)
│   └── admin/
│       └── page.tsx (Admin - future)
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   ├── Toast.tsx
│   │   └── Badge.tsx
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── ForgotPasswordForm.tsx
│   │   └── AuthLayout.tsx
│   ├── layout/
│   │   ├── Sidebar.tsx
│   │   ├── Navbar.tsx
│   │   └── ThemeToggle.tsx
│   └── charts/
│       └── ChartContainer.tsx
├── store/
│   ├── authStore.ts
│   ├── themeStore.ts
│   └── deviceStore.ts
├── services/
│   ├── api.ts
│   ├── auth.ts
│   └── devices.ts
├── types/
│   ├── index.ts
│   ├── auth.ts
│   └── device.ts
├── hooks/
│   ├── useAuth.ts
│   ├── useTheme.ts
│   └── useFetch.ts
├── constants/
│   └── index.ts
├── utils/
│   ├── cn.ts
│   └── format.ts
├── styles/
│   └── globals.css
├── tailwind.config.ts
├── tsconfig.json
└── next.config.ts
```

---

## 1. Types & Interfaces

### `types/index.ts`
```typescript
export type UserRole = 'admin' | 'user' | 'analyst';
export type DeviceStatus = 'active' | 'offline' | 'warning' | 'maintenance';
export type AlertSeverity = 'critical' | 'warning' | 'info';

export interface User {
  id: string;
  email: string;
  name: string;
  role: UserRole;
  avatar?: string;
  createdAt: Date;
}

export interface Device {
  id: string;
  name: string;
  status: DeviceStatus;
  location: string;
  userId: string;
  temperature: number;
  moisture: number;
  wasteProcessed: number;
  compostGenerated: number;
  carbonAvoided: number;
  lastSync: Date;
}

export interface Alert {
  id: string;
  deviceId: string;
  severity: AlertSeverity;
  title: string;
  message: string;
  timestamp: Date;
  read: boolean;
}

export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
}
```

### `types/auth.ts`
```typescript
export interface LoginRequest {
  email: string;
  password: string;
  rememberMe?: boolean;
}

export interface LoginResponse {
  token: string;
  refreshToken: string;
  user: {
    id: string;
    email: string;
    name: string;
    role: string;
  };
}

export interface ForgotPasswordRequest {
  email: string;
}

export interface ResetPasswordRequest {
  token: string;
  newPassword: string;
}
```

### `types/device.ts`
```typescript
export interface DeviceMetrics {
  temperature: number;
  moisture: number;
  co2Level: number;
  nitrogen: number;
  ph: number;
}

export interface CompostingCycle {
  id: string;
  startDate: Date;
  estimatedEndDate: Date;
  progress: number; // 0-100
  status: 'active' | 'completed' | 'paused';
}
```

---

## 2. Zustand Stores

### `store/authStore.ts`
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { User } from '@/types';

interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  error: string | null;
  isAuthenticated: boolean;

  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  setUser: (user: User | null) => void;
  clearError: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isLoading: false,
      error: null,
      isAuthenticated: false,

      login: async (email: string, password: string) => {
        set({ isLoading: true, error: null });
        try {
          const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
          });

          if (!response.ok) {
            throw new Error('Login failed');
          }

          const data = await response.json();
          set({
            user: data.user,
            token: data.token,
            isAuthenticated: true,
            isLoading: false,
          });
          localStorage.setItem('auth_token', data.token);
        } catch (err) {
          set({
            error: err instanceof Error ? err.message : 'Login failed',
            isLoading: false,
          });
          throw err;
        }
      },

      logout: () => {
        set({
          user: null,
          token: null,
          isAuthenticated: false,
          error: null,
        });
        localStorage.removeItem('auth_token');
      },

      setUser: (user) => set({ user }),
      clearError: () => set({ error: null }),
    }),
    {
      name: 'auth-storage',
    }
  )
);
```

### `store/themeStore.ts`
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

type Theme = 'light' | 'dark' | 'system';

interface ThemeState {
  theme: Theme;
  setTheme: (theme: Theme) => void;
}

export const useThemeStore = create<ThemeState>()(
  persist(
    (set) => ({
      theme: 'system' as Theme,
      setTheme: (theme) => {
        set({ theme });
        if (theme === 'dark') {
          document.documentElement.classList.add('dark');
        } else if (theme === 'light') {
          document.documentElement.classList.remove('dark');
        }
      },
    }),
    {
      name: 'theme-storage',
    }
  )
);
```

### `store/deviceStore.ts`
```typescript
import { create } from 'zustand';
import { Device, Alert } from '@/types';

interface DeviceState {
  devices: Device[];
  selectedDevice: Device | null;
  alerts: Alert[];
  isLoading: boolean;

  setDevices: (devices: Device[]) => void;
  selectDevice: (device: Device) => void;
  addDevice: (device: Device) => void;
  removeDevice: (id: string) => void;
  updateDevice: (id: string, data: Partial<Device>) => void;
  setAlerts: (alerts: Alert[]) => void;
  addAlert: (alert: Alert) => void;
}

export const useDeviceStore = create<DeviceState>((set) => ({
  devices: [],
  selectedDevice: null,
  alerts: [],
  isLoading: false,

  setDevices: (devices) => set({ devices }),
  selectDevice: (device) => set({ selectedDevice: device }),
  addDevice: (device) => set((state) => ({
    devices: [...state.devices, device],
  })),
  removeDevice: (id) => set((state) => ({
    devices: state.devices.filter((d) => d.id !== id),
  })),
  updateDevice: (id, data) => set((state) => ({
    devices: state.devices.map((d) =>
      d.id === id ? { ...d, ...data } : d
    ),
  })),
  setAlerts: (alerts) => set({ alerts }),
  addAlert: (alert) => set((state) => ({
    alerts: [alert, ...state.alerts],
  })),
}));
```

---

## 3. API Services

### `services/api.ts`
```typescript
import { ApiResponse } from '@/types';

const API_BASE = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000/api';

export class ApiClient {
  static async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${API_BASE}${endpoint}`;
    const token = localStorage.getItem('auth_token');

    const headers: HeadersInit = {
      'Content-Type': 'application/json',
      ...options.headers,
    };

    if (token) {
      headers['Authorization'] = `Bearer ${token}`;
    }

    const response = await fetch(url, {
      ...options,
      headers,
    });

    if (!response.ok) {
      throw new Error(`API Error: ${response.statusText}`);
    }

    return response.json();
  }

  static get<T>(endpoint: string) {
    return this.request<T>(endpoint, { method: 'GET' });
  }

  static post<T>(endpoint: string, data?: unknown) {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  static put<T>(endpoint: string, data?: unknown) {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  static delete<T>(endpoint: string) {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}
```

### `services/auth.ts`
```typescript
import { ApiClient } from './api';
import { LoginRequest, LoginResponse } from '@/types/auth';

export const authService = {
  async login(credentials: LoginRequest): Promise<LoginResponse> {
    return ApiClient.post<LoginResponse>('/auth/login', credentials);
  },

  async logout(): Promise<void> {
    await ApiClient.post('/auth/logout');
  },

  async forgotPassword(email: string): Promise<{ message: string }> {
    return ApiClient.post('/auth/forgot-password', { email });
  },

  async resetPassword(token: string, newPassword: string): Promise<{ message: string }> {
    return ApiClient.post('/auth/reset-password', { token, newPassword });
  },

  async getCurrentUser() {
    return ApiClient.get('/auth/me');
  },
};
```

### `services/devices.ts`
```typescript
import { ApiClient } from './api';
import { Device, Alert } from '@/types';

export const deviceService = {
  async getDevices(): Promise<Device[]> {
    return ApiClient.get<Device[]>('/devices');
  },

  async getDevice(id: string): Promise<Device> {
    return ApiClient.get<Device>(`/devices/${id}`);
  },

  async createDevice(data: Partial<Device>): Promise<Device> {
    return ApiClient.post<Device>('/devices', data);
  },

  async updateDevice(id: string, data: Partial<Device>): Promise<Device> {
    return ApiClient.put<Device>(`/devices/${id}`, data);
  },

  async deleteDevice(id: string): Promise<void> {
    await ApiClient.delete(`/devices/${id}`);
  },

  async getAlerts(): Promise<Alert[]> {
    return ApiClient.get<Alert[]>('/alerts');
  },

  async markAlertAsRead(id: string): Promise<void> {
    await ApiClient.put(`/alerts/${id}/read`, {});
  },
};
```

---

## 4. Hooks

### `hooks/useAuth.ts`
```typescript
import { useAuthStore } from '@/store/authStore';
import { useRouter } from 'next/navigation';

export function useAuth() {
  const router = useRouter();
  const { user, isAuthenticated, login, logout } = useAuthStore();

  const handleLogout = () => {
    logout();
    router.push('/');
  };

  return {
    user,
    isAuthenticated,
    login,
    logout: handleLogout,
  };
}
```

### `hooks/useTheme.ts`
```typescript
import { useThemeStore } from '@/store/themeStore';
import { useEffect, useState } from 'react';

export function useTheme() {
  const { theme, setTheme } = useThemeStore();
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  const isDark =
    mounted &&
    (theme === 'dark' || (theme === 'system' && window.matchMedia('(prefers-color-scheme: dark)').matches));

  return {
    theme,
    setTheme,
    isDark,
  };
}
```

### `hooks/useFetch.ts`
```typescript
import { useState, useEffect } from 'react';

interface UseFetchOptions {
  skip?: boolean;
  dependencies?: unknown[];
}

export function useFetch<T>(
  url: string,
  options?: UseFetchOptions
) {
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(!options?.skip);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    if (options?.skip) return;

    const fetchData = async () => {
      try {
        setIsLoading(true);
        const response = await fetch(url);
        if (!response.ok) throw new Error(response.statusText);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err instanceof Error ? err : new Error('Unknown error'));
      } finally {
        setIsLoading(false);
      }
    };

    fetchData();
  }, options?.dependencies || [url]);

  return { data, isLoading, error };
}
```

---

## 5. UI Components

### `components/ui/Button.tsx`
```typescript
import { ReactNode } from 'react';

interface ButtonProps {
  children: ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  type?: 'button' | 'submit' | 'reset';
  className?: string;
}

export function Button({
  children,
  onClick,
  variant = 'primary',
  size = 'md',
  disabled = false,
  type = 'button',
  className = '',
}: ButtonProps) {
  const baseStyles = 'font-medium rounded-lg transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';

  const variants = {
    primary: 'bg-emerald-600 text-white hover:bg-emerald-700 focus:ring-emerald-500 disabled:bg-gray-300',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-400 disabled:bg-gray-200',
    outline: 'border-2 border-emerald-600 text-emerald-600 hover:bg-emerald-50 focus:ring-emerald-500 disabled:border-gray-300 disabled:text-gray-300',
  };

  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2.5 text-base',
    lg: 'px-6 py-3 text-lg',
  };

  return (
    <button
      type={type}
      onClick={onClick}
      disabled={disabled}
      className={`${baseStyles} ${variants[variant]} ${sizes[size]} ${className}`}
    >
      {children}
    </button>
  );
}
```

### `components/ui/Input.tsx`
```typescript
interface InputProps {
  type?: string;
  placeholder?: string;
  value?: string;
  onChange?: (e: React.ChangeEvent<HTMLInputElement>) => void;
  disabled?: boolean;
  error?: string;
  label?: string;
  icon?: ReactNode;
  className?: string;
}

export function Input({
  type = 'text',
  placeholder,
  value,
  onChange,
  disabled,
  error,
  label,
  icon,
  className = '',
}: InputProps) {
  return (
    <div className="w-full">
      {label && (
        <label className="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
          {label}
        </label>
      )}
      <div className="relative">
        {icon && <div className="absolute left-3 top-3 text-gray-400">{icon}</div>}
        <input
          type={type}
          placeholder={placeholder}
          value={value}
          onChange={onChange}
          disabled={disabled}
          className={`w-full px-4 py-2.5 border-2 border-gray-200 rounded-lg focus:outline-none focus:border-emerald-500 focus:ring-2 focus:ring-emerald-200 disabled:bg-gray-100 disabled:cursor-not-allowed transition-colors ${
            icon ? 'pl-10' : ''
          } ${error ? 'border-red-500' : ''} ${className}`}
        />
      </div>
      {error && <p className="text-red-500 text-sm mt-1">{error}</p>}
    </div>
  );
}
```

### `components/ui/Card.tsx`
```typescript
import { ReactNode } from 'react';

interface CardProps {
  children: ReactNode;
  className?: string;
  hover?: boolean;
}

export function Card({ children, className = '', hover = false }: CardProps) {
  return (
    <div
      className={`bg-white dark:bg-gray-900 rounded-lg border border-gray-200 dark:border-gray-800 p-6 ${
        hover ? 'hover:shadow-lg hover:border-gray-300 dark:hover:border-gray-700 transition-all' : 'shadow-sm'
      } ${className}`}
    >
      {children}
    </div>
  );
}
```

### `components/ui/Modal.tsx`
```typescript
import { ReactNode } from 'react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: ReactNode;
}

export function Modal({ isOpen, onClose, title, children }: ModalProps) {
  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div className="bg-white dark:bg-gray-900 rounded-lg max-w-md w-full mx-4 p-6">
        <h2 className="text-lg font-semibold text-gray-900 dark:text-white mb-4">{title}</h2>
        {children}
        <button
          onClick={onClose}
          className="mt-6 w-full px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-900 dark:text-white rounded-lg hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors"
        >
          Close
        </button>
      </div>
    </div>
  );
}
```

### `components/ui/Badge.tsx`
```typescript
interface BadgeProps {
  label: string;
  variant?: 'success' | 'warning' | 'danger' | 'info';
  size?: 'sm' | 'md';
}

export function Badge({ label, variant = 'info', size = 'sm' }: BadgeProps) {
  const variants = {
    success: 'bg-emerald-100 text-emerald-800 dark:bg-emerald-900 dark:text-emerald-200',
    warning: 'bg-amber-100 text-amber-800 dark:bg-amber-900 dark:text-amber-200',
    danger: 'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200',
    info: 'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200',
  };

  const sizes = {
    sm: 'px-2 py-1 text-xs',
    md: 'px-3 py-1.5 text-sm',
  };

  return (
    <span className={`inline-block rounded-full font-medium ${variants[variant]} ${sizes[size]}`}>
      {label}
    </span>
  );
}
```

---

## 6. Auth Components

### `components/auth/LoginForm.tsx`
```typescript
'use client';

import { useState } from 'react';
import { useAuth } from '@/hooks/useAuth';
import { useRouter } from 'next/navigation';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';

export function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [rememberMe, setRememberMe] = useState(false);
  const [showPassword, setShowPassword] = useState(false);
  const [error, setError] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const { login } = useAuth();
  const router = useRouter();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setIsLoading(true);

    try {
      await login(email, password);
      router.push('/dashboard');
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Login failed');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <Input
        label="Email address"
        type="email"
        placeholder="your@email.com"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        required
      />

      <Input
        label="Password"
        type={showPassword ? 'text' : 'password'}
        placeholder="••••••••"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        required
      />

      <div className="flex items-center justify-between">
        <label className="flex items-center gap-2 cursor-pointer">
          <input
            type="checkbox"
            checked={rememberMe}
            onChange={(e) => setRememberMe(e.target.checked)}
            className="w-4 h-4 rounded border-gray-300 text-emerald-600 focus:ring-emerald-500"
          />
          <span className="text-sm text-gray-600 dark:text-gray-400">Remember me</span>
        </label>
        <a href="#" className="text-sm text-emerald-600 hover:text-emerald-700 font-medium">
          Forgot password?
        </a>
      </div>

      {error && <div className="p-3 bg-red-100 text-red-800 rounded-lg text-sm">{error}</div>}

      <Button
        type="submit"
        variant="primary"
        size="lg"
        className="w-full"
        disabled={isLoading}
      >
        {isLoading ? 'Signing in...' : 'Sign in'}
      </Button>
    </form>
  );
}
```

---

## 7. Main Auth Page

### `app/page.tsx`
```typescript
'use client';

import { LoginForm } from '@/components/auth/LoginForm';
import { Card } from '@/components/ui/Card';
import { ThemeToggle } from '@/components/layout/ThemeToggle';

export default function Home() {
  return (
    <div className="min-h-screen bg-gradient-to-br from-white to-gray-50 dark:from-gray-950 dark:to-gray-900 flex items-center justify-center p-4">
      <div className="absolute top-6 right-6">
        <ThemeToggle />
      </div>

      <div className="w-full max-w-md">
        <Card className="shadow-lg">
          <div className="text-center mb-8">
            <h1 className="text-3xl font-bold text-gray-900 dark:text-white mb-2">
              Rawbin
            </h1>
            <p className="text-gray-600 dark:text-gray-400">
              Composting made smart. Sustainability made simple.
            </p>
          </div>

          <LoginForm />

          <div className="mt-6 pt-6 border-t border-gray-200 dark:border-gray-800 text-center text-sm text-gray-600 dark:text-gray-400">
            Don't have an account?{' '}
            <a href="#" className="text-emerald-600 hover:text-emerald-700 font-medium">
              Sign up here
            </a>
          </div>
        </Card>

        <p className="text-center text-xs text-gray-500 dark:text-gray-500 mt-8">
          Protected by enterprise security. We never share your data.
        </p>
      </div>
    </div>
  );
}
```

### `components/layout/ThemeToggle.tsx`
```typescript
'use client';

import { useTheme } from '@/hooks/useTheme';

export function ThemeToggle() {
  const { theme, setTheme, isDark } = useTheme();

  return (
    <button
      onClick={() => setTheme(isDark ? 'light' : 'dark')}
      className="p-2 rounded-lg bg-gray-200 dark:bg-gray-800 text-gray-900 dark:text-yellow-400 hover:bg-gray-300 dark:hover:bg-gray-700 transition-colors"
      aria-label="Toggle theme"
    >
      {isDark ? '☀️' : '🌙'}
    </button>
  );
}
```

---

## 8. Configuration Files

### `tailwind.config.ts`
```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: 'class',
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        emerald: {
          50: '#f0fdf4',
          100: '#dcfce7',
          200: '#bbf7d0',
          300: '#86efac',
          400: '#4ade80',
          500: '#22c55e',
          600: '#16a34a',
          700: '#15803d',
          800: '#166534',
          900: '#145231',
        },
      },
      fontFamily: {
        sans: ['var(--font-sans)', 'system-ui'],
      },
    },
  },
  plugins: [],
};

export default config;
```

### `tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "resolveJsonModule": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["app/**/*.ts", "app/**/*.tsx", "components/**/*.ts", "components/**/*.tsx"],
  "exclude": ["node_modules"]
}
```

### `.env.local`
```
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

---

## 9. Installation & Setup

### Package dependencies
```bash
npm install next react react-dom zustand typescript tailwindcss postcss autoprefixer
npm install -D @types/react @types/node
```

### Next.js config
```typescript
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  reactStrictMode: true,
};

export default nextConfig;
```

---

## Mock API Responses

When testing locally, mock endpoints return:

**POST /api/auth/login**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "user_123",
    "email": "mayank@rawbin.com",
    "name": "Mayank Malik",
    "role": "admin"
  }
}
```

**GET /api/devices**
```json
[
  {
    "id": "device_001",
    "name": "Rawbin Unit #1",
    "status": "active",
    "location": "Rohtak, Haryana",
    "temperature": 62,
    "moisture": 45,
    "wasteProcessed": 145,
    "compostGenerated": 45,
    "carbonAvoided": 12.5,
    "lastSync": "2026-06-01T14:30:00Z"
  }
]
```

---

## Next Steps

Ship this. Build Device Setup → Main Dashboard → Analytics → Alerts → Admin in order.

Each screen builds on these foundations. Ready for Screen 2?
