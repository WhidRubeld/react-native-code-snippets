# Expo Asset Helpers: Document & Image Conversion

## Overview

This recipe provides utility functions for working with assets from `expo-document-picker` and `expo-image-picker` in React Native. It covers:
- Converting picked images to base64 data URLs
- Converting picked documents or images to `File` objects for uploads

These helpers simplify asset handling for uploads, previews, and API integration.

---

## 1. Convert Image to Base64 Data URL

Use `imageToBase64DataUrl` to convert an `ImagePickerAsset` to a base64-encoded data URL string.

```ts
import { ImagePickerAsset } from 'expo-image-picker'

export function imageToBase64DataUrl(image: ImagePickerAsset): string {
  const { uri, base64 } = image
  const name = uri.split('/').pop() || ''
  const ext = name.split('.').pop() || 'jpeg'
  const type = `image/${ext}`
  return `data:${type};base64,${base64}`
}
```

**Usage:**
```ts
const dataUrl = imageToBase64DataUrl(imageAsset)
```

---

## 2. Convert Asset to File Object

Use `assetToFile` to convert a `DocumentPickerAsset` or `ImagePickerAsset` to a `File` object suitable for FormData uploads.

```ts
import { DocumentPickerAsset } from 'expo-document-picker'
import { ImagePickerAsset } from 'expo-image-picker'

export async function assetToFile(asset: DocumentPickerAsset | ImagePickerAsset): Promise<File> {
  const { uri } = asset
  const fileName = uri.split('/').pop() || 'file'
  const response = await fetch(uri)
  const blob = await response.blob()
  // @ts-ignore
  return new File([blob], fileName, { type: blob.type })
}
```

**Usage:**
```ts
const file = await assetToFile(documentOrImageAsset)
```

---

## Best Practices
- Always check for the presence of `base64` in the image asset before using `imageToBase64DataUrl`.
- Use `assetToFile` for uploading files via FormData with APIs that expect `File` objects.
- Handle errors for missing file names or fetch failures.

---

These helpers streamline asset processing for uploads and previews in Expo-based React Native apps.
