# Advanced RTK Query Configuration for React Native

## Overview

This guide covers advanced patterns for configuring RTK Query in React Native, including:
- Handling FormData requests
- Global error interception and UI notification
- Automatic token refresh on 401 errors
- Custom base URLs per request
- Authorization flag for endpoints
- Suppressing error notifications
- Setting custom HTTP headers (language, auth, etc.)

All examples use generic names and constants for clarity.

---

## 1. FormData Helper

To send FormData (e.g., for file uploads), use a helper that converts objects to FormData using the `object-to-formdata` package.

```ts
import { serialize } from 'object-to-formdata'

export function toFormData(data: Record<string, any>) => serialize(data, { indices: true })
```

Use this helper in your query logic to transform the request body when needed.

---

## 2. Global Error Interceptor

To display server errors globally (e.g., via a Snackbar or Modal), implement an interceptor that checks for error responses and triggers your UI notification system.

```ts
function showErrorMessage(message?: string, args: any) {
  // Replace with your notification logic
  // if (isDebug) $alert.show(message ?? 'An unexpected error occurred', 'warning')
  // console.warn('API request error:', { message, args })
}
```

Call this function in your base query or custom query logic when an error is detected and error display is enabled.

---

## 3. Automatic Token Refresh (401 Handling)

When a request fails with a 401 Unauthorized error, automatically attempt to refresh the access token and retry the original request.

```ts
import { Mutex } from 'async-mutex'

const refreshMutex = new Mutex()

async function handle401AndRefresh(baseQuery, args, api, extraOptions, getRefreshToken, setTokens, clearTokens) {
  if (!refreshMutex.isLocked()) {
    const release = await refreshMutex.acquire()
    try {
      const refreshResult = await baseQuery({
        url: '/auth/refresh',
        method: 'POST',
        body: { refresh_token: getRefreshToken() },
      }, api, extraOptions)
      if (refreshResult.data) {
        setTokens(refreshResult.data)
        return await baseQuery(args, api, extraOptions)
      } else {
        clearTokens()
        showErrorMessage('Session expired', args)
      }
    } finally {
      release()
    }
  } else {
    return await baseQuery(args, api, extraOptions)
  }
}
```

---

## 4. Custom Base URL per Request

Allow endpoints to override the default API base URL by passing a `customBaseUrl` option.

```ts
function getBaseQuery(customBaseUrl?: string) {
  return fetchBaseQuery({ baseUrl: customBaseUrl || DEFAULT_API_URL })
}

// Usage in your query logic:
const baseQuery = getBaseQuery(extraOptions.customBaseUrl)
```

---

## 5. Authorization Flag

Add an `authorized` flag to endpoint options to require or forbid authentication for specific requests. If the flag is set and the auth state does not match, throw an error or show a notification.

```ts
function checkAuthorization(authorized: boolean | undefined, hasToken: boolean, args: any) {
  if (typeof authorized === 'boolean') {
    if ((authorized && !hasToken) || (!authorized && hasToken)) {
      showErrorMessage('Authorization conflict', args)
      throw new Error('Authorization conflict')
    }
  }
}
```

---

## 6. Suppressing Error Notifications

Allow endpoints to disable global error notifications by passing `withError: false` in extra options.

```ts
if (extraOptions.withError !== false) {
  showErrorMessage(error.message, args)
}
```

---

## 7. Setting Custom HTTP Headers

Set headers such as language, authorization, etc., in the `prepareHeaders` function of your base query.

```ts
const baseQuery = fetchBaseQuery({
  baseUrl: DEFAULT_API_URL,
  prepareHeaders: (headers, { getState }) => {
    const state = getState() as RootState
    const { token } = state.auth
    const { locale } = state.settings

    if (locale) headers.set('Accept-Language', locale)
    if (token) headers.set('Authorization', `Bearer ${token}`)
    return headers
  }
})
```

---


## Full Example: Advanced RTK Query Integration

Below is a complete example that demonstrates how to set up a custom base query, configure your API slice, and use endpoint flags for mutation and query requests.

### 1. Custom Base Query Implementation

