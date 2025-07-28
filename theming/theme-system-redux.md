# Theme System with Redux Integration

## Overview

Implement a complete theme system with Redux state management, automatic device theme detection, and seamless switching between light, dark, and auto modes. This recipe provides a production-ready theming solution with TypeScript support.

## What You'll Achieve

✅ Complete theme system with light/dark/auto modes  
✅ Redux integration for theme state management  
✅ Automatic device theme detection and sync  
✅ Theme persistence across app sessions  
✅ TypeScript support for theme values  
✅ Hot theme switching without restart  

## Prerequisites

- Redux Toolkit configured ([see recipe](../redux/basic-configuration.md))
- TypeScript path mapping ([see recipe](../base/typescript-relative-path.md))
- Basic understanding of React Context

## Architecture

This recipe creates a theme system with the following structure:

```
src/
├── interfaces/
│   └── theme.ts              # Theme interfaces
├── providers/
│   └── Theme.tsx             # Theme provider with Redux sync
├── store/slices/
│   └── settings.ts           # Redux slice with theme actions
├── hooks/
│   └── useTheme.ts           # Theme hook
└── constants/
    └── theme.ts              # Theme definitions
```

## Implementation

### Step 1 - Define theme interfaces

Create theme type definitions. Add to `src/interfaces/theme.ts`:

```typescript
// src/interfaces/theme.ts

export enum Theme {
  light = 'light',
  dark = 'dark',
  auto = 'auto',
}

export interface ThemeColors {
  // Background colors
  background: string
  surface: string
  card: string
  
  // Text colors
  text: string
  textSecondary: string
  textTertiary: string
  
  // Brand colors
  primary: string
  secondary: string
  accent: string
  
  // Status colors
  success: string
  warning: string
  error: string
  info: string
  
  // UI colors
  border: string
  divider: string
  overlay: string
}

export interface ThemeSpacing {
  xs: number
  sm: number
  md: number
  lg: number
  xl: number
  xxl: number
}

export interface ThemeTypography {
  fontSize: {
    xs: number
    sm: number
    md: number
    lg: number
    xl: number
    xxl: number
  }
  fontWeight: {
    light: string
    normal: string
    medium: string
    semibold: string
    bold: string
  }
  lineHeight: {
    tight: number
    normal: number
    relaxed: number
  }
}

export interface ThemeDefinition {
  colors: ThemeColors
  spacing: ThemeSpacing
  typography: ThemeTypography
  borderRadius: {
    sm: number
    md: number
    lg: number
    xl: number
    full: number
  }
}

export interface ThemeContextValue {
  theme: ThemeDefinition
  activeTheme: Theme
  isDark: boolean
  setTheme: (theme: Theme) => void
}
```

### Step 2 - Create theme definitions

Define your theme constants. Add to `src/constants/theme.ts`:

```typescript
// src/constants/theme.ts

import { ThemeDefinition, ThemeColors } from '@/interfaces/theme'

const lightColors: ThemeColors = {
  background: '#FFFFFF',
  surface: '#F8F9FA',
  card: '#FFFFFF',
  
  text: '#1A1A1A',
  textSecondary: '#6B7280',
  textTertiary: '#9CA3AF',
  
  primary: '#007AFF',
  secondary: '#5856D6',
  accent: '#FF2D92',
  
  success: '#34C759',
  warning: '#FF9500',
  error: '#FF3B30',
  info: '#007AFF',
  
  border: '#E5E7EB',
  divider: '#F3F4F6',
  overlay: 'rgba(0, 0, 0, 0.5)',
}

const darkColors: ThemeColors = {
  background: '#000000',
  surface: '#1C1C1E',
  card: '#2C2C2E',
  
  text: '#FFFFFF',
  textSecondary: '#AEAEB2',
  textTertiary: '#8E8E93',
  
  primary: '#0A84FF',
  secondary: '#5E5CE6',
  accent: '#FF2D92',
  
  success: '#30D158',
  warning: '#FF9F0A',
  error: '#FF453A',
  info: '#64D2FF',
  
  border: '#38383A',
  divider: '#48484A',
  overlay: 'rgba(0, 0, 0, 0.7)',
}

const baseTheme = {
  spacing: {
    xs: 4,
    sm: 8,
    md: 16,
    lg: 24,
    xl: 32,
    xxl: 48,
  },
  typography: {
    fontSize: {
      xs: 12,
      sm: 14,
      md: 16,
      lg: 18,
      xl: 24,
      xxl: 32,
    },
    fontWeight: {
      light: '300',
      normal: '400',
      medium: '500',
      semibold: '600',
      bold: '700',
    },
    lineHeight: {
      tight: 1.2,
      normal: 1.5,
      relaxed: 1.8,
    },
  },
  borderRadius: {
    sm: 4,
    md: 8,
    lg: 12,
    xl: 16,
    full: 9999,
  },
}

export const lightTheme: ThemeDefinition = {
  ...baseTheme,
  colors: lightColors,
}

export const darkTheme: ThemeDefinition = {
  ...baseTheme,
  colors: darkColors,
}

export const themes = {
  light: lightTheme,
  dark: darkTheme,
} as const
```

