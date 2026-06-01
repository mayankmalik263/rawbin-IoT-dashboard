# Rawbin IoT Dashboard — Device Setup & Onboarding

Complete multi-step device pairing flow with QR scanner, manual entry, WiFi setup, and success states.

---

## Components

### `components/device-setup/DeviceSetupStepper.tsx`
```typescript
'use client';

import { useState } from 'react';
import { QRScanStep } from './steps/QRScanStep';
import { ManualEntryStep } from './steps/ManualEntryStep';
import { WiFiSetupStep } from './steps/WiFiSetupStep';
import { ConnectionStep } from './steps/ConnectionStep';
import { SuccessStep } from './steps/SuccessStep';
import { Card } from '@/components/ui/Card';
import { Button } from '@/components/ui/Button';

type Step = 'qr' | 'manual' | 'wifi' | 'connecting' | 'success';

interface StepperState {
  deviceId: string | null;
  deviceName: string | null;
  wifiSSID: string | null;
  wifiPassword: string | null;
  isConnecting: boolean;
  connectionProgress: number;
}

export function DeviceSetupStepper() {
  const [currentStep, setCurrentStep] = useState<Step>('qr');
  const [state, setState] = useState<StepperState>({
    deviceId: null,
    deviceName: null,
    wifiSSID: null,
    wifiPassword: null,
    isConnecting: false,
    connectionProgress: 0,
  });

  const steps: { id: Step; label: string; number: number }[] = [
    { id: 'qr', label: 'Scan QR Code', number: 1 },
    { id: 'wifi', label: 'WiFi Setup', number: 2 },
    { id: 'connecting', label: 'Connecting', number: 3 },
    { id: 'success', label: 'Complete', number: 4 },
  ];

  const handleQRSuccess = (deviceId: string) => {
    setState((prev) => ({ ...prev, deviceId, deviceName: `Rawbin ${deviceId.slice(0, 5)}` }));
    setCurrentStep('wifi');
  };

  const handleManualEntry = (deviceId: string) => {
    setState((prev) => ({ ...prev, deviceId, deviceName: `Rawbin ${deviceId.slice(0, 5)}` }));
    setCurrentStep('wifi');
  };

  const handleWiFiSetup = (ssid: string, password: string) => {
    setState((prev) => ({ ...prev, wifiSSID: ssid, wifiPassword: password }));
    setCurrentStep('connecting');
    simulateConnection();
  };

  const simulateConnection = () => {
    setState((prev) => ({ ...prev, isConnecting: true, connectionProgress: 0 }));
    let progress = 0;
    const interval = setInterval(() => {
      progress += Math.random() * 30;
      if (progress >= 100) {
        progress = 100;
        clearInterval(interval);
        setState((prev) => ({ ...prev, connectionProgress: 100 }));
        setTimeout(() => setCurrentStep('success'), 800);
      } else {
        setState((prev) => ({ ...prev, connectionProgress: progress }));
      }
    }, 500);
  };

  const handleReset = () => {
    setCurrentStep('qr');
    setState({
      deviceId: null,
      deviceName: null,
      wifiSSID: null,
      wifiPassword: null,
      isConnecting: false,
      connectionProgress: 0,
    });
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-white to-gray-50 dark:from-gray-950 dark:to-gray-900 p-4 flex flex-col items-center justify-center">
      <div className="w-full max-w-md">
        <div className="text-center mb-8">
          <h1 className="text-3xl font-bold text-gray-900 dark:text-white mb-2">
            Connect your device
          </h1>
          <p className="text-gray-600 dark:text-gray-400">
            Let's get your Rawbin composting unit online
          </p>
        </div>

        <StepperIndicator steps={steps} currentStep={currentStep} />

        <Card className="mt-8 shadow-lg">
          {currentStep === 'qr' && (
            <QRScanStep
              deviceName={state.deviceName}
              onSuccess={handleQRSSuccess}
              onManualEntry={() => setCurrentStep('manual')}
            />
          )}

          {currentStep === 'manual' && (
            <ManualEntryStep
              onSuccess={handleManualEntry}
              onBack={() => setCurrentStep('qr')}
            />
          )}

          {currentStep === 'wifi' && (
            <WiFiSetupStep
              deviceName={state.deviceName}
              onSuccess={handleWiFiSetup}
              onBack={() => setCurrentStep('qr')}
            />
          )}

          {currentStep === 'connecting' && (
            <ConnectionStep
              progress={state.connectionProgress}
              deviceName={state.deviceName}
            />
          )}

          {currentStep === 'success' && (
            <SuccessStep
              deviceName={state.deviceName}
              onComplete={handleReset}
            />
          )}
        </Card>
      </div>
    </div>
  );
}

interface StepperIndicatorProps {
  steps: { id: Step; label: string; number: number }[];
  currentStep: Step;
}

function StepperIndicator({ steps, currentStep }: StepperIndicatorProps) {
  const currentStepIndex = steps.findIndex((s) => s.id === currentStep);

  return (
    <div className="flex items-center gap-2">
      {steps.map((step, idx) => (
        <div key={step.id} className="flex items-center gap-2">
          <div
            className={`w-10 h-10 rounded-full flex items-center justify-center font-semibold text-sm transition-all ${
              idx <= currentStepIndex
                ? 'bg-emerald-600 text-white'
                : 'bg-gray-200 text-gray-600 dark:bg-gray-700 dark:text-gray-400'
            }`}
          >
            {idx < currentStepIndex ? '✓' : step.number}
          </div>
          {idx < steps.length - 1 && (
            <div
              className={`h-1 flex-grow transition-all ${
                idx < currentStepIndex
                  ? 'bg-emerald-600'
                  : 'bg-gray-200 dark:bg-gray-700'
              }`}
            />
          )}
        </div>
      ))}
    </div>
  );
}
```

