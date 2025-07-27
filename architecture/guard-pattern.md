# Guard Pattern for React Native

## Overview

Implement the Guard Pattern - a powerful architectural approach for managing application states and controlling access to different parts of your React Native app. Guards act as gatekeepers that render different UI based on specific conditions while keeping your main components clean and focused.

## What You'll Achieve

✅ Clean separation of concerns with conditional rendering logic  
✅ Reusable guard components for common app states  
✅ Centralized access control and state management  
✅ Improved code maintainability and testability  
✅ Consistent UX patterns across your application  

## Prerequisites

- Redux Toolkit configured ([see recipe](../redux/basic-configuration.md))
- TypeScript path mapping ([see recipe](../base/typescript-relative-path.md))
- Basic understanding of React patterns

## Architecture

The Guard Pattern organizes conditional rendering logic into reusable components:

```
src/
├── guards/
│   ├── index.ts              # Export all guards
│   ├── Auth/
│   │   └── index.tsx         # Authentication guard
│   ├── Loading/
│   │   └── index.tsx         # Loading state guard
│   ├── Error/
│   │   └── index.tsx         # Error handling guard
│   ├── Empty/
│   │   └── index.tsx         # Empty state guard
│   └── Task/
│       ├── index.tsx         # App initialization guard
│       └── tasks.ts          # Initialization tasks
└── interfaces/
    └── guards.ts             # Guard interfaces
```

## Core Concept

Guards are Higher-Order Components that:
1. **Check a condition** (auth status, loading state, etc.)
2. **Render fallback UI** if condition is not met
3. **Render children** if condition is satisfied

```tsx
// Basic Guard Pattern
const Guard = ({ condition, fallback, children }) => {
  return condition ? children : fallback
}
```

## Implementation

### Step 1 - Create guard interfaces

Define common interfaces for guards. Add to `src/interfaces/guards.ts`:

```typescript
// src/interfaces/guards.ts

import { ReactElement, ReactNode } from 'react'

export interface BaseGuardProps {
  children: ReactNode
  fallback?: ReactElement
}

export interface ConditionalGuardProps extends BaseGuardProps {
  condition: boolean
}

export interface LoadingGuardProps extends BaseGuardProps {
  isLoading: boolean
  loadingComponent?: ReactElement
}

export interface AuthGuardProps extends BaseGuardProps {
  requireAuth?: boolean
  redirectTo?: string
}

export interface ErrorGuardProps extends BaseGuardProps {
  error?: Error | string | null
  onRetry?: () => void
  errorComponent?: ReactElement
}
```

### Step 2 - Create Loading Guard

Implement a loading state guard. Add to `src/guards/Loading/index.tsx`:

```tsx
// src/guards/Loading/index.tsx

import React, { ReactElement } from 'react'
import { View, ActivityIndicator, StyleSheet } from 'react-native'

import { LoadingGuardProps } from '@/interfaces/guards'
import { useTheme } from '@/hooks/useTheme'

const DefaultLoadingComponent = () => {
  const { theme } = useTheme()
  
  const styles = StyleSheet.create({
    container: {
      flex: 1,
      justifyContent: 'center',
      alignItems: 'center',
      backgroundColor: theme.colors.background,
    },
  })

  return (
    <View style={styles.container}>
      <ActivityIndicator 
        size="large" 
        color={theme.colors.primary} 
      />
    </View>
  )
}

const LoadingGuard: React.FC<LoadingGuardProps> = ({
  children,
  isLoading,
  loadingComponent = <DefaultLoadingComponent />,
}) => {
  if (isLoading) {
    return <>{loadingComponent}</>
  }

  return <>{children}</>
}

export default LoadingGuard
```

### Step 3 - Create Auth Guard

Implement authentication guard. Add to `src/guards/Auth/index.tsx`:

