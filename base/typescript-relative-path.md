# TypeScript Path Mapping for React Native

## Overview

Configure TypeScript path mapping to enable clean, consistent imports throughout your React Native application. This recipe establishes import aliases that make your codebase more maintainable and professional.

## What You'll Achieve

✅ Clean imports: `import { Button } from '@/components'` instead of `'../../components'`  
✅ Separate aliases for source code (`@/*`) and assets (`~/*`)  
✅ Better code organization and maintainability  
✅ Consistent import patterns across your team  

## Prerequisites

- Expo project with TypeScript
- Basic understanding of TypeScript configuration

## Implementation

### Step 1: Update TypeScript Configuration

Modify your `tsconfig.json` file to include path mapping:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "jsx": "react-jsx",
    "resolveJsonModule": true,
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "~/*": ["./*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

### Step 2: Verify Configuration

Create a test file to ensure path mapping works correctly:

```typescript
// Test imports using the new aliases
import { SomeComponent } from '@/components'
import AppIcon from '~/assets/icon.png'

export const testImports = () => {
  console.log('Path mapping is working!')
}
```

## Usage Examples

### Source Code Imports (Using `@/*`)

```typescript
// Components
import { Header, Footer } from '@/components'
import { Button } from '@/components/core'

// Services  
import { StorageService, SecureStoreService } from '@/services'
import { SocialService } from '@/services'

// Utilities
import { formatDate } from '@/utils'
import { intl } from '@/utils/intl'

// Store
import { useDispatch, useSelector } from '@/hooks'
import { setTheme } from '@/store/slices/settings'

// API
import api from '@/api'
import { useGetProfileQuery } from '@/api/profile'
```

### Asset Imports (Using `~/*`)

```typescript
// Images
import AppIcon from '~/assets/icon.png'
import SplashImage from '~/assets/splash.png'
import BrandLogo from '~/assets/brand/logo-brand.svg'

// Animations
import AudioAnimation from '~/assets/animations/audio-light.json'

// App configuration
import AppConfig from '~/app.config'
import ThemeConfig from '~/theme.config'
```

## Real Project Examples

Based on the actual project structure, here are common import patterns:

```typescript
// Screen components
import { AuthScreen } from '@/screens/Auth'
import { ProfileScreen } from '@/screens/Profile'

// Feature components  
import { ChatMessage } from '@/features/chat'
import { BillingCard } from '@/features/billing'

// Core UI components
import { Text, Button, Div } from '@/components/core'
import { TextInput, DateInput } from '@/components'

// Guards and providers
import { AuthGuard } from '@/guards'
import { ThemeProvider } from '@/providers'

// Hooks and utilities
import { useTheme, useTranslations } from '@/hooks'
import { useAuth } from '@/hooks/auth'
import { $notify } from '@/utils'

// Store and API
import { RootState } from '@/store'
import { useRegisterMutation } from '@/api/auth'
```

## Best Practices

### ✅ Do

- Use `@/*` for all source code imports
- Use `~/*` for assets, configs, and root-level files
- Keep imports organized by category (components, services, utils)
- Use consistent naming conventions

### ❌ Don't

- Mix relative and absolute imports in the same file
- Use path mapping for external dependencies (npm packages)
- Create overly complex nested path structures

## IDE Configuration

Most modern IDEs automatically recognize TypeScript path mapping. For VS Code, ensure you have:

- TypeScript extension enabled
- Workspace using the correct TypeScript version
- Auto-import suggestions configured for custom paths

## Common Issues

### Issue: Path mapping not working in Metro bundler

Metro bundler may need additional configuration for complex path mappings. This is typically handled automatically in Expo projects.

### Issue: Import suggestions not working

Restart your TypeScript language server:
- **VS Code**: `Cmd/Ctrl + Shift + P` → "TypeScript: Restart TS Server"
- **WebStorm**: File → Invalidate Caches and Restart

## Related Recipes

- [Redux Toolkit Setup](../redux/basic-configuration.md) - Uses these path mappings for store configuration
- [Component Organization](#) - Coming soon
- [Service Layer Architecture](#) - Coming soon