---

## Step Components

### `components/device-setup/steps/QRScanStep.tsx`
```typescript
'use client';

import { Button } from '@/components/ui/Button';

interface QRScanStepProps {
  deviceName: string | null;
  onSuccess: (deviceId: string) => void;
  onManualEntry: () => void;
}

export function QRScanStep({
  deviceName,
  onSuccess,
  onManualEntry,
}: QRScanStepProps) {
  const handleScanClick = () => {
    const mockDeviceId = 'RB' + Math.random().toString(36).slice(2, 8).toUpperCase();
    onSuccess(mockDeviceId);
  };

  return (
    <div className="text-center space-y-6">
      <div>
        <h2 className="text-xl font-semibold text-gray-900 dark:text-white mb-2">
          Scan QR code
        </h2>
        <p className="text-sm text-gray-600 dark:text-gray-400">
          Point your phone camera at the QR code on your Rawbin device
        </p>
      </div>

      <div className="relative bg-gray-100 dark:bg-gray-800 rounded-lg aspect-square flex items-center justify-center border-2 border-dashed border-gray-300 dark:border-gray-600">
        <svg
          className="w-32 h-32 text-gray-400 dark:text-gray-500"
          fill="currentColor"
          viewBox="0 0 100 100"
        >
          <rect x="10" y="10" width="30" height="30" />
          <rect x="60" y="10" width="30" height="30" />
          <rect x="10" y="60" width="30" height="30" />
          <rect x="20" y="20" width="10" height="10" fill="white" />
          <rect x="70" y="20" width="10" height="10" fill="white" />
          <rect x="20" y="70" width="10" height="10" fill="white" />
          <rect x="45" y="45" width="10" height="10" />
        </svg>
      </div>

      <div className="space-y-3">
        <Button
          variant="primary"
          size="lg"
          className="w-full"
          onClick={handleScanClick}
        >
          📷 Open Camera
        </Button>

        <Button
          variant="outline"
          size="lg"
          className="w-full"
          onClick={onManualEntry}
        >
          Enter device ID manually
        </Button>
      </div>

      <div className="p-3 bg-blue-50 dark:bg-blue-900/20 rounded-lg text-sm text-blue-800 dark:text-blue-200">
        Can't find the QR code? Check the bottom or back of your device.
      </div>
    </div>
  );
}
```

