# React Native Code Recipes
Production-ready code recipes and architect

## Legend

- ✅ **Ready**: Complete and tested
- 🚧 **In Progress**: Currently being written
- 📋 **Planned**: Scheduled for developmenttterns for React Native development

## About This Repository

This repository contains a curated collection of React Native code recipes and best practices. Each recipe provides step-by-step implementation guides for common development challenges, tested in real-world applications.

🌟 **Star this repository** if you find these recipes helpful for your React Native projects!

## Architecture Overview

All recipes follow a consistent architectural approach:

- **Platform**: Expo with Managed Workflow
- **Language**: TypeScript for enhanced developer experience
- **Structure**: Modular architecture with organized `src/` directory
- **Imports**: Clean path mapping with `@/*` for source and `~/*` for assets
- **State**: Redux Toolkit with persistence and real-time capabilities

## Recipe Categories

### 🏗️ Foundation & Setup
Essential configuration recipes for project setup.

| Recipe | Description | Complexity | Status |
|--------|-------------|------------|--------|
| [TypeScript Path Mapping](./base/typescript-relative-path.md) | Configure clean import aliases | 🟢 Basic | ✅ Ready |
| [SVG Support via Metro](./base/svg-support-metro.md) | Configure Metro bundler for SVG files | 🟢 Basic | ✅ Ready |
| [Dynamic Import Context](./base/dynamic-import-context.md) | Dynamic loading with require.context | 🟡 Intermediate | ✅ Ready |
| [FullWindowOverlay Usage](./base/full-window-overlay.md) | Overlay UI above native navigation (iOS) | 🟡 Intermediate | ✅ Ready |

### 🔄 State Management  
Redux Toolkit patterns with persistence and listeners.

| Recipe | Description | Complexity | Status |
|--------|-------------|------------|--------|
| [Redux Toolkit Complete Setup](./redux/basic-configuration.md) | Full Redux configuration with persistence | 🟡 Intermediate | ✅ Ready |

### 🎨 Theming & Styling
Theme systems and styling approaches.

| Recipe | Description | Complexity | Status |
|--------|-------------|------------|--------|
| [Theme System with Redux](./theming/theme-system-redux.md) | Complete theme system with Redux sync | 🟡 Intermediate | ✅ Ready |
| [Utility-First Styling](./theming/utility-styling.md) | createStyles utility for efficient styling | 🟢 Basic | ✅ Ready |

### 🛡️ Guards & Architecture
Architectural patterns and application guards.

| Recipe | Description | Complexity | Status |
|--------|-------------|------------|--------|
| [Guard Pattern](./architecture/guard-pattern.md) | Implement application guard system | 🟡 Intermediate | ✅ Ready |

### 🔧 Providers & Context
Essential providers for React Native applications.

| Recipe | Description | Complexity | Status |
|--------|-------------|------------|--------|
| [Essential Providers Stack](./providers/essential-providers.md) | Network, AppState, Notification providers | 🟡 Intermediate | 📋 Planned |

### 🔐 Authentication & Security
Authentication patterns and security implementations.

| Recipe | Description | Complexity | Status |
|--------|-------------|------------|--------|
| [Universal Social Auth](./best-practices/social-auth-using-browser.md) | Cross-platform social authentication | 🟡 Intermediate | ✅ Ready |


### � Coming Soon
- **Navigation**: Routing patterns and navigation strategies
- **API Integration**: RTK Query patterns and data fetching
- **Performance**: Optimization techniques and best practices
- **Testing**: Testing strategies for React Native apps
- **Deployment**: CI/CD and release automation
- **UI Components**: Reusable component patterns

## Quick Start

1. Browse the recipe categories above
2. Select a recipe that matches your needs
3. Follow the step-by-step implementation guide
4. Customize the code for your specific use case

## Recipe Structure

Each recipe includes:
- **🎯 Overview**: What the recipe accomplishes
- **📋 Prerequisites**: Required setup and dependencies
- **🔧 Implementation**: Detailed code examples
- **💡 Usage**: Practical implementation examples
- **⚡ Best Practices**: Tips and common pitfalls

## Contributing

We welcome contributions! Feel free to:
- 🐛 Report issues or bugs
- 💡 Suggest new recipes
- 🔄 Submit improvements via pull requests
- ⭐ Star the repository if you find it useful

---

*Crafted with ❤️ for the React Native community*

