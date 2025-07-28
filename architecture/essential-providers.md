# Essential Providers: Network, AppState, Keyboard

## Overview

This recipe describes three essential providers for React Native apps:
- **NetworkProvider**: Unified network status and connectivity hook
- **AppStateProvider**: Unified app state (foreground/background) hook
- **KeyboardProvider**: Unified keyboard state and events, powered by [react-native-keyboard-controller](https://github.com/kirillzyusko/react-native-keyboard-controller)

These providers help you solve many common and future problems, such as:
- Triggering actions after user inactivity (idle detection)
- Reacting to network changes in a single place
- Building responsive UIs that adapt to keyboard state and height

---

## 1. NetworkProvider & useNetwork

Centralizes network connectivity state and provides a hook for all components.

**Provider Example (with full type safety):**
```tsx
import React, { useEffect, useState, createContext, ReactNode, useContext } from 'react'
import NetInfo, { NetInfoState } from '@react-native-community/netinfo'

export const NetworkContext = createContext<NetInfoState | null>(null)

export function NetworkProvider({ children }: { children: ReactNode }) {
  const [state, setState] = useState<NetInfoState | null>(null)

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(setState)
    return () => unsubscribe()
  }, [])

  return (
    <NetworkContext.Provider value={state}>{children}</NetworkContext.Provider>
  )
}

export function useNetwork() {
  return useContext(NetworkContext)
}
```

**Usage:**
```tsx
const { isConnected } = useNetwork()
if (!isConnected) return <OfflineBanner />
```

---

## 2. AppStateProvider & useAppState

Centralizes app foreground/background state and provides a hook for all components. Useful for triggering actions after idle or when returning to the app.


**Provider Example (with full type safety):**
```tsx
import React, { useEffect, useState, useRef, createContext, ReactNode } from 'react'
import { AppState, AppStateStatus } from 'react-native'

export type AppStateContextValue = {
  foreground: boolean
  status: AppStateStatus
}

export const AppStateContext = createContext<AppStateContextValue>({
  foreground: AppState.currentState === 'active',
  status: AppState.currentState,
})

export function AppStateProvider({ children }: { children: ReactNode }) {
  const appState = useRef(AppState.currentState)
  const [state, setState] = useState<AppStateContextValue>({
    foreground: appState.current === 'active',
    status: appState.current,
  })

  useEffect(() => {
    const subscription = AppState.addEventListener('change', handleAppStateChange)
    return () => subscription.remove()
  }, [])

  const handleAppStateChange = (nextAppState: AppStateStatus) => {
    const foreground = !!(
      appState.current.match(/inactive|background/) && nextAppState === 'active'
    )
    appState.current = nextAppState
    setState({
      foreground,
      status: appState.current,
    })
  }

  return (
    <AppStateContext.Provider value={state}>{children}</AppStateContext.Provider>
  )
}

export function useAppState() {
  return React.useContext(AppStateContext)
}
```

**Usage:**
```tsx
const { appState } = useAppState()
useEffect(() => {
  if (appState === 'active') {
    // Trigger refresh or resume logic
  }
}, [appState])
```

---

## 3. KeyboardProvider & useKeyboard (with Keyboard Controller)

Provides keyboard state, height, and events for responsive UI. Uses `react-native-keyboard-controller` for best-in-class keyboard handling.


**Provider Example (with full type safety and dismiss):**
```tsx
import React, { createContext, useEffect, useState, ReactNode, useCallback } from 'react'
import { Keyboard, KeyboardEvent } from 'react-native'
import { KeyboardProvider as KeyboardControllerProvider } from 'react-native-keyboard-controller'

interface KeyboardContextValue {
  isKeyboardVisible: boolean
  keyboardHeight: number
  dismissKeyboard: () => void
}

export const KeyboardContext = createContext<KeyboardContextValue>({
  isKeyboardVisible: false,
  keyboardHeight: 0,
  dismissKeyboard: () => {},
})

export function KeyboardProvider({ children }: { children: ReactNode }) {
  const [isKeyboardVisible, setKeyboardVisible] = useState(false)
  const [keyboardHeight, setKeyboardHeight] = useState(0)

  useEffect(() => {
    const showListener = Keyboard.addListener(
      Platform.OS === 'ios' ? 'keyboardWillShow' : 'keyboardDidShow',
      (e: KeyboardEvent) => {
        setKeyboardVisible(true)
        setKeyboardHeight(e.endCoordinates.height)
      },
    )
    const hideListener = Keyboard.addListener(
      Platform.OS === 'ios' ? 'keyboardWillHide' : 'keyboardDidHide',
      () => {
        setKeyboardVisible(false)
        setKeyboardHeight(0)
      },
    )
    return () => {
      showListener.remove()
      hideListener.remove()
    }
  }, [])

  const dismissKeyboard = useCallback(() => {
    Keyboard.dismiss()
    setKeyboardVisible(false)
  }, [])

  return (
    <KeyboardControllerProvider>
      <KeyboardContext.Provider value={{ isKeyboardVisible, keyboardHeight, dismissKeyboard }}>
        {children}
      </KeyboardContext.Provider>
    </KeyboardControllerProvider>
  )
}

export function useKeyboard() {
  return React.useContext(KeyboardContext)
}
```

**Usage in createStyles:**
```ts
const useStyles = createStyles(({ isKeyboardVisible, keyboardHeight }) => ({
  inputWrapper: {
    marginBottom: isKeyboardVisible ? keyboardHeight : 0,
  },
}))
```

---

## Why Use These Providers?

- **Single source of truth** for network, app state, and keyboard events
- **Trigger actions** (refresh, sync, analytics) on app resume or network reconnect
- **Responsive UI**: adapt layouts to keyboard, avoid overlap, and improve UX
- **Future-proof**: easily add more global listeners (e.g., deep links, notifications)

---

## Best Practices
- Use these providers at the root of your app (e.g., in your main Providers stack)
- Use hooks in any component to react to state changes
- Combine with Redux or Context for global state management

---

These essential providers will help you avoid many common pitfalls and make your app more robust, responsive, and maintainable.