### `components/device-setup/steps/ManualEntryStep.tsx`
```typescript
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';

interface ManualEntryStepProps {
  onSuccess: (deviceId: string) => void;
  onBack: () => void;
}

export function ManualEntryStep({ onSuccess, onBack }: ManualEntryStepProps) {
  const [deviceId, setDeviceId] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!deviceId.trim()) {
      setError('Please enter a device ID');
      return;
    }
    if (deviceId.length < 6) {
      setError('Device ID must be at least 6 characters');
      return;
    }
    onSuccess(deviceId.toUpperCase());
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <h2 className="text-xl font-semibold text-gray-900 dark:text-white mb-2">
          Enter device ID manually
        </h2>
        <p className="text-sm text-gray-600 dark:text-gray-400">
          Found on the back or bottom of your device
        </p>
      </div>

      <Input
        label="Device ID"
        placeholder="e.g., RB1A2B3C"
        value={deviceId}
        onChange={(e) => {
          setDeviceId(e.target.value);
          setError('');
        }}
        error={error}
      />

      <div className="p-3 bg-amber-50 dark:bg-amber-900/20 rounded-lg text-sm text-amber-800 dark:text-amber-200">
        Device IDs are case-insensitive and typically start with RB or RC.
      </div>

      <div className="flex gap-3 pt-4">
        <Button
          type="button"
          variant="secondary"
          size="lg"
          className="flex-1"
          onClick={onBack}
        >
          Back
        </Button>
        <Button
          type="submit"
          variant="primary"
          size="lg"
          className="flex-1"
        >
          Continue
        </Button>
      </div>
    </form>
  );
}
```

### `components/device-setup/steps/WiFiSetupStep.tsx`
```typescript
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';

interface WiFiSetupStepProps {
  deviceName: string | null;
  onSuccess: (ssid: string, password: string) => void;
  onBack: () => void;
}

export function WiFiSetupStep({
  deviceName,
  onSuccess,
  onBack,
}: WiFiSetupStepProps) {
  const [ssid, setSSID] = useState('');
  const [password, setPassword] = useState('');
  const [showPassword, setShowPassword] = useState(false);
  const [error, setError] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!ssid.trim()) {
      setError('Please enter WiFi network name');
      return;
    }
    if (!password.trim()) {
      setError('Please enter WiFi password');
      return;
    }
    onSuccess(ssid, password);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <h2 className="text-xl font-semibold text-gray-900 dark:text-white mb-1">
          Connect to WiFi
        </h2>
        <p className="text-sm text-gray-600 dark:text-gray-400">
          {deviceName} will use this network to sync data
        </p>
      </div>

      <Input
        label="Network name (SSID)"
        placeholder="Your WiFi network"
        value={ssid}
        onChange={(e) => {
          setSSID(e.target.value);
          setError('');
        }}
        error={error && !ssid ? error : ''}
      />

      <Input
        label="WiFi password"
        type={showPassword ? 'text' : 'password'}
        placeholder="••••••••"
        value={password}
        onChange={(e) => {
          setPassword(e.target.value);
          setError('');
        }}
        error={error && password ? error : ''}
      />

      <label className="flex items-center gap-2 cursor-pointer">
        <input
          type="checkbox"
          checked={showPassword}
          onChange={(e) => setShowPassword(e.target.checked)}
          className="w-4 h-4 rounded border-gray-300 text-emerald-600 focus:ring-emerald-500"
        />
        <span className="text-sm text-gray-600 dark:text-gray-400">
          Show password
        </span>
      </label>

      <div className="p-3 bg-green-50 dark:bg-green-900/20 rounded-lg text-sm text-green-800 dark:text-green-200">
        We support 2.4GHz networks. 5GHz networks require manual setup.
      </div>

      <div className="flex gap-3 pt-4">
        <Button
          type="button"
          variant="secondary"
          size="lg"
          className="flex-1"
          onClick={onBack}
        >
          Back
        </Button>
        <Button
          type="submit"
          variant="primary"
          size="lg"
          className="flex-1"
        >
          Connect
        </Button>
      </div>
    </form>
  );
}
```