### Step 3 - Update Redux settings slice

Add theme management to your Redux slice:

```typescript
// src/store/slices/settings.ts (additions)

import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'
import { Appearance } from 'react-native'
import AsyncStorage from '@react-native-async-storage/async-storage'

import { Theme } from '@/interfaces/theme'

export interface SettingsState {
  theme: Theme
  // ... other settings
}

const initialState: SettingsState = {
  theme: Theme.auto,
  // ... other initial state
}

// Async thunks for theme management
export const initializeTheme = createAsyncThunk(
  'settings/initializeTheme',
  async () => {
    try {
      const scheme = Appearance.getColorScheme()
      const storedTheme = await AsyncStorage.getItem('theme')
      if (!!storedTheme && storedTheme !== scheme) {
        Appearance.setColorScheme(theme !== Theme.auto ? theme : null)
      }
      return storedTheme ? (storedTheme as Theme) : Theme.auto
    } catch {
      return Theme.auto
    }
  }
)

export const changeTheme = createAsyncThunk(
  'settings/changeTheme',
  async (theme: Theme) => {
    try {
      await AsyncStorage.setItem('theme', theme)
      Appearance.setColorScheme(theme !== Theme.auto ? theme : null)
      return theme
    } catch (error) {
      throw error
    }
  }
)

export const settingsSlice = createSlice({
  name: 'settings',
  initialState,
  reducers: {
    // ... other reducers
  },
  extraReducers: (builder) => {
    builder
      .addCase(initializeTheme.fulfilled, (state, action) => {
        state.theme = action.payload
      })
      .addCase(changeTheme.fulfilled, (state, action) => {
        state.theme = action.payload
      })
  },
})

export default settingsSlice
```

### Step 4 - Create theme provider

Create a theme provider that syncs with Redux. Add to `src/providers/Theme.tsx`:

```tsx
// src/providers/Theme.tsx

import React, { 
  createContext, 
  useContext, 
  useEffect, 
  useMemo, 
  ReactNode 
} from 'react'
import { Appearance, ColorSchemeName } from 'react-native'

import { useDispatch, useSelector } from '@/hooks'
import { initializeTheme, changeTheme } from '@/store/slices/settings'
import { Theme, ThemeContextValue } from '@/interfaces/theme'
import { themes, lightTheme, darkTheme } from '@/constants/theme'

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined)

interface ThemeProviderProps {
  children: ReactNode
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const dispatch = useDispatch()
  const activeTheme = useSelector((state) => state.settings.theme)

  // Initialize theme on mount
  useEffect(() => {
    dispatch(initializeTheme())
  }, [dispatch])

  // Listen to device theme changes when in auto mode
  useEffect(() => {
    if (activeTheme !== Theme.auto) return

    const subscription = Appearance.addChangeListener(({ colorScheme }) => {
      // Theme will be recalculated automatically through useMemo
    })

    return () => subscription.remove()
  }, [activeTheme])

  // Calculate actual theme to use
  const resolvedTheme = useMemo(() => {
    if (activeTheme === Theme.auto) {
      const deviceTheme = Appearance.getColorScheme()
      return deviceTheme === 'dark' ? darkTheme : lightTheme
    }
    return themes[activeTheme] || lightTheme
  }, [activeTheme])

  const isDark = useMemo(() => {
    if (activeTheme === Theme.auto) {
      return Appearance.getColorScheme() === 'dark'
    }
    return activeTheme === Theme.dark
  }, [activeTheme])

  const setTheme = (theme: Theme) => {
    dispatch(changeTheme(theme))
  }

  const contextValue: ThemeContextValue = useMemo(() => ({
    theme: resolvedTheme,
    activeTheme,
    isDark,
    setTheme,
  }), [resolvedTheme, activeTheme, isDark])

  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  )
}

export const useTheme = (): ThemeContextValue => {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider')
  }
  return context
}
```

### Step 5 - Create theme hook

