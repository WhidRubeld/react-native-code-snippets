
# Utility-First Styling with createStyles

## Overview

A unified approach to creating and using styles in React Native with the `createStyles` utility. This recipe allows you to centralize theme, safe area insets, screen dimensions, and keyboard state, providing type safety and convenience for your components.

## What You'll Get

✅ A single way to describe styles with access to theme, insets, screen sizes, and more  
✅ Typed hooks for getting styles and helper data  
✅ Easy integration with any component  
✅ Support for dynamic styles (e.g., dark mode, keyboard state)

## Prerequisites

- Expo/React Native project with TypeScript
- Installed libraries:
  - `react-native-safe-area-context`
  - `@react-navigation/native` (optional)
  - Your own implementation of useTheme/useKeyboard (or similar)
- Redux Toolkit configured ([see recipe](../redux/basic-configuration.md))
- Theme System configured ([see recipe](./theme-system-redux.md))
- Basic understanding of React Native StyleSheet system

## Architecture

Recommended structure:

```
src/
├── utils/
│   └── createStyles.ts      # Universal styling utility
├── hooks/
│   ├── useTheme.ts         # Theme hook
│   └── useKeyboard.ts      # Keyboard hook
└── components/
    └── ...
```

## Step 1 — Implement createStyles

Create the file `src/utils/createStyles.ts` with the following content:

```ts
import {
  StyleSheet,
  ViewStyle,
  TextStyle,
  ImageStyle,
  useWindowDimensions
} from 'react-native'
import { EdgeInsets, useSafeAreaInsets } from 'react-native-safe-area-context'

import useTheme from '@/hooks/useTheme'
import useKeyboard from '@/hooks/useKeyboard'

type NamedStyles<T> = { [P in keyof T]: ViewStyle | TextStyle | ImageStyle }
type StyleBuilder<T extends NamedStyles<T> | NamedStyles<any>> = (props: {
  isDark: boolean
  theme: any // Replace with your ThemeType
  insets: EdgeInsets
  absoluteFillObject: StyleSheet.AbsoluteFillStyle
  hairlineWidth: number
  windowWidth: number
  windowHeight: number
  isKeyboardVisible: boolean
  keyboardHeight: number
}) => T

export default function createStyles<
  T extends NamedStyles<T> | NamedStyles<any>
>(builder: StyleBuilder<T>) {
  return function useStyles() {
    const insets = useSafeAreaInsets()
    const { isKeyboardVisible, keyboardHeight } = useKeyboard()
    const { theme, isDark } = useTheme()
    const { width: windowWidth, height: windowHeight } = useWindowDimensions()
    const { absoluteFillObject, hairlineWidth } = StyleSheet

    return {
      styles: StyleSheet.create(
        typeof builder === 'function'
          ? builder({
              theme,
              isDark,
              insets,
              absoluteFillObject,
              hairlineWidth,
              windowWidth,
              windowHeight,
              isKeyboardVisible,
              keyboardHeight
            })
          : builder
      ),
      theme,
      isDark,
      insets,
      absoluteFillObject,
      hairlineWidth,
      windowWidth,
      windowHeight,
      isKeyboardVisible,
      keyboardHeight
    }
  }
}
```

## Step 2 — Using in Components

Create a hook for your component's styles:

```ts
// ExampleComponent.tsx
import React from 'react'
import { View, Text } from 'react-native'
import createStyles from '@/utils/createStyles'

const useStyles = createStyles(({ theme, isDark, insets }) => ({
  container: {
    flex: 1,
    backgroundColor: isDark ? theme.colors.backgroundDark : theme.colors.background,
    paddingTop: insets.top + 16,
    paddingHorizontal: 16,
  },
  title: {
    color: theme.colors.primary,
    fontSize: 20,
    fontWeight: 'bold',
  },
}))

export default function ExampleComponent() {
  const { styles, isDark, insets } = useStyles()
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Current theme: {isDark ? 'Dark' : 'Light'}</Text>
      <Text>SafeArea top: {insets.top}</Text>
    </View>
  )
}
```

## Step 3 — Dynamic Styles

You can use any parameters from the builder function for dynamic styling:

```ts
const useStyles = createStyles(({ isKeyboardVisible, keyboardHeight }) => ({
  inputWrapper: {
    marginBottom: isKeyboardVisible ? keyboardHeight : 0,
  },
}))
```

## Best Practices

- Use createStyles for all components to ensure unified styling and theme access
- Do not store computed values outside the builder — use the provided parameters
- For complex themes, create a separate ThemeType and use it instead of any
- Always type your returned styles

## What's Next?

- Extend the builder function for your needs (e.g., add platform parameters)
- Integrate with your theming system and global variables
- Use with memoization for performance optimization

---

This approach centralizes styling, theme, and safe area handling, making your component code cleaner and easier to maintain.
