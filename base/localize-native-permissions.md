# Localizing Native Permissions in Expo (NSUserTrackingUsageDescription, etc.)

## Overview

This recipe explains how to localize native permission strings (such as NSUserTrackingUsageDescription) in a React Native Expo project. Many Expo and React Native libraries (e.g., expo-tracking-transparency) suggest adding permission strings directly in app.json/app.config.js, but this approach does not support internationalization.

With Expo, you can easily localize all native permission prompts by moving them to your localization files and enabling the correct flags.

---

## Why Localize Native Permissions?
- Native permission dialogs (e.g., for tracking, camera, notifications) must be shown in the user's language for App Store compliance and better UX.
- Hardcoding permission strings in app.json does not scale for multiple languages.

---

## Step 1: Move Permission Strings to Localization Files

Instead of hardcoding permission strings in your plugin config, move them to your language files (e.g., `src/locales/en.json`, `src/locales/ru.json`).

**Example `src/locales/en.json`:**
```json
{
  "NSUserTrackingUsageDescription": "This identifier will be used to deliver personalized ads to you."
}
```

---

## Step 2: Configure expo.locales and iOS InfoPlist

In your `app.json` or `app.config.js`, add the `locales` field and enable mixed localizations for iOS:

```json
{
  "expo": {
    "locales": {
      "en": "./src/locales/en.json",
    },
    "ios": {
      "infoPlist": {
        "CFBundleAllowMixedLocalizations": true
      }
    }
  }
}
```

- The `locales` field tells Expo where to find your localization files.
- The `CFBundleAllowMixedLocalizations` flag allows iOS to use your custom permission strings for each language.

---

## Step 3: Remove Hardcoded Permission Strings from Plugins

If you previously set permission strings directly in plugin configs (e.g., `expo-tracking-transparency`), remove them. The system will now use the localized values from your language files.

**Before:**
```json
{
  "expo": {
    "plugins": [
      [
        "expo-tracking-transparency",
        {
          "userTrackingPermission": "This identifier will be used to deliver personalized ads to you."
        }
      ]
    ]
  }
}
```

**After:**
```json
{
  "expo": {
    "plugins": ["expo-tracking-transparency"]
  }
}
```

---

## Step 4: Add All Required Native Keys

Add all required native permission keys (e.g., `NSCameraUsageDescription`, `NSPhotoLibraryUsageDescription`, etc.) to your localization files for each language.

**Example `src/locales/en.json`:**
```json
{
  "NSUserTrackingUsageDescription": "This identifier will be used to deliver personalized ads to you.",
  "NSCameraUsageDescription": "We need access to your camera to take photos.",
  "NSPhotoLibraryUsageDescription": "We need access to your photo library to select images."
}
```

---

## Best Practices
- Always provide translations for all required native keys in every supported language.
- Test your app on devices with different system languages to verify correct localization.
- Keep your localization files organized and up to date as you add new permissions.

---

## Summary

By using Expo's `locales` and iOS's `CFBundleAllowMixedLocalizations`, you can fully localize all native permission prompts in your app, ensuring a better user experience and App Store compliance.
