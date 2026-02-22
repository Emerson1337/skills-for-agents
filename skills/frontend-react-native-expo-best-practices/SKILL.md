---
name: frontend-react-native-expo-best-practices
description: React Native with Expo architecture and performance best practices. Use when building, reviewing, or refactoring Expo apps, including navigation, state, device APIs, builds, OTA updates, and release workflows.
---

# React Native Expo Best Practices

Production-oriented patterns for building React Native apps with Expo. Contains 11 rules across 6 categories focused on maintainable architecture, predictable runtime behavior, performance, and safe release workflows.

## When to Apply

Reference these guidelines when:

- Starting a new Expo app
- Refactoring an existing React Native codebase
- Integrating device features (camera, location, notifications)
- Optimizing list and image performance
- Setting up EAS Build and EAS Update pipelines
- Reviewing Expo app code for production readiness

## Rules Summary

### Architecture (CRITICAL)

#### expo-managed-first - @rules/expo-managed-first.md

Prefer Expo managed workflow by default and only eject/prebuild for unavoidable native requirements.

```bash
# Managed workflow (preferred)
npx create-expo-app@latest mobile

# Prebuild only when you truly need native project edits
npx expo prebuild
```

#### app-config-ts-single-source - @rules/app-config-ts-single-source.md

Use `app.config.ts` as the single source of truth for environment-aware app config.

```ts
import "dotenv/config";

export default {
  expo: {
    name: process.env.APP_NAME ?? "Acme",
    ios: { bundleIdentifier: process.env.IOS_BUNDLE_ID },
    android: { package: process.env.ANDROID_PACKAGE },
  },
};
```

### Navigation (HIGH)

#### expo-router-file-based-routing - @rules/expo-router-file-based-routing.md

Use Expo Router file-based routing to keep navigation structure explicit and colocated with screens.

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from "expo-router";

export default function TabLayout() {
  return <Tabs screenOptions={{ headerShown: false }} />;
}
```

#### deep-link-contract - @rules/deep-link-contract.md

Define and test deep link contracts early so push notifications and external links remain stable.

```ts
import * as Linking from "expo-linking";

const url = Linking.createURL("/orders/123");
```

### Performance (CRITICAL)

#### flashlist-large-lists - @rules/flashlist-large-lists.md

Use `FlashList` for medium/large datasets and memoize row renderers.

```tsx
<FlashList
  data={orders}
  estimatedItemSize={72}
  keyExtractor={(item) => item.id}
  renderItem={renderOrderRow}
/>
```

#### expo-image-caching - @rules/expo-image-caching.md

Use `expo-image` with explicit sizing and caching policies instead of unbounded `<Image />` usage.

```tsx
<Image source={avatarUrl} style={{ width: 48, height: 48 }} contentFit="cover" cachePolicy="memory-disk" />
```

### Device APIs & Security (HIGH)

#### permissions-just-in-time - @rules/permissions-just-in-time.md

Request permissions at the moment of use, not at app startup.

```ts
const { status } = await Location.requestForegroundPermissionsAsync();
if (status !== "granted") return;
```

#### secure-store-secrets - @rules/secure-store-secrets.md

Store tokens and secrets in `expo-secure-store`, never in AsyncStorage.

```ts
await SecureStore.setItemAsync("session_token", token);
```

### Data & App Lifecycle (HIGH)

#### appstate-query-invalidation - @rules/appstate-query-invalidation.md

Reconnect and refresh stale server state when the app returns to foreground.

```ts
AppState.addEventListener("change", (state) => {
  if (state === "active") queryClient.invalidateQueries();
});
```

### Build, Update, and Release (CRITICAL)

#### eas-build-profiles - @rules/eas-build-profiles.md

Separate `development`, `preview`, and `production` build profiles in `eas.json`.

```json
{
  "build": {
    "development": { "developmentClient": true },
    "preview": {},
    "production": { "autoIncrement": true }
  }
}
```

#### eas-update-runtime-version - @rules/eas-update-runtime-version.md

Pin runtime versions and release OTA updates per channel/branch to avoid incompatible updates.

```json
{
  "expo": {
    "runtimeVersion": { "policy": "appVersion" },
    "updates": { "url": "https://u.expo.dev/<project-id>" }
  }
}
```

## Key Files

- `app/` - Expo Router routes and layouts
- `app.config.ts` - Dynamic Expo app configuration
- `eas.json` - Build profile definitions
- `package.json` - Scripts and dependency constraints
- `src/features/` - Feature modules and UI/state boundaries