```tsx
// src/guards/Auth/index.tsx

import React, { ReactElement } from 'react'
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native'

import { AuthGuardProps } from '@/interfaces/guards'
import { useSelector } from '@/hooks'
import { useTheme } from '@/hooks/useTheme'

const DefaultAuthFallback = () => {
  const { theme } = useTheme()
  
  const styles = StyleSheet.create({
    container: {
      flex: 1,
      justifyContent: 'center',
      alignItems: 'center',
      backgroundColor: theme.colors.background,
      padding: theme.spacing.lg,
    },
    title: {
      fontSize: theme.typography.fontSize.xl,
      fontWeight: theme.typography.fontWeight.bold,
      color: theme.colors.text,
      marginBottom: theme.spacing.md,
      textAlign: 'center',
    },
    description: {
      fontSize: theme.typography.fontSize.md,
      color: theme.colors.textSecondary,
      textAlign: 'center',
      marginBottom: theme.spacing.xl,
    },
    button: {
      backgroundColor: theme.colors.primary,
      paddingHorizontal: theme.spacing.xl,
      paddingVertical: theme.spacing.md,
      borderRadius: theme.borderRadius.md,
    },
    buttonText: {
      color: '#FFFFFF',
      fontSize: theme.typography.fontSize.md,
      fontWeight: theme.typography.fontWeight.semibold,
    },
  })

  const handleSignIn = () => {
    // Navigate to auth screen
    console.log('Navigate to sign in')
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome!</Text>
      <Text style={styles.description}>
        Sign in to access your account and enjoy all features.
      </Text>
      <TouchableOpacity style={styles.button} onPress={handleSignIn}>
        <Text style={styles.buttonText}>Sign In</Text>
      </TouchableOpacity>
    </View>
  )
}

const AuthGuard: React.FC<AuthGuardProps> = ({
  children,
  requireAuth = true,
  fallback = <DefaultAuthFallback />,
}) => {
  const { oauth } = useSelector((state) => state.auth)
  
  // If auth is required and user is not authenticated
  if (requireAuth && !oauth) {
    return <>{fallback}</>
  }
  
  // If auth is not required or user is authenticated
  return <>{children}</>
}

export default AuthGuard
```

### Step 4 - Create Error Guard

Implement error handling guard. Add to `src/guards/Error/index.tsx`:

```tsx
// src/guards/Error/index.tsx

import React, { ReactElement } from 'react'
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native'

import { ErrorGuardProps } from '@/interfaces/guards'
import { useTheme } from '@/hooks/useTheme'

const DefaultErrorComponent: React.FC<{ onRetry?: () => void }> = ({ 
  onRetry 
}) => {
  const { theme } = useTheme()
  
  const styles = StyleSheet.create({
    container: {
      flex: 1,
      justifyContent: 'center',
      alignItems: 'center',
      backgroundColor: theme.colors.background,
      padding: theme.spacing.lg,
    },
    title: {
      fontSize: theme.typography.fontSize.xl,
      fontWeight: theme.typography.fontWeight.bold,
      color: theme.colors.error,
      marginBottom: theme.spacing.md,
      textAlign: 'center',
    },
    description: {
      fontSize: theme.typography.fontSize.md,
      color: theme.colors.textSecondary,
      textAlign: 'center',
      marginBottom: theme.spacing.xl,
    },
    button: {
      backgroundColor: theme.colors.primary,
      paddingHorizontal: theme.spacing.xl,
      paddingVertical: theme.spacing.md,
      borderRadius: theme.borderRadius.md,
    },
    buttonText: {
      color: '#FFFFFF',
      fontSize: theme.typography.fontSize.md,
      fontWeight: theme.typography.fontWeight.semibold,
    },
  })

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Oops!</Text>
      <Text style={styles.description}>
        Something went wrong. Please try again.
      </Text>
      {onRetry && (
        <TouchableOpacity style={styles.button} onPress={onRetry}>
          <Text style={styles.buttonText}>Try Again</Text>
        </TouchableOpacity>
      )}
    </View>
  )
}

const ErrorGuard: React.FC<ErrorGuardProps> = ({
  children,
  error,
  onRetry,
  errorComponent,
}) => {
  if (error) {
    return <>{errorComponent || <DefaultErrorComponent onRetry={onRetry} />}</>
  }

  return <>{children}</>
}

export default ErrorGuard
```

