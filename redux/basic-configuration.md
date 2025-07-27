# Redux Toolkit Setup for React Native

## Overview

Complete Redux Toolkit configuration for React Native applications with persistence, real-time listeners, and TypeScript support. This recipe provides a production-ready Redux setup that handles state persistence, network connectivity, and app lifecycle events.

## What You'll Achieve

✅ Full Redux Toolkit configuration with TypeScript  
✅ State persistence using redux-persist  
✅ Real-time app state and network listeners  
✅ Type-safe hooks and selectors  
✅ RTK Query integration ready  

## Prerequisites

- Expo project with TypeScript
- TypeScript path mapping configured ([see recipe](../base/typescript-relative-path.md))
- Basic understanding of Redux concepts

## Architecture

This recipe organizes Redux logic in a clean, scalable structure:

```
src/
├── store/
│   ├── configure.ts          # Store configuration
│   ├── listenHandler.ts       # App state listeners  
│   ├── types.ts              # TypeScript types
│   ├── index.ts              # Exports
│   └── slices/
│       └── settings.ts       # Example slice
├── hooks/
│   └── index.ts              # Typed Redux hooks
├── providers/
│   └── index.tsx             # Provider stack
└── api/
    └── index.ts              # RTK Query setup
```

## Step 1 - Configure TypeScript relative paths

Set up TypeScript relative path prefixes for your project according to the [instructions](../base/typescript-relative-path.md).

## Step 2 - Installing packages

Install the core Redux packages:

```bash
yarn add react-redux @reduxjs/toolkit redux-persist
```

Install native dependencies for network and storage:

```bash
npx expo install @react-native-community/netinfo @react-native-async-storage/async-storage
```
Integration of RTK Query will be detailed in another recipe.

## Step 3 - Adding a listener function

Create app state and network listeners. Add `listenHandler.ts` to `./src/store/`:

```ts
// ./src/store/listenHandler.ts

import NetInfo, {
  NetInfoState,
  NetInfoSubscription,
} from '@react-native-community/netinfo'
import { ThunkDispatch } from '@reduxjs/toolkit'
import { AppState, AppStateStatus, NativeEventSubscription } from 'react-native'

let initialized = false

let appStateSubscription: NativeEventSubscription | null = null
let appState = AppState.currentState

let netInfoUnsubscribe: NetInfoSubscription | null = null

export default function listenHandler(
  dispatch: ThunkDispatch<any, any, any>,
  {
    onFocus,
    onFocusLost,
    onOffline,
    onOnline,
  }: {
    onFocus: () => void
    onFocusLost: () => void
    onOffline: () => void
    onOnline: () => void
  },
) {
  const handleFocus = () => dispatch(onFocus())
  const handleFocusLost = () => dispatch(onFocusLost())
  const handleOnline = () => dispatch(onOnline())
  const handleOffline = () => dispatch(onOffline())

  const _handleAppStateChange = (nextAppState: AppStateStatus) => {
    const foreground = !!(
      appState.match(/inactive|background/) && nextAppState === 'active'
    )

    if (foreground) handleFocus()
    else handleFocusLost()

    appState = nextAppState
  }

  const _handleNetInfoChange = (state: NetInfoState) => {
    const { isConnected } = state

    if (isConnected) handleOnline()
    else handleOffline()
  }

  if (!initialized) {
    appStateSubscription = AppState.addEventListener(
      'change',
      _handleAppStateChange,
    )
    netInfoUnsubscribe = NetInfo.addEventListener(_handleNetInfoChange)
    initialized = true
  }

  const unsubscribe = () => {
    if (appStateSubscription) {
      appStateSubscription.remove()
      appStateSubscription = null
    }
    if (netInfoUnsubscribe) {
      netInfoUnsubscribe()
      netInfoUnsubscribe = null
    }
    initialized = false
  }

  return unsubscribe
}
```


## Step 4 - Create your first slice