### `components/device-setup/steps/ConnectionStep.tsx`
```typescript
'use client';

interface ConnectionStepProps {
  progress: number;
  deviceName: string | null;
}

export function ConnectionStep({
  progress,
  deviceName,
}: ConnectionStepProps) {
  const stages = [
    { label: 'Scanning WiFi', progress: 25 },
    { label: 'Authenticating', progress: 50 },
    { label: 'Connecting to cloud', progress: 75 },
    { label: 'Syncing data', progress: 100 },
  ];

  const currentStage = stages.find((s) => progress <= s.progress);

  return (
    <div className="text-center space-y-6">
      <div>
        <h2 className="text-xl font-semibold text-gray-900 dark:text-white mb-2">
          Connecting {deviceName}
        </h2>
        <p className="text-sm text-gray-600 dark:text-gray-400">
          This usually takes 30-60 seconds
        </p>
      </div>

      <div className="space-y-4">
        <div className="w-full bg-gray-200 dark:bg-gray-700 rounded-full h-2 overflow-hidden">
          <div
            className="bg-emerald-600 h-full rounded-full transition-all duration-300"
            style={{ width: `${Math.min(progress, 100)}%` }}
          />
        </div>

        <p className="text-2xl font-semibold text-emerald-600">
          {Math.round(progress)}%
        </p>

        <p className="text-sm text-gray-600 dark:text-gray-400">
          {currentStage?.label || 'Almost done'}
        </p>
      </div>

      <div className="space-y-2">
        {stages.map((stage, idx) => (
          <div
            key={idx}
            className={`flex items-center gap-3 p-2 rounded text-sm transition-all ${
              progress >= stage.progress
                ? 'text-emerald-600 font-medium'
                : 'text-gray-500 dark:text-gray-400'
            }`}
          >
            <span
              className={`w-5 h-5 rounded-full flex items-center justify-center text-xs font-bold ${
                progress >= stage.progress
                  ? 'bg-emerald-600 text-white'
                  : 'bg-gray-200 dark:bg-gray-700'
              }`}
            >
              {progress >= stage.progress ? '✓' : idx + 1}
            </span>
            {stage.label}
          </div>
        ))}
      </div>

      <div className="p-3 bg-blue-50 dark:bg-blue-900/20 rounded-lg text-xs text-blue-800 dark:text-blue-200">
        Don't close this page or unplug your device while connecting.
      </div>
    </div>
  );
}
```

### `components/device-setup/steps/SuccessStep.tsx`
```typescript
'use client';

import { Button } from '@/components/ui/Button';
import { useRouter } from 'next/navigation';

interface SuccessStepProps {
  deviceName: string | null;
  onComplete: () => void;
}

export function SuccessStep({ deviceName, onComplete }: SuccessStepProps) {
  const router = useRouter();

  const handleDashboard = () => {
    router.push('/dashboard');
  };

  return (
    <div className="text-center space-y-6">
      <div className="flex justify-center">
        <div className="w-16 h-16 bg-emerald-100 dark:bg-emerald-900/30 rounded-full flex items-center justify-center text-4xl">
          ✓
        </div>
      </div>

      <div>
        <h2 className="text-2xl font-bold text-gray-900 dark:text-white mb-2">
          Device connected!
        </h2>
        <p className="text-sm text-gray-600 dark:text-gray-400">
          {deviceName} is now online and ready to track your composting
        </p>
      </div>

      <div className="p-4 bg-emerald-50 dark:bg-emerald-900/20 rounded-lg space-y-2">
        <p className="text-sm font-medium text-emerald-900 dark:text-emerald-200">
          What's next
        </p>
        <ul className="text-sm text-emerald-800 dark:text-emerald-300 space-y-1 text-left">
          <li>✓ Add your first waste</li>
          <li>✓ Monitor real-time metrics</li>
          <li>✓ Track environmental impact</li>
        </ul>
      </div>

      <div className="space-y-3 pt-2">
        <Button
          variant="primary"
          size="lg"
          className="w-full"
          onClick={handleDashboard}
        >
          Go to dashboard
        </Button>

        <Button
          variant="secondary"
          size="lg"
          className="w-full"
          onClick={onComplete}
        >
          Add another device
        </Button>
      </div>
    </div>
  );
}
```