### Step 5 - Create Task Guard (App Initialization)

Implement app initialization guard. Add to `src/guards/Task/index.tsx`:

```tsx
// src/guards/Task/index.tsx

import React, { ReactNode, useCallback, useEffect } from 'react'

import SplashScreen from './SplashScreen'
import { initializationTasks } from './tasks'

import { useDispatch, useSelector } from '@/hooks'
import { setReady } from '@/store/slices/settings'

interface TaskGuardProps {
  children: ReactNode
}

const TaskGuard: React.FC<TaskGuardProps> = ({ children }) => {
  const dispatch = useDispatch()
  const { ready } = useSelector((state) => state.settings)

  const runTasks = useCallback(async () => {
    if (ready) return

    try {
      // Run tasks sequentially
      for (const task of initializationTasks) {
        await task()
      }
      
      dispatch(setReady(true))
    } catch (error) {
      console.error('Initialization failed:', error)
      // Handle initialization failure
      // Could show error screen or retry logic
    }
  }, [ready, dispatch])

  useEffect(() => {
    runTasks()
  }, [runTasks])

  return (
    <>
      {ready && children}
      <SplashScreen isVisible={!ready} />
    </>
  )
}

export default TaskGuard
```

### Step 6 - Create initialization tasks

Define app initialization tasks. Add to `src/guards/Task/tasks.ts`:

```typescript
// src/guards/Task/tasks.ts

import { store } from '@/store'
import { 
  initializeTheme, 
  configureLocale,
  getNotificationPermission 
} from '@/store/slices/settings'
import { initDeviceIdAsync } from '@/store/slices/auth'

type InitTask = () => Promise<void>

export const initializationTasks: InitTask[] = [
  // Initialize theme system
  async () => {
    await store.dispatch(initializeTheme()).unwrap()
  },

  // Configure locale/language
  async () => {
    await store.dispatch(configureLocale()).unwrap()
  },

  // Initialize device ID for analytics
  async () => {
    await store.dispatch(initDeviceIdAsync()).unwrap()
  },

  // Get notification permissions
  async () => {
    await store.dispatch(getNotificationPermission()).unwrap()
  },

  // Initialize other services
  async () => {
    // Add custom initialization logic here
    // Examples: analytics, crash reporting, feature flags, etc.
    await new Promise(resolve => setTimeout(resolve, 500)) // Simulate async work
  },
]
```

### Step 7 - Create splash screen component

Create a splash screen for the task guard. Add to `src/guards/Task/SplashScreen.tsx`:

```tsx
// src/guards/Task/SplashScreen.tsx

import React from 'react'
import { View, Text, ActivityIndicator, StyleSheet } from 'react-native'

import { useTheme } from '@/hooks/useTheme'

interface SplashScreenProps {
  isVisible: boolean
}

const SplashScreen: React.FC<SplashScreenProps> = ({ isVisible }) => {
  const { theme } = useTheme()

  const styles = StyleSheet.create({
    container: {
      ...StyleSheet.absoluteFillObject,
      justifyContent: 'center',
      alignItems: 'center',
      backgroundColor: theme.colors.background,
    },
    logo: {
      fontSize: theme.typography.fontSize.xxl,
      fontWeight: theme.typography.fontWeight.bold,
      color: theme.colors.primary,
      marginBottom: theme.spacing.xl,
    },
    loader: {
      marginTop: theme.spacing.lg,
    },
  })

  if (!isVisible) return null

  return (
    <View style={styles.container}>
      <Text style={styles.logo}>Your App</Text>
      <ActivityIndicator 
        size="large" 
        color={theme.colors.primary}
        style={styles.loader}
      />
    </View>
  )
}

export default SplashScreen
```

### Step 8 - Create guard exports

Export all guards. Add to `src/guards/index.ts`:

```typescript
// src/guards/index.ts

export { default as AuthGuard } from './Auth'
export { default as LoadingGuard } from './Loading'
export { default as ErrorGuard } from './Error'
export { default as TaskGuard } from './Task'

// Additional guards
export { default as EmptyGuard } from './Empty'
export { default as SyncGuard } from './Sync'
```