Create a convenient hook for accessing theme. Add to `src/hooks/useTheme.ts`:

```typescript
// src/hooks/useTheme.ts

export { useTheme } from '@/providers/Theme'

// Additional theme utilities
export const useThemeColors = () => {
  const { theme } = useTheme()
  return theme.colors
}

export const useThemeSpacing = () => {
  const { theme } = useTheme()
  return theme.spacing
}

export const useThemeTypography = () => {
  const { theme } = useTheme()
  return theme.typography
}
```

### Step 6 - Add provider to your app

Update your providers stack to include the theme provider:

```tsx
// src/providers/index.tsx (update)

import { ThemeProvider } from './Theme'
// ... other imports

function RootProviders({ children }: { children: ReactNode }) {
  return (
    <SafeAreaProvider>
      <ReduxProvider store={store}>
        <PersistGate loading={null} persistor={persistor}>
          <ThemeProvider>
            {children}
          </ThemeProvider>
        </PersistGate>
      </ReduxProvider>
    </SafeAreaProvider>
  )
}
```

## Usage Examples

### Basic theme usage in components

```tsx
// src/components/ThemedButton.tsx

import React from 'react'
import { TouchableOpacity, Text, StyleSheet } from 'react-native'

import { useTheme } from '@/hooks/useTheme'

interface ThemedButtonProps {
  title: string
  onPress: () => void
}

const ThemedButton: React.FC<ThemedButtonProps> = ({ title, onPress }) => {
  const { theme } = useTheme()

  const styles = StyleSheet.create({
    button: {
      backgroundColor: theme.colors.primary,
      paddingHorizontal: theme.spacing.lg,
      paddingVertical: theme.spacing.md,
      borderRadius: theme.borderRadius.md,
    },
    text: {
      color: '#FFFFFF',
      fontSize: theme.typography.fontSize.md,
      fontWeight: theme.typography.fontWeight.semibold,
    },
  })

  return (
    <TouchableOpacity style={styles.button} onPress={onPress}>
      <Text style={styles.text}>{title}</Text>
    </TouchableOpacity>
  )
}

export default ThemedButton
```

### Theme settings screen

```tsx
// src/screens/ThemeSettings.tsx

import React from 'react'
import { View, Text, StyleSheet } from 'react-native'

import { useTheme } from '@/hooks/useTheme'
import { Theme } from '@/interfaces/theme'

const ThemeSettings = () => {
  const { theme, activeTheme, setTheme } = useTheme()

  const styles = StyleSheet.create({
    container: {
      flex: 1,
      backgroundColor: theme.colors.background,
      padding: theme.spacing.lg,
    },
    title: {
      fontSize: theme.typography.fontSize.xl,
      fontWeight: theme.typography.fontWeight.bold,
      color: theme.colors.text,
      marginBottom: theme.spacing.lg,
    },
    option: {
      padding: theme.spacing.md,
      borderRadius: theme.borderRadius.md,
      backgroundColor: theme.colors.surface,
      marginBottom: theme.spacing.sm,
    },
    optionText: {
      fontSize: theme.typography.fontSize.md,
      color: theme.colors.text,
    },
  })

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Theme Settings</Text>
      
      {Object.values(Theme).map((themeOption) => (
        <TouchableOpacity
          key={themeOption}
          style={[
            styles.option,
            activeTheme === themeOption && {
              backgroundColor: theme.colors.primary,
            },
          ]}
          onPress={() => setTheme(themeOption)}
        >
          <Text
            style={[
              styles.optionText,
              activeTheme === themeOption && { color: '#FFFFFF' },
            ]}
          >
            {themeOption.charAt(0).toUpperCase() + themeOption.slice(1)}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
  )
}

export default ThemeSettings
```

## Best Practices

### ✅ Do

- Define consistent color palettes for both themes
- Use semantic color names (primary, surface) over descriptive ones (blue, gray)
- Implement theme switching without app restart
- Test both themes thoroughly
- Use theme values for all styling (avoid hardcoded colors)

### ❌ Don't

- Hardcode colors in components
- Forget to handle theme transitions in animations
- Mix theme approaches (stick to one system)
- Skip accessibility considerations for dark mode

## Related Recipes

- [Redux Toolkit Setup](../redux/basic-configuration.md) - Required for theme state management
- [Utility-First Styling](./utility-styling.md) - Enhanced styling with theme integration
- [Essential Providers](../providers/essential-providers.md) - Provider stack organization

---

Your theme system is now complete! You have a flexible, Redux-integrated theming solution that supports light, dark, and auto modes with full TypeScript support.