Create a simple example slice to test the configuration. Add `settings.ts` to `./src/store/slices/`:

```ts
// ./src/store/slices/settings.ts

import { createSlice } from '@reduxjs/toolkit'

export interface SettingsState {
  foo: string
}

const initialState: SettingsState = {
  foo: 'bar',
}

export const settingsSlice = createSlice({
  name: 'settings',
  initialState,
  reducers: {
    setFoo: (state, { payload }: { payload: string }) => {
      state.foo = payload
    },
    resetSettings: (state) => {
      state.foo = 'bar'
    },
  },
})

export const { setFoo, resetSettings } = settingsSlice.actions

export default settingsSlice
```

## Step 5 - Configure the store

Create the main store configuration. Add `configure.ts` to `./src/store/`:

```ts
// ./src/store/configure.ts

import AsyncStorage from '@react-native-async-storage/async-storage'
import { combineReducers, configureStore } from '@reduxjs/toolkit'
import { setupListeners } from '@reduxjs/toolkit/query'
import {
  PersistConfig,
  persistReducer,
  persistStore,
  FLUSH,
  REHYDRATE,
  PAUSE,
  PERSIST,
  PURGE,
  REGISTER,
} from 'redux-persist'

import { listenHandler } from './listenHandler'
import settingsSlice from './slices/settings'
// import api from '../api' // RTK Query API slice

const CACHE_VERSION = 1

const rootReducer = combineReducers({
  // [api.reducerPath]: api.reducer, // Uncomment when using RTK Query
  [settingsSlice.reducerPath]: settingsSlice.reducer,
})

const persistConfig: PersistConfig<any> = {
  key: 'root',
  storage: AsyncStorage,
  version: CACHE_VERSION,
  whitelist: ['settings'], // Add slices you want to persist
  migrate: (state, version) => {
    if (version === CACHE_VERSION) return Promise.resolve(state)
    return Promise.resolve({
      ...rootReducer(undefined, { type: '@@INIT' }),
      _persist: { version: CACHE_VERSION, rehydrated: true },
    })
  },
}

const persistedReducer = persistReducer<ReturnType<typeof rootReducer>>(
  persistConfig,
  rootReducer,
)

const store = configureStore({
  devTools: __DEV__,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    }), // .concat(api.middleware), // Uncomment when using RTK Query
  reducer: persistedReducer,
})

// Setup listeners for app state and network changes
setupListeners(store.dispatch, listenHandler)

export const persistor = persistStore(store)
export default store
```

## Step 6 - Export store types

Create type definitions for TypeScript support. Add `types.ts` to `./src/store/`:

```ts
// ./src/store/types.ts

import store from './configure'

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

## Step 7 - Create store exports

Add main exports. Create `index.ts` in `./src/store/`:

```ts
// ./src/store/index.ts

export { default as store, persistor } from './configure'
export * from './types'
```

## Step 8 - Create typed hooks

Create type-safe Redux hooks. Add to `./src/hooks/index.ts`:

```ts
// ./src/hooks/index.ts

import {
  useDispatch as useDefaultDispatch,
  useSelector as useDefaultSelector,
  TypedUseSelectorHook,
} from 'react-redux'

import { AppDispatch, RootState } from '@/store'

export const useDispatch = () => useDefaultDispatch<AppDispatch>()
export const useSelector: TypedUseSelectorHook<RootState> = useDefaultSelector
```

## Step 9 - Setup providers

Create the provider stack. Add `index.tsx` to `./src/providers/`:

```tsx
// ./src/providers/index.tsx

import { ReactNode } from 'react'
import { gestureHandlerRootHOC } from 'react-native-gesture-handler'
import { SafeAreaProvider } from 'react-native-safe-area-context'
import { Provider as ReduxProvider } from 'react-redux'
import { PersistGate } from 'redux-persist/integration/react'

import { store, persistor } from '@/store'

