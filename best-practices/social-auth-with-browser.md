# Setting up Social Authentication for React Native Using Web Browsers

## Introduction
Sometimes implementing authentication through social networks needs to be done quickly. While most social networks provide their own SDKs for handling OAuth2 authorization in mobile applications, there are situations where using a web browser approach is simpler:

1. When developing multiple applications simultaneously for the same project (both native and web interfaces)
2. When you want to avoid dealing with multiple SDKs and their specific issues
3. When you need to accelerate the development process
4. This tutorial demonstrates a universal authentication method for social networks using a web browser window. You'll only need a website with a redirect page. The exception is Apple ID authentication, which cannot be handled via HTTP routes and will be implemented differently.

We'll cover authentication via:
1. Apple
2. Facebook
This approach can be extended to any social network that supports OAuth2.

This example is part of the React Native Code Snippets codebase. You can find the repository [here](https://github.com/WhidRubeld/react-native-code-snippets), which contains many useful implementations to help with your React Native projects.

## Project Architecture

This tutorial assumes that you are working with an [Expo project](https://expo.dev) in the [Managed Workflow](https://docs.expo.dev/guides/managed-workflow/). Additionally, the project uses [TypeScript](https://www.typescriptlang.org/), providing static type checking and enhanced development experience. 

All future services (include Social auth) will be located in the `./src/services/*` directory.

## Step 1 - Configure TypeScript relative paths

Set up TypeScript relative path prefixes for your project according to the [instructions](../base/typescript-relative-path.md).

## Step 2 - Installing packages  in Your React Native App

To install the following packages using `yarn`, run the command:

```bash
yarn add queryString
```

Install Expo packages that handle native functionality:

```bash
npx expo install expo-web-browser expo-linking expo-apple-authentication
```

## Step 3 - Create the Social Authentication Service

```ts
// ./services/SocialService.ts

import * as AppleAuthentication from 'expo-apple-authentication'
import * as Linking from 'expo-linking'
import {
  maybeCompleteAuthSession,
  openAuthSessionAsync,
} from 'expo-web-browser'
import queryString from 'query-string'

maybeCompleteAuthSession()

const WEB_URL = 'YOUR_WEB_URL' // TODO
const REDIRECT_URI = encodeURIComponent(`${WEB_URL}/redirect`)
const RETURN_URI = Linking.createURL('oauth')

export type SocialResponseProps = {
  driver: SocialDriver
  [key: string]: any
}

export enum SocialDriver {
  apple = 'apple',
  google = 'google',
  facebook = 'facebook',
}
export default class SocialService {
  static async authorize({
    driver,
    url,
    clientId,
    inputIdentityKey,
    outputIdentityKey,
    scope,
    opts,
  }: {
    driver: SocialDriver
    url: string
    clientId: string
    inputIdentityKey: string
    outputIdentityKey: string
    scope?: string
    opts?: string
  }) {
    const computedUri =
      url +
      '?' +
      `client_id=${clientId}` +
      (scope ? `&scope=${scope}` : '') +
      `&redirect_uri=${REDIRECT_URI}` +
      `&state=${btoa(
        JSON.stringify({
          return_uri: RETURN_URI,
          // NOTE(dev): may need additional props for backend (for example - CSRF token), if is another backend
          // NOTE(dev): see Step 5 for adding redirect in your website
        }),
      )}` +
      (opts ? `&${opts}` : '')

    return new Promise<SocialResponseProps>((resolve, reject) => {
      openAuthSessionAsync(computedUri, RETURN_URI)
        .then((res: { type: string; url?: string | null }) => {
          const { url } = res
          if (!url) return reject(new Error(`${driver} return URL not found`))

          const { query } = queryString.parseUrl(url)
          const identityValue = query[inputIdentityKey] as string | null

          if (!identityValue)
            return reject(new Error(`${driver} auth key not found`))
          delete query[inputIdentityKey]

          resolve({
            driver,
            [outputIdentityKey]: identityValue,
            redirect_uri: REDIRECT_URI,
            ...query,
          })
        })
        .catch(reject)
    })
  }

  static async appleLogin() {
    return new Promise<SocialResponseProps>((resolve, reject) => {
      AppleAuthentication.signInAsync({
        requestedScopes: [
          AppleAuthentication.AppleAuthenticationScope.FULL_NAME,
          AppleAuthentication.AppleAuthenticationScope.EMAIL,
        ],
      })
        .then((credential) => {
          const { identityToken, ...opts } = credential
          if (!identityToken)
            return reject(new Error(`${SocialDriver.apple} auth key not found`))

          resolve({
            driver: SocialDriver.apple,
            token: identityToken,
            ...opts,
          })
        })
        .catch(reject)
    })
  }

  static login(driver: SocialDriver, clientId?: string) {
    if (driver !== SocialDriver.apple && !clientId) {
      return Promise.reject(
        new Error(`Client ID is required for ${driver} driver`),
      )
    }

    switch (driver) {
      case SocialDriver.apple: {
        return SocialService.appleLogin()
      }
      case SocialDriver.google: {
        return SocialService.authorize({
          driver: SocialDriver.google,
          clientId: clientId!,
          url: 'https://accounts.google.com/o/oauth2/v2/auth',
          inputIdentityKey: 'access_token',
          outputIdentityKey: 'token',
          scope: 'https://www.googleapis.com/auth/userinfo.email',
          opts: 'response_type=token&include_granted_scopes=true',
        })
      }
      case SocialDriver.facebook: {
        return SocialService.authorize({
          driver: SocialDriver.facebook,
          clientId: clientId!,
          url: 'https://www.facebook.com/v20.0/dialog/oauth',
          inputIdentityKey: 'code',
          outputIdentityKey: 'code',
          scope: 'public_profile',
        })
      }
      default: {
        return Promise.reject(new Error('Not found social variant'))
      }
    }
  }
}
```

## Step 4 - Export the Service in the Index File

```ts
// ./services/index.ts

export {
  default as SocialService,
  SocialDriver,
  SocialResponseProps,
} from './SocialService'
```

## Step 5 - Create a Redirect Page on Your Website
Create a `/redirect` page on your website with a loading spinner to indicate the transition process. Here's the JavaScript code to handle redirects:
```js
// Extract parameters from query string
function extractParams(queryString) {
  const params = new URLSearchParams(queryString)
  const state = params.get('state')
  params.delete('state')
  return { state, cleanedParams: params }
}

// Check if URI is valid
function isValidUri(uri) {
  return uri.includes('://')
}

const NATIVE = 'native'
const RETURN_KEY = 'return_uri'

// Decode state parameter
function decodeState(state) {
  try {
    const data = JSON.parse(atob(state))
    const status =
      !Array.isArray(data) &&
      Object.keys(data).includes(RETURN_KEY) &&
      typeof data[RETURN_KEY] === 'string'

    if (!status) {
      return { redirectUri: state, params: null }
    }

    const opts = Object.keys(data)
      .filter((key) => key !== RETURN_KEY)
      .map((key) => ({ key, value: data[key] }))

    if (!opts.length) {
      return { redirectUri: data[RETURN_KEY], params: null }
    }

    const params = new URLSearchParams()
    opts.forEach(({ key, value }) => {
      params.append(key, value)
    })

    return { redirectUri: data[RETURN_KEY], params }
  } catch (e) {
    return { redirectUri: state, params: null }
  }
}

// Handle redirect logic
function handleRedirect() {
  const searchParams = window.location.search
  const hashParams = window.location.hash.substring(1)

  const searchObj = extractParams(searchParams)
  const hashObj = extractParams(hashParams)

  const rawState = hashObj.state || searchObj.state
  if (!rawState) {
    return
  }

  const { redirectUri, params } = decodeState(rawState)
  if (redirectUri === NATIVE || !isValidUri(redirectUri)) {
    return
  }

  const combinedParams = new URLSearchParams(searchObj.cleanedParams)
  hashObj.cleanedParams.forEach((value, key) =>
    combinedParams.append(key, value),
  )
  if (params) {
    params.forEach((value, key) => combinedParams.append(key, value))
  }

  const finalParams = combinedParams.toString()
  setTimeout(() => {
    window.location.replace(
      `${redirectUri}${finalParams ? `?${finalParams}` : ''}`,
    )
  }, 1500)
}

// Call the redirect handler
handleRedirect()
```

## Implementation in Your App
Call the service method in your React Native application:

```tsx
import { SocialDriver, SocialService } from '@/services'

SocialService.login(SocialDriver.google, 'YOUR_GOOGLE_CLIENT_ID')
  .then((res) => {
    // server validation or other processing
    console.log('Google login response:', res)
  })
  .catch((e) => console.warn('Google login error:', e))
```

By following this approach, you can implement social authentication in your React Native application without dealing with the complexities of individual SDKs while maintaining a consistent user experience across different authentication providers.




