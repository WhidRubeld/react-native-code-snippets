# Dynamic Import with require.context

## Overview

Implement dynamic file importing using Metro bundler's `require.context` functionality. This recipe shows how to dynamically load local project files at runtime while understanding Metro bundler limitations and best practices.

## What You'll Achieve

✅ Dynamic loading of local project files  
✅ Runtime file discovery and imports  
✅ Type-safe dynamic imports with TypeScript  
✅ Proper error handling for missing files  
✅ Metro bundler optimization compatibility  
✅ Fallback strategies for failed imports  

## Prerequisites

- Metro bundler configured with `allowRequireContext: true`
- TypeScript path mapping ([see recipe](./typescript-relative-path.md))
- Understanding of Metro bundler limitations

## Architecture

This recipe creates a dynamic import system with the following principles:

```
src/
├── utils/
│   └── dynamicLoader.ts       # Dynamic loader utility
├── assets/                    # Local files for dynamic import
│   ├── file1.json
│   ├── file2.json
│   └── ...
└── constants/
    └── files.ts               # File registry
```

## Metro Configuration

First, ensure Metro is configured for `require.context`. Update `metro.config.js`:

```javascript
// metro.config.js

const { getDefaultConfig } = require('expo/metro-config')

module.exports = (async () => {
  const config = await getDefaultConfig(__dirname)

  const { transformer, resolver } = config

  config.transformer = {
    ...transformer,
    allowRequireContext: true, // Enable require.context
  }

  config.resolver = {
    ...resolver,
    sourceExts: [...resolver.sourceExts, 'json'],
  }

  return config
})()
```

## Implementation

### Step 1 - Create dynamic loader utility

Create a dynamic file loader that handles local files. Add to `src/utils/dynamicLoader.ts`:

```typescript
// src/utils/dynamicLoader.ts

/**
 * Dynamic file loader using require.context
 * Works only with local project files, not node_modules
 */

export interface LoaderConfig {
  directory: string
  recursive?: boolean
  pattern?: RegExp
  fallback?: any
}

export interface LoaderState {
  loading: boolean
  loaded: Set<string>
  cache: Map<string, any>
  error: string | null
}

/**
 * Dynamic file loader class
 */
export class DynamicFileLoader<T = any> {
  private context: any
  private state: LoaderState
  private fallbackData: T | null

  constructor(config: LoaderConfig) {
    this.state = {
      loading: false,
      loaded: new Set(),
      cache: new Map(),
      error: null,
    }
    
    this.fallbackData = config.fallback || null
    this.initializeContext(config)
  }

  /**
   * Initialize require.context for the specified directory
   * NOTE: Only works with local project files
   */
  private initializeContext(config: LoaderConfig) {
    try {
      // require.context parameters:
      // 1. directory: relative path from current file
      // 2. recursive: scan subdirectories
      // 3. pattern: file pattern regex
      this.context = require.context(
        config.directory,
        config.recursive || false,
        config.pattern || /\.(json|js|ts)$/
      )
      
      console.log(`✅ Context initialized for: ${config.directory}`)
      console.log(`📁 Available files:`, this.context.keys())
    } catch (error) {
      console.warn('❌ Failed to initialize context:', error)
      this.context = null
      this.state.error = 'Context initialization failed'
    }
  }

  /**
   * Get all available file keys
   * Returns empty array if context failed to initialize
   */
  public getAvailableFiles(): string[] {
    if (!this.context) return []
    
    try {
      return this.context.keys()
    } catch (error) {
      console.warn('Failed to get context keys:', error)
      return []
    }
  }

  /**
   * Load file dynamically with caching
   */
  public async loadFile(fileName: string): Promise<T | null> {
    // Check cache first
    if (this.state.cache.has(fileName)) {
      return this.state.cache.get(fileName)
    }

    // Check if context is available
    if (!this.context) {
      console.warn('Context not available, using fallback')
      return this.fallbackData
    }

    this.state.loading = true
    this.state.error = null

    try {
      const availableFiles = this.getAvailableFiles()
      const targetFile = `./${fileName}`

      // Check if file exists in context
      if (!availableFiles.includes(targetFile)) {
        throw new Error(`File not found: ${fileName}`)
      }

      // Load file using context
      const fileData = this.context(targetFile)
      
      // Cache the result
      this.state.cache.set(fileName, fileData)
      this.state.loaded.add(fileName)

      console.log(`✅ Loaded file: ${fileName}`)
      return fileData
    } catch (error) {
      console.error(`❌ Failed to load file ${fileName}:`, error)
      this.state.error = `Failed to load: ${fileName}`
      
      // Return fallback if available
      return this.fallbackData
    } finally {
      this.state.loading = false
    }
  }

  /**
   * Load multiple files concurrently
   */
  public async loadMultipleFiles(fileNames: string[]): Promise<(T | null)[]> {
    const loadPromises = fileNames.map(fileName => this.loadFile(fileName))
    return Promise.allSettled(loadPromises).then(results =>
      results.map(result => 
        result.status === 'fulfilled' ? result.value : null
      )
    )
  }

  /**
   * Preload all available files
   */
  public async preloadAllFiles(): Promise<void> {
    const availableFiles = this.getAvailableFiles()
    const fileNames = availableFiles.map(file => file.replace('./', ''))
    
    await this.loadMultipleFiles(fileNames)
    console.log(`📦 Preloaded ${availableFiles.length} files`)
  }

  /**
   * Check if file is loaded
   */
  public isFileLoaded(fileName: string): boolean {
    return this.state.loaded.has(fileName)
  }

  /**
   * Get loader state
   */
  public getState(): LoaderState {
    return { ...this.state }
  }

  /**
   * Clear cache
   */
  public clearCache(): void {
    this.state.cache.clear()
    this.state.loaded.clear()
  }
}
```