function RootProviders({ children }: { children: ReactNode }) {
  return (
    <SafeAreaProvider>
      <ReduxProvider store={store}>
        <PersistGate loading={null} persistor={persistor}>
          {children}
        </PersistGate>
      </ReduxProvider>
    </SafeAreaProvider>
  )
}

export default gestureHandlerRootHOC(RootProviders)
```

## Step 10 - Connect to your app

Update your main `App.tsx` to use the providers:

```tsx
// App.tsx

import Launcher from '@/app'
import RootProviders from '@/providers'

export default function App() {
  return (
    <RootProviders>
      <Launcher />
    </RootProviders>
  )
}
```

export default gestureHandlerRootHOC(RootProviders)
```

## Step 8 - Connect Provider

Wrap your application in the providers stack in `./App.tsx`.

```ts
// ./App.tsx

// import * as Notifications from 'expo-notifications'
// import * as SplashScreen from 'expo-splash-screen'
// import { Platform, UIManager } from 'react-native'

import Launcher from '@/app'
import RootProviders from '@/providers'

// SplashScreen.preventAutoHideAsync()

// if (
//   Platform.OS === 'android' &&
//   UIManager.setLayoutAnimationEnabledExperimental
// ) {
//   UIManager.setLayoutAnimationEnabledExperimental(true)
// }

// Notifications.setNotificationHandler({
//   handleNotification: async () => {
//     return {
//       shouldShowAlert: false,
//       shouldPlaySound: false,
//       shouldSetBadge: true,
//     }
//   },
// })

export default function Root() {
  return (
    <RootProviders>
      <Launcher />
    </RootProviders>
  )
}

```

```

## Step 11 - RTK Query integration (Optional)

If you want to use RTK Query for API calls, create the API slice. Add `index.ts` to `./src/api/`:

```ts
// ./src/api/index.ts

import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { REHYDRATE } from 'redux-persist'

const api = createApi({
  baseQuery: fetchBaseQuery({ 
    baseUrl: 'https://your-api.com',
    // Add auth headers, etc.
  }),
  reducerPath: 'api',
  tagTypes: ['User', 'Post'], // Define your cache tags
  extractRehydrationInfo(action, { reducerPath }) {
    if (action.type === REHYDRATE && action.payload) {
      const rehydratedState = (action.payload as any)[reducerPath]
      return {
        ...rehydratedState,
        mutations: {}, // Clear mutations on rehydration
      }
    }
  },
  endpoints: () => ({}),
})

export default api
```

Then uncomment the API-related lines in your store configuration.

## Usage Examples

Now you can use Redux in your components:

```tsx
// ./src/components/ExampleComponent.tsx

import React from 'react'
import { View, Text, Button } from 'react-native'

import { useDispatch, useSelector } from '@/hooks'
import { setFoo } from '@/store/slices/settings'

const ExampleComponent = () => {
  const dispatch = useDispatch()
  const { foo } = useSelector((state) => state.settings)

  return (
    <View style={{ padding: 20 }}>
      <Text>Current value: {foo}</Text>
      <Button 
        title="Update Value" 
        onPress={() => dispatch(setFoo('updated value'))} 
      />
    </View>
  )
}

export default ExampleComponent
```

## Best Practices

### ✅ Do
- Use typed hooks for type safety
- Organize slices by feature/domain
- Use RTK Query for server state
- Persist only necessary data
- Handle rehydration properly

### ❌ Don't
- Store derived state in Redux
- Persist sensitive data without encryption
- Ignore migration strategies
- Over-engineer simple state

## What's Next?

- **RTK Query Advanced Patterns**: Caching strategies, optimistic updates
- **Real-time Updates**: WebSocket integration with Redux
- **Performance**: Selector optimization and memoization
- **Testing**: Testing Redux logic and components

---

Your Redux setup is now complete! This configuration provides a solid foundation for scalable state management in your React Native application.