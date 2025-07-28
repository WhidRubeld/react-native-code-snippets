# FullWindowOverlay: Overlay UI Elements Above Native Navigation (React Navigation Native Stack)

## 🎯 Overview

This recipe demonstrates how to use `FullWindowOverlay` from `react-native-screens` to render UI elements (such as modals, snackbars, or notifications) above the native navigation bar when using React Navigation's Native Stack. This is especially useful for iOS, where native navigation can obscure overlays rendered in the normal React tree.

## 📋 Prerequisites
- React Native project using React Navigation Native Stack
- `react-native-screens` installed and properly configured

## 🔧 Implementation

Create a utility function to wrap your overlay elements:

```tsx
import { FullWindowOverlay } from 'react-native-screens';
import { Platform } from 'react-native';

const isIOS = Platform.OS === 'ios';

export const withOverlay = (element: React.ReactNode) =>
  isIOS ? <FullWindowOverlay>{element}</FullWindowOverlay> : element;
```

Use this utility to render overlays above the navigation bar:

```tsx
withOverlay(
  <>
    <ConfirmModal />
    <NetworkModal />
    <SnackbarWrapper />
  </>
)
```

## 💡 Usage
- Place the result of `withOverlay(...)` at the root of your app, typically above your navigation container or in your main layout component.
- On iOS, overlays will appear above the native navigation bar. On Android, overlays render as usual.

## ⚡ Best Practices
- Only use `FullWindowOverlay` for UI elements that must appear above navigation (e.g., global modals, toasts).
- Avoid using it for regular content, as it bypasses navigation stacking and can lead to unexpected UI layering.
- Test overlays on both iOS and Android to ensure consistent behavior.

## Example

```tsx
// App.tsx or RootLayout.tsx

import { withOverlay } from './withOverlay';

function App() {
  return (
    <NavigationContainer>
      {/* ...your navigation setup... */}
      {withOverlay(
        <>
          <ConfirmModal />
          <NetworkModal />
          <SnackbarWrapper />
        </>
      )}
    </NavigationContainer>
  );
}
```

---

*This pattern ensures your overlays are always visible, even above native navigation bars on iOS.*