```ts
// src/api/customBaseQuery.ts
import { fetchBaseQuery, BaseQueryFn, FetchArgs, FetchBaseQueryError } from '@reduxjs/toolkit/query/react'
import { Mutex } from 'async-mutex'
import { serialize } from 'object-to-formdata'

const API_URL = 'https://api.example.com'
const refreshMutex = new Mutex()

function toFormData(data: Record<string, any>, method?: string) => serialize({ ...data, _method: 'PUT' }, { indices: true })

function showErrorMessage(message?: string, args: any) {
  // Replace with your notification logic
  // if (isDebug) $alert.show(message ?? 'An unexpected error occurred', 'warning')
  // console.warn('API request error:', { message, args })
}

function checkAuthorization(authorized: boolean | undefined, hasToken: boolean, args: any) {
  if (typeof authorized === 'boolean') {
    if ((authorized && !hasToken) || (!authorized && hasToken)) {
      showErrorMessage('Authorization conflict', args)
      throw new Error('Authorization conflict')
    }
  }
}

async function handle401AndRefresh(baseQuery, args, api, extraOptions, getRefreshToken, setTokens, clearTokens) {
  if (!refreshMutex.isLocked()) {
    const release = await refreshMutex.acquire()
    try {
      const refreshResult = await baseQuery({
        url: '/auth/refresh',
        method: 'POST',
        body: { refresh_token: getRefreshToken() },
      }, api, extraOptions)
      if (refreshResult.data) {
        setTokens(refreshResult.data)
        return await baseQuery(args, api, extraOptions)
      } else {
        clearTokens()
        showErrorMessage('Session expired', args)
      }
    } finally {
      release()
    }
  } else {
    return await baseQuery(args, api, extraOptions)
  }
}

export const customBaseQuery: BaseQueryFn<any, unknown, FetchBaseQueryError, any> = async (args, api, extraOptions = {}) => {
  // 1. Authorization check
  checkAuthorization(extraOptions.authorized, !!api.getState().auth?.token, args)

  // 2. FormData
  if (extraOptions.withFormData && typeof args !== 'string') {
    args.body = toFormData(args.body)
  }

  // 3. Custom base URL
  const baseQuery = fetchBaseQuery({
    baseUrl: extraOptions.customBaseUrl || API_URL,
    prepareHeaders: (headers, { getState }) => {
      const state = getState() as RootState
      const { token } = state.auth
      const { locale } = state.settings
      if (locale) headers.set('Accept-Language', locale)
      if (token) headers.set('Authorization', `Bearer ${token}`)
      return headers
    }
  })

  // 4. Make request
  let result = await baseQuery(args, api, extraOptions)

  // 5. Handle errors
  if (result.error) {
    if (result.error.status === 401) {
      // Replace with your token accessors
      result = await handle401AndRefresh(baseQuery, args, api, extraOptions, () => api.getState().auth?.refreshToken, (tokens) => api.dispatch({ type: 'auth/setTokens', payload: tokens }), () => api.dispatch({ type: 'auth/clearTokens' }))
    } else if (extraOptions.withError !== false) {
      showErrorMessage(result.error.message, args)
    }
  }

  return result
}
```

### 2. API Slice Setup

```ts
// src/api/index.ts
import { createApi } from '@reduxjs/toolkit/query/react'
import { customBaseQuery } from './customBaseQuery'

export const api = createApi({
  baseQuery: customBaseQuery,
  reducerPath: 'api',
  tagTypes: ['User', 'Post'],
  endpoints: () => ({}),
})
```

### 3. Example Endpoint with Flags

```ts
// src/api/example.ts
import { api } from './index'

type ExampleRequest = { foo: string; file?: File }
type ExampleResponse = { result: string }

export const exampleApi = api.injectEndpoints({
  endpoints: (build) => ({
    uploadFile: build.mutation<ExampleResponse, ExampleRequest>({
      query: (body) => ({
        url: '/upload',
        method: 'POST',
        body,
      }),
      extraOptions: {
        authorized: true, // Require auth
        withFormData: true, // Convert body to FormData
        withError: true, // Show global error
        customBaseUrl: 'https://uploads.example.com', // Use custom URL
      },
    }),
    fetchData: build.query<ExampleResponse, void>({
      query: () => ({
        url: '/data',
        method: 'GET',
      }),
      extraOptions: {
        authorized: false, // Forbid auth
        withError: false, // Suppress error notification
      },
    }),
  }),
})

export const { useUploadFileMutation, useFetchDataQuery } = exampleApi
```

---

This full example demonstrates how to:
- Centralize all advanced API logic in a custom base query
- Use endpoint flags for per-request behavior (auth, error, FormData, custom URL)
- Integrate with RTK Query's API slice and hooks

You can now use `useUploadFileMutation` and `useFetchDataQuery` in your components, and all advanced behaviors will be handled automatically.
