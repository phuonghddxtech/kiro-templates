# Push Notifications — React Native

> expo-notifications setup, permission handling, and navigation on tap.

## Setup

```bash
npx expo install expo-notifications expo-device expo-constants
```

### app.config.ts
```ts
export default {
  expo: {
    plugins: [
      [
        'expo-notifications',
        {
          icon: './assets/notification-icon.png',
          color: '#3B82F6',
          // Android channel
          defaultChannel: 'default',
        },
      ],
    ],
  },
};
```

---

## Register for Push Notifications

```ts
// services/notifications.service.ts
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import Constants from 'expo-constants';
import { Platform } from 'react-native';
import { logger } from '@/utils/logger';

// Configure foreground behavior
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

export async function registerForPushNotifications(): Promise<string | null> {
  // Must be physical device
  if (!Device.isDevice) {
    logger.warn({ event: 'push.not_device' });
    return null;
  }

  // Check existing permission
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  // Request permission if not granted
  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    logger.info({ event: 'push.permission_denied' });
    return null;
  }

  // Get Expo push token
  const projectId = Constants.expoConfig?.extra?.eas?.projectId;
  const tokenData = await Notifications.getExpoPushTokenAsync({ projectId });
  const token = tokenData.data;

  logger.info({ event: 'push.token_registered', token });

  // Android: create notification channel
  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'Default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
    });
  }

  return token;
}
```

---

## Handle Notification Events

```ts
// hooks/useNotifications.ts
import { useEffect, useRef } from 'react';
import * as Notifications from 'expo-notifications';
import { useRouter } from 'expo-router';

export function useNotifications() {
  const router = useRouter();
  const notificationListener = useRef<Notifications.Subscription>();
  const responseListener = useRef<Notifications.Subscription>();

  useEffect(() => {
    // Foreground: notification received while app is open
    notificationListener.current = Notifications.addNotificationReceivedListener(
      (notification) => {
        logger.info({
          event: 'push.received_foreground',
          title: notification.request.content.title,
        });
      },
    );

    // User tapped on notification → navigate
    responseListener.current = Notifications.addNotificationResponseReceivedListener(
      (response) => {
        const data = response.notification.request.content.data;
        handleNotificationNavigation(data, router);
      },
    );

    return () => {
      notificationListener.current?.remove();
      responseListener.current?.remove();
    };
  }, [router]);
}

function handleNotificationNavigation(
  data: Record<string, unknown>,
  router: ReturnType<typeof useRouter>,
) {
  // Navigate based on notification data
  switch (data.type) {
    case 'order':
      router.push(`/orders/${data.orderId}`);
      break;
    case 'message':
      router.push(`/messages/${data.chatId}`);
      break;
    default:
      router.push('/(tabs)');
  }
}
```

### Initialize in Root Layout
```tsx
// app/_layout.tsx
import { useNotifications } from '@/hooks/useNotifications';

export default function RootLayout() {
  useNotifications(); // Register listeners

  useEffect(() => {
    registerForPushNotifications().then((token) => {
      if (token) {
        // Send token to backend
        api.post('/users/push-token', { token });
      }
    });
  }, []);

  return <Slot />;
}
```

---

## Rules

### ✅ MUST
- **Register on physical device only** — simulators don't support push
- **Handle permission denied gracefully** — don't block app usage
- **Navigate on tap** — always handle notification response for deep navigation
- **Log all notification events** with structured logger
- **Android channels** — create channels for different notification types
- **Send token to backend** after registration

### ❌ NEVER
- Never assume permission is granted — always check first
- Never crash if notification permission is denied
- Never ignore notification tap — always navigate to relevant screen
