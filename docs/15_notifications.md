---
title: Notifications
layout: default
nav_order: 15
---

# Notifications with Firebase

Hey team! Let's talk about implementing push notifications in our React applications using Firebase Cloud Messaging (FCM). This guide will help you set up notifications that work across web browsers and provide a great user experience.

## Overview

Firebase Cloud Messaging (FCM) is Google's solution for sending messages and notifications to users across different platforms. For web applications, it provides a reliable way to send push notifications even when the browser is closed.

## Setup

### 1. Firebase Project Configuration

First, you'll need to set up a Firebase project:

1. Go to the [Firebase Console](https://console.firebase.google.com/)
2. Create a new project or select an existing one
3. Add a web app to your project
4. Download the configuration file (`firebaseConfig`)

### 2. Install Dependencies

```bash
npm install firebase
# or
yarn add firebase
```

### 3. Initialize Firebase

Create a `firebase.js` file in your project:

```javascript
import { initializeApp } from 'firebase/app';
import { getMessaging, getToken, onMessage } from 'firebase/messaging';

const firebaseConfig = {
  apiKey: "your-api-key",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "your-app-id"
};

const app = initializeApp(firebaseConfig);
const messaging = getMessaging(app);

export { messaging, getToken, onMessage };
```

### 4. Service Worker

Create a `firebase-messaging-sw.js` file in your `public` folder:

```javascript
importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js');
importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-messaging-compat.js');

firebase.initializeApp({
  apiKey: "your-api-key",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "your-app-id"
});

const messaging = firebase.messaging();

messaging.onBackgroundMessage((payload) => {
  console.log('Received background message:', payload);
  
  const notificationTitle = payload.notification.title;
  const notificationOptions = {
    body: payload.notification.body,
    icon: '/logo192.png'
  };

  self.registration.showNotification(notificationTitle, notificationOptions);
});
```

## Implementation

### Requesting Permission

```javascript
import { getToken } from 'firebase/messaging';
import { messaging } from './firebase';

const requestNotificationPermission = async () => {
  try {
    const permission = await Notification.requestPermission();
    
    if (permission === 'granted') {
      const token = await getToken(messaging, {
        vapidKey: 'YOUR_VAPID_KEY'
      });
      
      // Send this token to your server
      console.log('FCM Token:', token);
      return token;
    }
  } catch (error) {
    console.error('Error getting notification permission:', error);
  }
};
```

### Handling Foreground Messages

```javascript
import { onMessage } from 'firebase/messaging';
import { messaging } from './firebase';

const handleForegroundMessage = () => {
  onMessage(messaging, (payload) => {
    console.log('Message received in foreground:', payload);
    
    // Show a custom notification or update UI
    const notification = new Notification(payload.notification.title, {
      body: payload.notification.body,
      icon: '/logo192.png'
    });
  });
};
```

### React Hook Example

```javascript
import { useState, useEffect } from 'react';
import { getToken, onMessage } from 'firebase/messaging';
import { messaging } from './firebase';

export const useNotifications = () => {
  const [token, setToken] = useState(null);
  const [permission, setPermission] = useState('default');

  useEffect(() => {
    const initializeNotifications = async () => {
      try {
        const currentPermission = await Notification.requestPermission();
        setPermission(currentPermission);

        if (currentPermission === 'granted') {
          const fcmToken = await getToken(messaging, {
            vapidKey: 'YOUR_VAPID_KEY'
          });
          setToken(fcmToken);
        }
      } catch (error) {
        console.error('Error initializing notifications:', error);
      }
    };

    initializeNotifications();
  }, []);

  useEffect(() => {
    const unsubscribe = onMessage(messaging, (payload) => {
      console.log('Foreground message received:', payload);
      // Handle foreground messages here
    });

    return () => unsubscribe();
  }, []);

  return { token, permission };
};
```

## Best Practices

### 1. Permission Handling

- Always request permission explicitly
- Handle all permission states (granted, denied, default)
- Provide clear explanations of why notifications are needed

### 2. Token Management

- Store FCM tokens securely
- Update tokens when they refresh
- Send tokens to your backend for server-side notifications

### 3. User Experience

- Show notifications only when relevant
- Allow users to customize notification preferences
- Provide easy ways to disable notifications

### 4. Error Handling

```javascript
const handleNotificationError = (error) => {
  switch (error.code) {
    case 'messaging/permission-blocked':
      console.log('User blocked notifications');
      break;
    case 'messaging/permission-default':
      console.log('User hasn\'t made a choice yet');
      break;
    default:
      console.error('Notification error:', error);
  }
};
```

## Testing

### Local Testing

1. Use Firebase Console to send test messages
2. Test both foreground and background scenarios
3. Verify token generation and storage

### Production Testing

- Test on different browsers
- Verify notification delivery
- Monitor error rates and user engagement

## Useful Links

- [Firebase Cloud Messaging Documentation](https://firebase.google.com/docs/cloud-messaging)
- [Web Push Protocol](https://tools.ietf.org/html/rfc8030)
- [Notification API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Notification)
- [Firebase Console](https://console.firebase.google.com/)

## Example Component

```javascript
import React, { useState, useEffect } from 'react';
import { useNotifications } from './hooks/useNotifications';

const NotificationComponent = () => {
  const { token, permission } = useNotifications();
  const [isSubscribed, setIsSubscribed] = useState(false);

  const handleSubscribe = async () => {
    if (permission === 'granted' && token) {
      try {
        // Send token to your backend
        await fetch('/api/notifications/subscribe', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ token })
        });
        setIsSubscribed(true);
      } catch (error) {
        console.error('Error subscribing to notifications:', error);
      }
    }
  };

  return (
    <div>
      <h3>Notifications</h3>
      {permission === 'default' && (
        <button onClick={() => Notification.requestPermission()}>
          Enable Notifications
        </button>
      )}
      {permission === 'granted' && !isSubscribed && (
        <button onClick={handleSubscribe}>
          Subscribe to Notifications
        </button>
      )}
      {isSubscribed && <p>✅ Subscribed to notifications!</p>}
    </div>
  );
};

export default NotificationComponent;
```

---

**Last updated: July 10, 2025**

Remember to replace placeholder values (API keys, project IDs, etc.) with your actual Firebase configuration. Keep your VAPID key secure and never expose it in client-side code that's publicly accessible. 