## Usage Examples

### Basic guard composition

```tsx
// src/App.tsx

import React from 'react'

import MainNavigator from '@/navigation/MainNavigator'
import { TaskGuard, AuthGuard } from '@/guards'

const App = () => {
  return (
    <TaskGuard>
      <AuthGuard>
        <MainNavigator />
      </AuthGuard>
    </TaskGuard>
  )
}

export default App
```

### Data fetching with guards

```tsx
// src/screens/ProfileScreen.tsx

import React from 'react'
import { View, Text } from 'react-native'

import { LoadingGuard, ErrorGuard } from '@/guards'
import { useGetProfileQuery } from '@/api/profile'

const ProfileScreen = () => {
  const { data: profile, isLoading, error, refetch } = useGetProfileQuery()

  return (
    <ErrorGuard error={error} onRetry={refetch}>
      <LoadingGuard isLoading={isLoading}>
        <View>
          <Text>Welcome, {profile?.name}!</Text>
          {/* Profile content */}
        </View>
      </LoadingGuard>
    </ErrorGuard>
  )
}

export default ProfileScreen
```

### Nested guard composition

```tsx
// src/screens/DashboardScreen.tsx

import React from 'react'

import { AuthGuard, LoadingGuard, ErrorGuard } from '@/guards'
import DashboardContent from './DashboardContent'
import { useDashboardData } from '@/hooks/useDashboardData'

const DashboardScreen = () => {
  const { data, isLoading, error, refetch } = useDashboardData()

  return (
    <AuthGuard>
      <ErrorGuard error={error} onRetry={refetch}>
        <LoadingGuard isLoading={isLoading}>
          <DashboardContent data={data} />
        </LoadingGuard>
      </ErrorGuard>
    </AuthGuard>
  )
}

export default DashboardScreen
```

## Advanced Patterns

### Conditional Guard

```tsx
// src/guards/ConditionalGuard.tsx

import React from 'react'
import { ConditionalGuardProps } from '@/interfaces/guards'

const ConditionalGuard: React.FC<ConditionalGuardProps> = ({
  condition,
  children,
  fallback = null,
}) => {
  return condition ? <>{children}</> : <>{fallback}</>
}

export default ConditionalGuard
```

### Permission Guard

```tsx
// src/guards/PermissionGuard.tsx

import React from 'react'
import { useSelector } from '@/hooks'

interface PermissionGuardProps {
  children: React.ReactNode
  permission: string
  fallback?: React.ReactElement
}

const PermissionGuard: React.FC<PermissionGuardProps> = ({
  children,
  permission,
  fallback = null,
}) => {
  const userPermissions = useSelector((state) => state.auth.permissions)
  
  const hasPermission = userPermissions?.includes(permission)
  
  return hasPermission ? <>{children}</> : <>{fallback}</>
}

export default PermissionGuard
```

## Best Practices

### ✅ Do

- Keep guards simple and focused on single responsibility
- Use composition over complex nested conditions
- Provide meaningful fallback components
- Test guards independently
- Document guard behavior and use cases

### ❌ Don't

- Mix business logic with guard logic
- Create overly complex guard hierarchies
- Forget to handle edge cases and errors
- Hardcode UI in guards (use configurable fallbacks)

## Benefits

1. **Separation of Concerns**: Conditional logic is separated from business logic
2. **Reusability**: Guards can be used across different screens/components
3. **Consistency**: Uniform UX patterns for common states
4. **Testability**: Easy to test conditional rendering logic
5. **Maintainability**: Centralized access control and state management

## Related Recipes

- [Redux Toolkit Setup](../redux/basic-configuration.md) - Required for state management
- [Theme System](../theming/theme-system-redux.md) - For consistent guard styling
- [Custom Splash Screen](./custom-splash-screen.md) - Advanced splash implementation

---

Your Guard Pattern implementation is complete! You now have a robust system for managing conditional rendering and access control throughout your React Native application.