### Step 2 - Create file registry

Define file registry for better organization. Add to `src/constants/files.ts`:

```typescript
// src/constants/files.ts

/**
 * File registry for dynamic imports
 * Helps with type safety and organization
 */

export const AVAILABLE_CONFIGS = [
  'theme-light.json',
  'theme-dark.json',
  'app-settings.json',
  'feature-flags.json',
] as const

export const AVAILABLE_TRANSLATIONS = [
  'en.json',
  'es.json',
  'fr.json',
  'de.json',
] as const

export type ConfigFile = typeof AVAILABLE_CONFIGS[number]
export type TranslationFile = typeof AVAILABLE_TRANSLATIONS[number]

/**
 * File validation utilities
 */
export const isValidConfigFile = (fileName: string): fileName is ConfigFile => {
  return AVAILABLE_CONFIGS.includes(fileName as ConfigFile)
}

export const isValidTranslationFile = (fileName: string): fileName is TranslationFile => {
  return AVAILABLE_TRANSLATIONS.includes(fileName as TranslationFile)
}
```

### Step 3 - Create specialized loaders

Create specialized loaders for different file types:

```typescript
// src/utils/configLoader.ts

import { DynamicFileLoader } from './dynamicLoader'
import { ConfigFile, AVAILABLE_CONFIGS } from '@/constants/files'

interface AppConfig {
  version: string
  features: Record<string, boolean>
  settings: Record<string, any>
}

/**
 * Configuration file loader
 * Handles JSON configuration files with fallbacks
 */
export class ConfigLoader extends DynamicFileLoader<AppConfig> {
  constructor() {
    super({
      directory: '../assets/configs', // Relative to this file
      recursive: false,
      pattern: /\.json$/,
      fallback: {
        version: '1.0.0',
        features: {},
        settings: {},
      },
    })
  }

  /**
   * Load configuration with validation
   */
  public async loadConfig(configName: ConfigFile): Promise<AppConfig> {
    const config = await this.loadFile(configName)
    
    // Validate config structure
    if (config && this.validateConfig(config)) {
      return config
    }

    console.warn(`Invalid config structure for: ${configName}`)
    return this.fallbackData!
  }

  /**
   * Validate configuration structure
   */
  private validateConfig(config: any): config is AppConfig {
    return (
      typeof config === 'object' &&
      typeof config.version === 'string' &&
      typeof config.features === 'object' &&
      typeof config.settings === 'object'
    )
  }

  /**
   * Get available configuration files
   */
  public getAvailableConfigs(): ConfigFile[] {
    const available = this.getAvailableFiles()
    return AVAILABLE_CONFIGS.filter(config => 
      available.includes(`./${config}`)
    )
  }
}

// Export singleton instance
export const configLoader = new ConfigLoader()
```

### Step 4 - Create node_modules fallback strategy

For external dependencies (like date libraries), use static imports:

