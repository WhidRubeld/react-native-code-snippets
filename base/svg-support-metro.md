# SVG Support via Metro Configuration

## Overview

Configure Metro bundler to support SVG files in your React Native application. This recipe enables you to import and use SVG files as React components with full TypeScript support.

## What You'll Achieve

✅ Import SVG files as React components  
✅ Full TypeScript support for SVG imports  
✅ Optimized bundling with Metro configuration  
✅ Support for both local and remote SVG assets  

## Prerequisites

- Expo project with TypeScript
- Basic understanding of Metro bundler
- SVG files ready to use in your project

## Implementation

### Step 1 - Install required packages

Install the necessary packages for SVG support:

```bash
npx expo install react-native-svg
yarn add --dev react-native-svg-transformer
```

### Step 2 - Configure Metro bundler

Create or update `metro.config.js` in your project root:

```js
// metro.config.js

const { getDefaultConfig } = require('expo/metro-config')

const config = getDefaultConfig(__dirname)

// Add SVG support
config.transformer = {
  ...config.transformer,
  babelTransformerPath: require.resolve('react-native-svg-transformer'),
}

config.resolver = {
  ...config.resolver,
  assetExts: config.resolver.assetExts.filter((ext) => ext !== 'svg'),
  sourceExts: [...config.resolver.sourceExts, 'svg'],
}

module.exports = config
```

### Step 3 - Add TypeScript declarations

Create type declarations for SVG imports. Add to `types/svg.d.ts`:

```typescript
// types/svg.d.ts

declare module '*.svg' {
  import React from 'react'
  import { SvgProps } from 'react-native-svg'
  const content: React.FC<SvgProps>
  export default content
}
```

### Step 4 - Update TypeScript configuration

Ensure your `tsconfig.json` includes the types directory:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "~/*": ["./*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", "types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## Usage Examples

### Basic SVG import and usage

```tsx
// src/components/IconExample.tsx

import React from 'react'
import { View } from 'react-native'

// Import SVG as React component
import LogoIcon from '~/assets/icons/logo.svg'
import HomeIcon from '~/assets/icons/home.svg'

const IconExample = () => {
  return (
    <View>
      {/* Use as regular React component */}
      <LogoIcon width={100} height={50} />
      
      {/* Customize with props */}
      <HomeIcon 
        width={24} 
        height={24} 
        fill="#007AFF" 
      />
    </View>
  )
}

export default IconExample
```

### SVG with custom styling

```tsx
// src/components/StyledIcon.tsx

import React from 'react'
import { useColorScheme } from 'react-native'

import CheckIcon from '~/assets/icons/check.svg'

const StyledIcon = () => {
  const colorScheme = useColorScheme()
  
  return (
    <CheckIcon
      width={32}
      height={32}
      fill={colorScheme === 'dark' ? '#FFFFFF' : '#000000'}
    />
  )
}

export default StyledIcon
```

### Reusable Icon component

```tsx
// src/components/ui/Icon.tsx

import React from 'react'
import { SvgProps } from 'react-native-svg'

// Icon mapping
const icons = {
  home: require('~/assets/icons/home.svg').default,
  user: require('~/assets/icons/user.svg').default,
  settings: require('~/assets/icons/settings.svg').default,
} as const

type IconName = keyof typeof icons

interface IconProps extends SvgProps {
  name: IconName
  size?: number
}

const Icon: React.FC<IconProps> = ({ 
  name, 
  size = 24, 
  width, 
  height, 
  ...props 
}) => {
  const IconComponent = icons[name]
  
  return (
    <IconComponent
      width={width || size}
      height={height || size}
      {...props}
    />
  )
}

export default Icon

// Usage:
// <Icon name="home" size={24} fill="#007AFF" />
```

## Best Practices

### ✅ Do

- Optimize SVG files before using them (remove unnecessary elements)
- Use consistent naming for SVG files (kebab-case recommended)
- Set appropriate `viewBox` attributes in your SVG files
- Use fill and stroke props for dynamic theming
- Group related icons in subdirectories

### ❌ Don't

- Include very complex SVG files (impacts performance)
- Hardcode colors in SVG files if you need dynamic theming
- Use SVG for large illustrations (consider PNG/WebP instead)
- Forget to add accessibility props when needed

## File Organization

Organize your SVG assets in a structured way:

```
assets/
├── icons/
│   ├── navigation/
│   │   ├── home.svg
│   │   ├── profile.svg
│   │   └── settings.svg
│   ├── social/
│   │   ├── facebook.svg
│   │   ├── google.svg
│   │   └── apple.svg
│   └── common/
│       ├── check.svg
│       ├── close.svg
│       └── arrow.svg
└── illustrations/
    ├── empty-state.svg
    └── success.svg
```

## Troubleshooting

### Issue: SVG not rendering

**Solution**: Check that your SVG has proper `viewBox` attribute:
```xml
<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <!-- SVG content -->
</svg>
```

### Issue: TypeScript errors on SVG imports

**Solution**: Ensure `types/svg.d.ts` is included in your `tsconfig.json` and restart your TypeScript server.

### Issue: Metro bundler not recognizing SVG files

**Solution**: Clear Metro cache and restart:
```bash
npx expo start --clear
```

## Related Recipes

- [TypeScript Path Mapping](./typescript-relative-path.md) - For clean SVG imports
- [Utility-First Styling](../theming/utility-styling.md) - For styling SVG components
- [Theme System](../theming/theme-system-redux.md) - For dynamic SVG theming

---

Your SVG support is now configured! You can import and use SVG files as React components throughout your application.
