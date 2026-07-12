# Mobile Architecture

**Project:** [PROJECT_NAME]  
**Platforms:** [iOS/Android/Both]  
**Framework:** [React Native/Flutter/NativeScript/Native/Hybrid]  
**Last Updated:** [DATE]  
**Next Review:** [DATE]

---

## Overview

This document defines mobile architecture standards for [PROJECT_NAME]. Mobile apps must be designed with mobile-first principles and handle device constraints effectively.

**Core Philosophy:** Mobile-First Design + Multi-Platform Support + Offline Capability

---

## Table of Contents

1. [Mobile-First Design Approach](#mobile-first-design-approach)
2. [Platform Support Strategy](#platform-support-strategy)
3. [Device Constraints](#device-constraints)
4. [Offline-First Architecture](#offline-first-architecture)
5. [Real-Time Synchronization](#real-time-synchronization)
6. [Mobile API Design](#mobile-api-design)
7. [Native Integration](#native-integration)
8. [Performance Optimization](#performance-optimization)
9. [Testing Strategy](#testing-strategy)
10. [Deployment & Distribution](#deployment--distribution)

---

## Mobile-First Design Approach

### Principle

**Design for the most constrained platform first (mobile), then scale up.**

Mobile devices have:
- Smallest screens (320px - 480px width)
- Slowest networks (3G/4G, high latency)
- Limited battery (power consumption matters)
- Limited storage (disk space finite)
- Variable connectivity (frequent offline)

### Design Order

```
1. Mobile (Most constrained)
   ↓
2. Tablet (Medium constraints)
   ↓
3. Desktop (Least constrained)
```

**NOT the reverse!**

### Mobile-First Breakpoints

```css
/* Start with mobile */
.container {
  width: 100%;
  padding: 1rem;
  font-size: 16px;
}

/* Scale UP for larger screens */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
    font-size: 18px;
  }
}

@media (min-width: 1024px) {
  .container {
    padding: 3rem;
    font-size: 20px;
  }
}
```

### Touch-First Interaction

**Mobile users interact with fingers, not mice.**

```
❌ WRONG - Hover-based UX
button:hover { background: blue; }  // Doesn't work on mobile!

✅ RIGHT - Touch-friendly UX
button { background: blue; }        // Always visible
button:active { opacity: 0.8; }     // Feedback on tap
```

**Touch Target Sizes:**
- Minimum: 44x44 pixels (Apple Human Interface Guidelines)
- Safe: 48x48 pixels
- Spacing: 8px minimum between targets

```html
<!-- ❌ WRONG - Too small -->
<button style="width: 24px; height: 24px;">Delete</button>

<!-- ✅ RIGHT - Touch-friendly -->
<button style="width: 48px; height: 48px; padding: 12px;">Delete</button>
```

### Progressive Enhancement

**App works at every level of capability.**

```
Level 1: HTML only (no JavaScript)
  - Content displays
  - Forms work
  - Links navigate
  ↓
Level 2: CSS loaded
  - Layout renders
  - Responsive design works
  - Animations smooth
  ↓
Level 3: JavaScript loaded
  - Interactivity enhanced
  - Real-time updates
  - Advanced features
  ↓
Level 4: Native APIs
  - Camera access
  - Location tracking
  - Biometric auth
```

### Mobile-First Example

```html
<!-- Start simple for mobile -->
<div class="card">
  <h3>{{ item.title }}</h3>
  <p>{{ item.description }}</p>
  <button>Details</button>
</div>

<!-- CSS: Mobile first -->
<style>
  .card {
    display: block;        /* Stack vertically */
    width: 100%;           /* Full width */
    padding: 1rem;
    margin-bottom: 1rem;
  }
  
  button {
    width: 100%;           /* Full width button */
    padding: 12px;
    font-size: 16px;
    touch-action: manipulation;  /* No double-tap zoom */
  }
  
  /* Tablet and up: Multi-column */
  @media (min-width: 768px) {
    .cards-container {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 1rem;
    }
    
    .card { width: auto; }
    button { width: auto; }
  }
  
  /* Desktop and up: 3 columns */
  @media (min-width: 1024px) {
    .cards-container {
      grid-template-columns: repeat(3, 1fr);
    }
  }
</style>
```

---

## Platform Support Strategy

### Option 1: Cross-Platform (Single Codebase)

**Frameworks:** React Native, Flutter, NativeScript, Ionic

**Approach:**
```
Shared Code (70-80%)
├── Business logic
├── API calls
├── State management
└── Shared components
    ↓
Platform-Specific Code (20-30%)
├── iOS UI components
├── Android UI components
└── Native API wrappers
```

**Pros:**
- ✅ Code reuse (faster development)
- ✅ Single team can maintain
- ✅ Consistent behavior across platforms
- ✅ Faster time to market

**Cons:**
- ❌ Limited native performance
- ❌ Restricted native API access
- ❌ Larger app size
- ❌ Potential platform-specific bugs

**When to use:**
- Startups (speed matters)
- Consistent UX required
- Limited budget
- Feature-rich apps

---

### Option 2: Native + Shared Backend

**Approach:**
```
iOS App (Swift)          Android App (Kotlin)
    ↓                          ↓
    └─────── Shared API ───────┘
             (REST/GraphQL)
                  ↓
            Backend Services
```

**Pros:**
- ✅ Native performance
- ✅ Full native API access
- ✅ Best possible UX
- ✅ Smaller app size

**Cons:**
- ❌ Code duplication (same features twice)
- ❌ Larger team required
- ❌ Slower development
- ❌ More expensive

**When to use:**
- Performance critical (games, heavy graphics)
- Native features required (AR, advanced sensors)
- Mature product (can afford investment)
- Large budget

---

### Option 3: Hybrid (Web View + Native Shell)

**Approach:**
```
Web App (HTML/CSS/JS)
    ↓
WebView Container
    ↓
Native Shell (platform APIs)
```

**Pros:**
- ✅ Fastest development (web dev experience)
- ✅ Code reuse with web

**Cons:**
- ❌ Worst performance
- ❌ Limited native access
- ❌ Poor user experience
- ❌ App store restrictions

**When to use:**
- Very tight timeline
- Simple app (not performance-critical)
- Proof of concept

---

### Recommended Strategy

**For most projects:**

```
Phase 1: Web + Cross-Platform (React Native/Flutter)
         - Fastest to market
         - Share logic between web and mobile
         - Good performance for most apps

Phase 2: If performance insufficient
         - Native iOS (Swift)
         - Native Android (Kotlin)
         - Share backend API only
```

---

## Device Constraints

### Screen Size Management

**Common screen sizes:**

```
Mobile:
├── Small:  320px width  (iPhone SE, old phones)
├── Medium: 375px width  (iPhone 11, Galaxy S10)
└── Large:  414px width  (iPhone 12 Pro, Galaxy S20)

Tablet:
├── iPad:   768px width  (portrait), 1024px (landscape)
└── Others: 600px-900px

Foldable:
└── When folded:  variable (500px-600px)
```

**Responsive Layout:**

```html
<!-- ✅ Use flexible layouts -->
<style>
  .container {
    max-width: 100%;
    padding: max(1rem, 5vw);  /* Responsive padding */
  }
  
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1rem;
  }
</style>
```

### Battery Optimization

**Reduce power consumption:**

```
❌ WRONG - Drains battery fast
- Continuous network requests (every second)
- High brightness animations
- GPS tracking always on
- Bluetooth scanning constant
- CPU-intensive operations in background

✅ RIGHT - Battery conscious
- Batch network requests (debounce/throttle)
- Smooth 60fps animations only when visible
- GPS only when user needs it
- Bluetooth connects on demand
- Background tasks minimized
```

**Example:**

```typescript
// ❌ WRONG - Polls every 1 second
setInterval(() => {
  this.api.getUpdates().subscribe(...);
}, 1000);  // Drains battery!

// ✅ RIGHT - Smart syncing
class SyncManager {
  private syncInterval = 30000;  // 30 seconds
  
  sync() {
    if (!this.isOnline) return;        // Don't sync offline
    if (!this.hasChanges) return;      // Only sync if needed
    if (this.onBattery && this.lowBattery) {
      this.syncInterval = 5 * 60000;   // Increase interval
    }
    
    this.api.sync().subscribe(...);
  }
}
```

### Storage Management

**Devices have limited storage:**

```
Budget phones:
├── Internal: 16GB - 64GB
└── After OS: 8GB - 32GB available

High-end phones:
├── Internal: 128GB - 1TB
└── After OS: 100GB - 900GB available

Tablets:
└── Similar to high-end phones
```

**Storage best practices:**

```
❌ WRONG - Wastes storage
- Store full-resolution images
- Cache everything indefinitely
- No cleanup mechanism
- Multiple copies of same data

✅ RIGHT - Storage conscious
- Downscale images before storing
- Implement cache expiration (7-30 days)
- Delete old data automatically
- Deduplicate data
- Allow user to clear cache
```

**Example:**

```typescript
class CacheManager {
  async cacheImage(url: string) {
    // ❌ WRONG - Full resolution
    // const blob = await fetch(url);
    
    // ✅ RIGHT - Downscaled
    const blob = await fetch(`${url}?w=400&q=80`);
    const cache = await caches.open('images-v1');
    await cache.put(url, new Response(blob));
  }
  
  async cleanExpiredCache() {
    const cache = await caches.open('images-v1');
    const keys = await cache.keys();
    const now = Date.now();
    
    keys.forEach(request => {
      // Delete if older than 30 days
      if (now - request.timestamp > 30 * 24 * 60 * 60 * 1000) {
        cache.delete(request);
      }
    });
  }
}
```

### Network Optimization

**Mobile networks are slow and expensive:**

```
Network speeds:
├── 3G: 1-5 Mbps (high latency)
├── 4G/LTE: 5-50 Mbps (medium latency)
├── 5G: 50-300+ Mbps (low latency)
└── WiFi: 5-300+ Mbps (variable latency)

Challenges:
- High latency (200-500ms vs WiFi 10-50ms)
- Packet loss (requests fail)
- Connection switches (WiFi ↔ cellular)
- Metered data (users pay per MB)
```

**Network best practices:**

```
❌ WRONG - Wastes bandwidth
- Large image files (2-5MB per image)
- Uncompressed JSON responses
- No caching
- Retransmit on timeout
- Download all data at once

✅ RIGHT - Bandwidth conscious
- Compressed images (<100KB per image)
- Gzipped JSON responses
- Aggressive caching
- Smart retry with exponential backoff
- Lazy load data (pagination, virtual scroll)
```

**Example:**

```typescript
class ImageLoader {
  async loadImage(url: string, maxWidth: number = 400): Promise<string> {
    // ✅ Request optimized image
    const optimizedUrl = `${url}?w=${maxWidth}&q=75&format=webp`;
    
    // ✅ Use cache first
    const cached = await this.getFromCache(optimizedUrl);
    if (cached) return cached;
    
    // ✅ Lazy load with intersection observer
    return new Promise((resolve) => {
      const observer = new IntersectionObserver(async (entries) => {
        if (entries[0].isIntersecting) {
          const blob = await fetch(optimizedUrl).then(r => r.blob());
          await this.saveToCache(optimizedUrl, blob);
          resolve(URL.createObjectURL(blob));
          observer.unobserve(entries[0].target);
        }
      });
      observer.observe(imageElement);
    });
  }
}
```

### Permissions

**Always request permissions at runtime:**

```typescript
// ❌ WRONG - Request all at install
{
  "permissions": [
    "CAMERA",
    "LOCATION",
    "CONTACTS",
    "MICROPHONE"
  ]
}

// ✅ RIGHT - Request when needed
class PermissionManager {
  async requestCamera(): Promise<boolean> {
    const result = await navigator.permissions.query({ name: 'camera' });
    
    if (result.state === 'prompt') {
      // Show explanation first
      this.showDialog('We need camera access to take photos');
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      return !!stream;
    }
    
    return result.state === 'granted';
  }
  
  async requestLocation(): Promise<GeolocationCoordinates | null> {
    if (!navigator.geolocation) return null;
    
    return new Promise((resolve) => {
      navigator.geolocation.getCurrentPosition(
        position => resolve(position.coords),
        error => {
          console.error('Location error:', error);
          resolve(null);  // Graceful fallback
        }
      );
    });
  }
}
```

---

## Offline-First Architecture

### Principle

**App works without internet. Sync when reconnected.**

### Architecture

```
User Interaction
    ↓
Check if online?
    ↓
    ├─ YES → Send to backend
    │         ├─ Success → Update local
    │         └─ Fail → Queue locally + retry
    │
    └─ NO → Queue locally + show "offline" indicator
            ↓
    (When online again)
    └─ Sync queue → Send to backend
                  ├─ Success → Clear queue
                  └─ Fail → Keep in queue + retry later
```

### Implementation

```typescript
class OfflineQueueService {
  private queue: QueuedAction[] = [];
  private isOnline = navigator.onLine;
  
  constructor(
    private db: LocalDatabase,
    private api: ApiService
  ) {
    // Monitor connectivity
    window.addEventListener('online', () => this.onOnline());
    window.addEventListener('offline', () => this.onOffline());
  }
  
  async executeAction(action: Action): Promise<void> {
    if (this.isOnline) {
      // Execute immediately
      try {
        await this.api.execute(action);
      } catch (error) {
        // Queue on failure
        await this.queueAction(action);
        throw error;
      }
    } else {
      // Queue for later
      await this.queueAction(action);
    }
  }
  
  private async queueAction(action: Action): Promise<void> {
    const queued: QueuedAction = {
      id: generateId(),
      action,
      timestamp: Date.now(),
      retries: 0
    };
    
    await this.db.queue.add(queued);
    this.queue.push(queued);
  }
  
  private async onOnline(): Promise<void> {
    this.isOnline = true;
    console.log('Online - syncing queue');
    await this.syncQueue();
  }
  
  private async syncQueue(): Promise<void> {
    const queued = await this.db.queue.getAll();
    
    for (const item of queued) {
      try {
        await this.api.execute(item.action);
        await this.db.queue.remove(item.id);
      } catch (error) {
        // Exponential backoff
        item.retries++;
        if (item.retries > 5) {
          await this.db.queue.remove(item.id);
          this.notifyError(`Failed to sync: ${item.action.type}`);
        }
      }
    }
  }
  
  private onOffline(): void {
    this.isOnline = false;
    console.log('Offline - queuing actions');
    this.showOfflineIndicator();
  }
}
```

### Conflict Resolution

**When offline changes conflict with backend changes:**

```typescript
class ConflictResolver {
  async resolveConflict(
    local: Data,
    remote: Data
  ): Promise<Data> {
    // Strategy 1: Last-write-wins
    if (local.updatedAt > remote.updatedAt) {
      return local;
    }
    return remote;
    
    // Strategy 2: User chooses
    const choice = await this.showConflictDialog(local, remote);
    return choice;
    
    // Strategy 3: Merge fields
    return {
      ...remote,
      userEditedField: local.userEditedField
    };
  }
}
```

---

## Real-Time Synchronization

### WebSocket vs Polling

**Option 1: WebSocket (Recommended)**

```typescript
class RealtimeSync {
  private socket: WebSocket;
  
  connect() {
    this.socket = new WebSocket('wss://api.example.com/sync');
    
    this.socket.onmessage = (event) => {
      const update = JSON.parse(event.data);
      this.handleUpdate(update);
    };
    
    this.socket.onerror = (error) => {
      console.error('WebSocket error:', error);
      this.fallbackToPolling();
    };
  }
  
  sendUpdate(data: any) {
    if (this.socket.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify(data));
    }
  }
}
```

**Option 2: Polling (Fallback)**

```typescript
class PollingSync {
  private pollInterval = 30000;  // 30 seconds
  
  startPolling() {
    setInterval(async () => {
      if (!navigator.onLine) return;  // Don't poll offline
      
      try {
        const updates = await this.api.getUpdates();
        this.handleUpdates(updates);
      } catch (error) {
        // Backoff on error
        this.pollInterval = Math.min(this.pollInterval * 1.5, 5 * 60000);
      }
    }, this.pollInterval);
  }
}
```

**Pros/Cons:**

| Approach | Pros | Cons |
|----------|------|------|
| WebSocket | Low latency, low bandwidth, real-time | More complex, not all servers support |
| Polling | Simple, works everywhere | Higher latency, higher bandwidth usage |

---

## Mobile API Design

### Payload Optimization

**Minimize data transfer:**

```
❌ WRONG - 5MB response
{
  "users": [
    { "id": 1, "name": "...", "avatar_large": "...",
      "biography": "...", "followers": 1000000,
      "following": 500, "posts": [...50 full objects...]
    }
  ]
}

✅ RIGHT - 50KB response (100x smaller)
{
  "users": [
    { "id": 1, "name": "...", "avatar_url": "..." }
  ]
}
```

**Use sparse fields:**

```
GET /api/users?fields=id,name,avatar_url
```

### Pagination for Mobile

**Mobile apps must paginate:**

```
❌ WRONG - Download all 10,000 users
GET /api/users → 10MB response

✅ RIGHT - Cursor-based pagination
GET /api/users?limit=20&cursor=abc123 → 50KB response

Response:
{
  "users": [...20 items...],
  "cursor_next": "def456",
  "has_more": true
}
```

### Compression

**Always compress responses:**

```typescript
// Server-side
app.use(compression());  // Gzip compression

// Client-side verification
const response = await fetch('/api/users');
const size = response.headers.get('content-length');
console.log('Compressed size:', size);
```

---

## Native Integration

### Platform-Specific APIs

**iOS (Swift):**

```swift
import CoreLocation

class LocationManager {
  let manager = CLLocationManager()
  
  func requestLocation() {
    manager.requestWhenInUseAuthorization()
    manager.startUpdatingLocation()
  }
}
```

**Android (Kotlin):**

```kotlin
import android.location.Location
import android.Manifest

class LocationManager {
  fun requestLocation() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
      activity.requestPermissions(
        arrayOf(Manifest.permission.ACCESS_FINE_LOCATION),
        PERMISSION_CODE
      )
    }
  }
}
```

### Bridging Native & Web

**React Native:**

```typescript
import { NativeModules } from 'react-native';

const BiometricModule = NativeModules.Biometric;

async function authenticate() {
  try {
    const result = await BiometricModule.authenticate();
    return result;
  } catch (error) {
    console.error('Biometric auth failed:', error);
  }
}
```

**Flutter:**

```dart
import 'package:flutter/services.dart';

const platform = MethodChannel('com.example.app/biometric');

Future<void> authenticate() async {
  try {
    final result = await platform.invokeMethod('authenticate');
  } catch (e) {
    print('Biometric auth failed: $e');
  }
}
```

---

## Performance Optimization

### App Size Management

```
Target app sizes:
├── Startup: < 50MB
├── After first use: < 100MB
└── Maximum: < 200MB
```

**Reduce bundle size:**

```
❌ WRONG - Large dependencies
import _ from 'lodash';  // 70KB
import moment from 'moment';  // 67KB
import crypto from 'crypto-js';  // 100KB

✅ RIGHT - Minimal dependencies
import { debounce } from './utils';  // 500B
use native Date API
use native crypto APIs
```

### Memory Management

```typescript
// ❌ WRONG - Memory leak
listItems.forEach(item => {
  element.addEventListener('click', () => {
    handleClick(item);
  });
});

// ✅ RIGHT - Event delegation
element.addEventListener('click', (e) => {
  const item = findItem(e.target);
  handleClick(item);
});

// Clean up on unmount
ngOnDestroy() {
  element.removeEventListener('click', handler);
}
```

### Render Performance

```typescript
// ❌ WRONG - Re-renders 1000 items every change
<div *ngFor="let item of items">
  {{ item.name }} - {{ complexCalculation(item) }}
</div>

// ✅ RIGHT - Virtual scrolling
<cdk-virtual-scroll-viewport itemSize="50" class="list">
  <div *cdkVirtualFor="let item of items">
    {{ item.name }}
  </div>
</cdk-virtual-scroll-viewport>

// Or React with react-window:
<FixedSizeList
  height={600}
  itemCount={items.length}
  itemSize={50}
  width="100%"
>
  {renderRow}
</FixedSizeList>
```

---

## Testing Strategy

### Testing on Real Devices

**Always test on:**
- Smallest phone (iPhone SE, Galaxy A11)
- Common phone (iPhone 12, Galaxy S10)
- Large phone (iPhone 12 Pro Max, Galaxy S20 Ultra)
- Tablet (iPad, Android tablet)
- Foldable (if targeting)

### Network Throttling

```typescript
// Simulate slow network
async function testOnSlowNetwork() {
  // Chrome DevTools: Network tab → Throttling
  // Or simulate programmatically
  
  const slowFetch = async (url: string) => {
    const delay = 2000;  // 2 second latency
    await new Promise(r => setTimeout(r, delay));
    return fetch(url);
  };
}
```

### Battery & Memory Profiling

```
iOS: Xcode Instruments
Android: Android Profiler
Web: Chrome DevTools Performance tab
```

---

## Deployment & Distribution

### App Store Guidelines

**iOS (Apple App Store):**
- Minimum 50MB free space on device
- iOS 11+ support (typical)
- HTTPS for all connections
- Privacy policy required

**Android (Google Play):**
- Minimum 4.4 API level (typical)
- Signed APK/AAB
- Privacy policy required
- Optional: 64-bit support

### Beta Testing

```
Phase 1: Internal testing
         ├─ Team members
         └─ Trusted users

Phase 2: Beta program
         ├─ TestFlight (iOS)
         ├─ Google Play beta
         └─ 100-1000 users

Phase 3: Production release
         └─ Full rollout
```

### Update Strategy

```
Critical bug → Push update immediately
Feature update → Batch with other changes
Minor fix → Can wait for next cycle

Versioning: MAJOR.MINOR.PATCH
├─ MAJOR: Breaking changes
├─ MINOR: New features
└─ PATCH: Bug fixes
```

---

## Mobile Checklist

**Before shipping mobile app:**

- [ ] Works on smallest screen (320px)
- [ ] Works on largest screen (foldable)
- [ ] Touch targets ≥ 44px
- [ ] Offline queue implemented
- [ ] Sync works when reconnected
- [ ] Network throttling tested
- [ ] Battery drain acceptable
- [ ] Storage use < 100MB
- [ ] App size < 100MB
- [ ] Animations smooth (60fps)
- [ ] No memory leaks
- [ ] Permissions requested properly
- [ ] Loading states for all API calls
- [ ] Error states handled
- [ ] No sensitive data in localStorage
- [ ] App works on slow network
- [ ] Tested on real devices
- [ ] App store submission requirements met
- [ ] Privacy policy present
- [ ] Analytics configured (if needed)

---

## Quick Reference

**Always:**
- ✅ Design mobile-first
- ✅ Handle offline
- ✅ Sync on reconnect
- ✅ Optimize for battery
- ✅ Minimize data transfer
- ✅ Test on real devices
- ✅ Show loading states
- ✅ Handle permission denials
- ✅ Use touch-friendly UI
- ✅ Implement error handling

**Never:**
- ❌ Design desktop-first
- ❌ Assume always online
- ❌ Transfer large payloads
- ❌ Drain battery unnecessarily
- ❌ Request all permissions at startup
- ❌ Use tiny touch targets
- ❌ Forget error states
- ❌ Store sensitive data locally
- ❌ Rely on hover interactions
- ❌ Test only on emulators

---

## Related Documents

- FRONTEND_ARCHITECTURE.md — General frontend patterns
- API_STANDARDS.md — Backend API design for mobile
- ARCHITECTURE_PRINCIPLES.md — Overall system principles

---

**Last Updated:** [DATE]  
**Next Review:** [DATE]  
**Maintainer:** [TEAM/PERSON]
