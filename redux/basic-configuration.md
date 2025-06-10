# Setting up Redux for React Native with persist and listeners


## Annotation

This example is part of the **React Native Code Snippets** codebase. You can find the repository [here](https://github.com/WhidRubeld/react-native-code-snippets). The repository contains many useful implementations of various functionalities that can help you address issues in your React Native project or expand its capabilities.

## Project architecture

This tutorial assumes that you are working with an [Expo project](https://expo.dev) in the [Managed Workflow](https://docs.expo.dev/guides/managed-workflow/). Additionally, the project uses [TypeScript](https://www.typescriptlang.org/), providing static type checking and enhanced development experience. 

The project is organized with a nested `src` folder where all Redux-related logic is placed in the `store` directory. The general path for these files will be `./src/store/*`.

All hooks for working with Redux will be imported from the common hooks folder located at `./src/hooks/*`.

All future providers (include Redux) will be located in the `./src/providers/*` directory.

## Step 1 - Configure TypeScript relative paths

Set up TypeScript relative path prefixes for your project according to the [instructions](https://gist.github.com/WhidRubeld/31319a5cd4de05bde79ad6e50743f154).

## Step 2 - Installing packages

To install the following packages using `yarn`, run the command:

```bash
yarn add react-redux @reduxjs/toolkit redux-persist
```

To install libraries that require native modules using `yarn`, run the following command:

```bash
npx expo install @react-native-community/netinfo @react-native-async-storage/async-storage
```

## Step 3 - Adding a listener function

To implement all the required listeners, add a `listen-handler.ts` file to the `./src/store/*` directory.

```ts
// ./src/store/listen-handler.ts

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


## Step 4 - Adding first reducer

Add the first reducer in the `settings.ts` file to the `./src/store/slices/*` directory.

```ts
// ./src/store/slices/settings.ts

import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'

export interface SettingState {
  foo: string
}

const initialState: SettingState = {
  foo: 'bar',
}

export const settingSlice = createSlice({
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
  extraReducers(builder) {},
})

export const { setFoo, resetSettings } = settingSlice.actions

export default settingSlice
```

## Step 5 - Adding the store configuration file

Add the `configure.ts` file to the `./src/store/*` directory for configuring your Redux Toolkit.

```ts
import AsyncStorage from '@react-native-async-storage/async-storage'
import { combineReducers, configureStore } from '@reduxjs/toolkit'
import { setupListeners } from '@reduxjs/toolkit/query'
import {
  PersistConfig,
  PersistedState,
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
import settingSlice from './slices/settings'
// import api from '../api'

// import { isDevelopment } from '@/constants'

const CACHE_VERSION = 1

const rootReducer = combineReducers({
  // [api.reducerPath]: api.reducer,  // for RTK query
  [settingSlice.reducerPath]: settingSlice.reducer,
})

const persistConfig: PersistConfig<any> = {
  key: 'root',
  storage: AsyncStorage,
  version: CACHE_VERSION,
  // whitelist: [api.reducerPath],  // for RTK query
  // can use createMigrate from redux-persist to work out better backward compatibility
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
  // devTools: isDevelopment,
  middleware: (gDM) =>
    gDM({
      serializableCheck: {
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    }),
  // }).concat(api.middleware), // for RTK query
  reducer: persistedReducer,
})

setupListeners(store.dispatch, listenHandler)

export const persistor = persistStore(store)

export default store

```

## Step 6 - Adding type exports

Add the exports for the necessary types that will be useful for your work, especially for the hooks.


```ts
// ./src/store/types.ts

import store from './configure'

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

## Step 7 - Implement the Redux Provider

Add the `index.ts` file to the `./src/providers/*` directory.

```ts
// ./src/providers/index.tsx

import { ReactNode } from 'react'
import { gestureHandlerRootHOC } from 'react-native-gesture-handler'
import { SafeAreaProvider } from 'react-native-safe-area-context'
import { Provider as ReduxProvider } from 'react-redux
import { PersistGate } from 'redux-persist/integration/react'

// import AppStateProvider from './AppState'
// import NetworkProvider from './Network'
// import ThemeProvider from './Theme'

import { store, persistor } from '@/store'

// export { default as NotificationProvider } from './Notification'

function RootProviders({ children }: { children: ReactNode }) {
  return (
    <SafeAreaProvider>
      {/* <AppStateProvider>
        <NetworkProvider> */}
      <ReduxProvider store={store}>
         <PersistGate loading={null} persistor={persistor}>
            {/* <ThemeProvider> */}
            {children}
            {/* </ThemeProvider> */}
         </PersistGate>
      </ReduxProvider>
      {/* </NetworkProvider>
      </AppStateProvider> */}
    </SafeAreaProvider>
  )
}

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

## Step 9 - Adding Hooks

Add typed hooks for working with the store to avoid dealing with type definitions each time. Create the `index.ts` file in the `./src/hooks/*` directory.

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

## Step 10 - Persist and rehydration for RTK Query

If you plan to use RTK Query, you need to add extractRehydrationInfo to your api. Create the `index.ts` file in the `./src/api/*` directory.

```
// ./src/api/index.ts

import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import { REHYDRATE } from 'redux-persist'

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: 'https://your-api.com' }),
  reducerPath: 'api',
  tagTypes: [],
  // persist and rehydration
  extractRehydrationInfo(action, { reducerPath }) {
    if (action.type === REHYDRATE && !!action.payload) {
      const rehydratedState = (action.payload as any)[reducerPath]

      return {
        ...rehydratedState,
        mutations: {},
      }
    }
  },
  endpoints: () => ({}),
})

export default api
```

## Summary

Now you have configured Redux Toolkit for managing the store in your project. Here's an example of how to use the hooks:
```tsx
import { Text, View, Button } from 'react-native'

import { useDispatch, useSelector } from '@/hooks'
import { setFoo } from '@/store/slices/settings'

const YourComponent = () => {
  const dispatch = useDispatch()
  const { foo } = useSelector((state) => state.settings)

  return (
    <View>
      <Text>{foo}</Text>
      <Button
        title="Update state"
        onPress={() => dispatch(setFoo('updated state value'))}
      />
    </View>
  )
}
```

Integration of RTK Query will be detailed in another Gist.