```typescript
// src/utils/staticImportFallback.ts

/**
 * Static import strategy for node_modules
 * require.context doesn't work with external packages
 */

// Static mapping for external dependencies
const STATIC_LOCALE_LOADERS: Record<string, () => any> = {
  en: () => require('some-library/locales/en'),
  es: () => require('some-library/locales/es'),
  fr: () => require('some-library/locales/fr'),
  de: () => require('some-library/locales/de'),
}

/**
 * Load external package files statically
 * Use this pattern when require.context returns empty keys
 */
export class StaticImportLoader {
  private loadedModules = new Set<string>()
  
  /**
   * Load external module locale
   */
  public loadLocale(locale: string): boolean {
    try {
      const loader = STATIC_LOCALE_LOADERS[locale]
      
      if (!loader) {
        console.warn(`⚠️ Locale not available: ${locale}`)
        return false
      }

      // Execute static import
      loader()
      this.loadedModules.add(locale)
      console.log(`✅ Loaded external locale: ${locale}`)
      return true
    } catch (error) {
      console.warn(`⚠️ Failed to load locale ${locale}:`, error)
      return false
    }
  }

  /**
   * Load multiple locales
   */
  public loadMultipleLocales(locales: string[]): void {
    locales.forEach(locale => this.loadLocale(locale))
  }

  /**
   * Check if module is loaded
   */
  public isModuleLoaded(module: string): boolean {
    return this.loadedModules.has(module)
  }
}

export const staticLoader = new StaticImportLoader()
```

## Usage Examples

### Basic dynamic file loading

```typescript
// src/services/AppInitializer.ts

import { configLoader } from '@/utils/configLoader'
import { DynamicFileLoader } from '@/utils/dynamicLoader'

export class AppInitializer {
  private translationLoader: DynamicFileLoader

  constructor() {
    // Initialize translation loader
    this.translationLoader = new DynamicFileLoader({
      directory: '../locales',
      recursive: false,
      pattern: /\.json$/,
      fallback: { hello: 'Hello' },
    })
  }

  /**
   * Initialize app with dynamic loading
   */
  public async initialize(): Promise<void> {
    try {
      // Load app configuration
      const config = await configLoader.loadConfig('app-settings.json')
      console.log('App config loaded:', config)

      // Load translation dynamically
      const translation = await this.translationLoader.loadFile('en.json')
      console.log('Translation loaded:', translation)

      // Preload all available translations
      await this.translationLoader.preloadAllFiles()
      
    } catch (error) {
      console.error('App initialization failed:', error)
    }
  }
}
```

### Runtime language switching

```typescript
// src/services/LocalizationService.ts

import { DynamicFileLoader } from '@/utils/dynamicLoader'

export class LocalizationService {
  private loader: DynamicFileLoader<Record<string, string>>
  private currentLanguage = 'en'

  constructor() {
    this.loader = new DynamicFileLoader({
      directory: '../locales',
      pattern: /\.json$/,
      fallback: {},
    })
  }

  /**
   * Change language dynamically
   */
  public async changeLanguage(language: string): Promise<boolean> {
    try {
      const translations = await this.loader.loadFile(`${language}.json`)
      
      if (translations) {
        this.currentLanguage = language
        console.log(`Language changed to: ${language}`)
        return true
      }
      
      return false
    } catch (error) {
      console.error('Language change failed:', error)
      return false
    }
  }

  /**
   * Get available languages from context
   */
  public getAvailableLanguages(): string[] {
    return this.loader
      .getAvailableFiles()
      .map(file => file.replace('./', '').replace('.json', ''))
  }
}
```

## Best Practices

### ✅ Do

- Use require.context for local project files only
- Implement fallback strategies for missing files
- Cache loaded files to avoid repeated imports
- Validate file structure after loading
- Use TypeScript for type safety
- Log import success/failure for debugging

### ❌ Don't

- Try to use require.context with node_modules (returns empty array)
- Ignore error handling for missing files
- Load files synchronously in main thread
- Hardcode file paths without validation
- Skip fallback data for critical files

### ⚠️ Metro Bundler Limitations

- **node_modules exclusion**: require.context only works with local project files
- **Static analysis**: Metro needs to analyze requires at build time
- **Pattern limitations**: Complex regex patterns may not work as expected
- **Directory restrictions**: Must use relative paths from the calling file

## Troubleshooting

### Empty context keys array

```typescript
// Check if context is working
const context = require.context('../locales', false, /\.json$/)
console.log('Context keys:', context.keys()) // Should not be empty

// If empty, context failed - use static imports instead
```

### File not found errors

```typescript
// Always check if file exists before loading
const availableFiles = loader.getAvailableFiles()
const targetFile = `./${fileName}`

if (!availableFiles.includes(targetFile)) {
  console.warn(`File not available: ${fileName}`)
  return fallbackData
}
```

### Metro bundler issues

```typescript
// Ensure Metro config includes allowRequireContext
config.transformer = {
  ...transformer,
  allowRequireContext: true, // Required for require.context
}
```

## Related Recipes

- [TypeScript Path Mapping](./typescript-relative-path.md) - Clean import paths
- [Metro SVG Support](./svg-support-metro.md) - Metro bundler configuration
- [Theme System](../theming/theme-system-redux.md) - Dynamic theme loading

---

Your dynamic import system is now ready! You can load local project files dynamically while handling Metro bundler limitations gracefully.