---

## Layout Integration

### `app/device-setup/page.tsx`
```typescript
'use client';

import { useAuth } from '@/hooks/useAuth';
import { useRouter } from 'next/navigation';
import { useEffect } from 'react';
import { DeviceSetupStepper } from '@/components/device-setup/DeviceSetupStepper';

export default function DeviceSetupPage() {
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

  return <DeviceSetupStepper />;
}
```

---

## Integration with API

### `services/devices.ts` (Extended)
```typescript
export const deviceService = {
  // Existing methods...

  async pairDevice(deviceId: string, name: string): Promise<Device> {
    return ApiClient.post<Device>('/devices/pair', {
      deviceId,
      name,
      pairedAt: new Date(),
    });
  },

  async setupWiFi(
    deviceId: string,
    ssid: string,
    password: string
  ): Promise<{ status: string }> {
    return ApiClient.post('/devices/wifi-setup', {
      deviceId,
      ssid,
      password,
    });
  },

  async pollConnectionStatus(deviceId: string): Promise<{
    connected: boolean;
    lastSeen: Date;
  }> {
    return ApiClient.get(`/devices/${deviceId}/status`);
  },
};
```

---

## Hooks for Device Setup

### `hooks/useDeviceSetup.ts`
```typescript
import { useState } from 'react';
import { deviceService } from '@/services/devices';
import { useDeviceStore } from '@/store/deviceStore';

export function useDeviceSetup() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const { addDevice } = useDeviceStore();

  const pairDevice = async (deviceId: string, name: string) => {
    setIsLoading(true);
    setError(null);
    try {
      const device = await deviceService.pairDevice(deviceId, name);
      addDevice(device);
      return device;
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Pairing failed';
      setError(message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  const setupWiFi = async (deviceId: string, ssid: string, password: string) => {
    setIsLoading(true);
    setError(null);
    try {
      return await deviceService.setupWiFi(deviceId, ssid, password);
    } catch (err) {
      const message = err instanceof Error ? err.message : 'WiFi setup failed';
      setError(message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  return {
    pairDevice,
    setupWiFi,
    isLoading,
    error,
  };
}
```

---

## Mock Device Data

When testing, mock `POST /api/devices/pair`:
```json
{
  "id": "device_001",
  "name": "Rawbin Unit #1",
  "status": "active",
  "location": "Home - Kitchen",
  "temperature": 62,
  "moisture": 45,
  "wasteProcessed": 0,
  "compostGenerated": 0,
  "carbonAvoided": 0,
  "lastSync": "2026-06-01T14:30:00Z"
}
```

Mock `POST /api/devices/wifi-setup`:
```json
{
  "status": "connecting",
  "message": "Attempting WiFi connection"
}
```

---

## Features

✓ QR code scanner placeholder (ready for jsQR lib)
✓ Manual device ID entry with validation
✓ Multi-step progress indicator
✓ WiFi credential form with show/hide password
✓ Real-time connection progress animation
✓ Success state with next steps
✓ Error handling & validation
✓ Dark mode support
✓ Responsive mobile-first design
✓ Accessibility (44px+ tap targets)
✓ TypeScript typing throughout

---

## Next: Main Dashboard

Ready to build the core dashboard (metrics cards, charts, sidebar navigation)?
