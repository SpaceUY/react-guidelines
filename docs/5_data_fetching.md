---
title: Data fetching
layout: default
nav_order: 5
---

# Data fetching

**Technology Review**  
_Written by: Federico Castañares, Facundo Panizza, Enzo Corrales_

## RTK Query vs React Query

## RTK Query

RTK Query is a powerful tool integrated with Redux Toolkit, specifically designed for fetching and caching server state efficiently.

### Pros:

- Easy-to-use caching and cache invalidation.
- Built-in loading states and error handling.

### Cons:

- Best used with Redux Toolkit; familiarity with Redux helps but isn't strictly required.
- More boilerplate required

### Example:

```ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Users'],
  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => 'users',
      providesTags: ['Users'],
    }),
    createUser: builder.mutation({
      query: (body) => ({
        url: 'users',
        method: 'POST',
        body,
      }),
      invalidatesTags: ['Users'],
    }),
  }),
});

export const { useGetUsersQuery } = api;
```

This creates a cache for `getUsers`. When a new user is created, the cache automatically refreshes.

Usage:

```tsx
import { useGetUsersQuery } from './api';

const UsersList = () => {
  const { data, isLoading, isError } = useGetUsersQuery();

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Error fetching data</p>;

  return (
    <div>
      <h1>Users List</h1>
      <ul>
        {data.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};

export default UsersList;
```

## React Query

React Query is a popular library designed specifically for managing server state, data fetching, and caching seamlessly.

### Pros:

- Automatic caching, refetching, and background syncing.
- Minimal boilerplate.
- Built-in support for loading, error, and success states.
- Ideal for apps with many GET requests.

### Cons:

- Primarily for server state; not intended for UI state.

### Example:

```tsx
import { useQuery } from '@tanstack/react-query';
import axios from 'axios';

function UsersList() {
  const { data, isLoading, isError } = useQuery({
    queryKey: ['users'],
    queryFn: () => axios.get('/api/users').then((res) => res.data),
  });

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Error fetching data</p>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

## Additional information

This table created by tanstack ( creators of React Query ) is a helpful visual aid to quickly see underlying differences
[Comparative table](https://tanstack.com/query/v4/docs/framework/react/comparison)

## Fetch vs Axios

Both Fetch and Axios are commonly used to perform HTTP requests, each with its strengths.

### Fetch

Fetch is a native browser API for making HTTP requests.

```ts
fetch('/api/users')
  .then((res) => res.json())
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

#### Pros:

- Native and requires no additional libraries.
- Universally supported.

#### Cons:

- Requires manual handling of JSON and errors.
- Slightly less ergonomic than Axios.

### Axios

Axios is a popular, promise-based HTTP client.

```ts
import axios from 'axios';

axios
  .get('/api/users')
  .then((res) => console.log(res.data))
  .catch((err) => console.error(err));
```

#### Pros:

- Simple syntax.
- Automatic JSON handling.
- Built-in interceptors for global error handling.

#### Cons:

- External library increases bundle size.

## Handling 401 and 403 with Axios

Axios interceptors allow easy global handling of common HTTP errors like unauthorized (401) or forbidden (403).

### Example:

```ts
import axios from 'axios';

const api = axios.create({ baseURL: '/api' });

api.interceptors.response.use(
  (res) => res,
  (error) => {
    if (error.response?.status === 401) {
      window.location.href = '/login';
    }

    if (error.response?.status === 403) {
      alert('Access denied.');
    }

    return Promise.reject(error);
  }
);

export default api;
```

## Using Axios with RTK Query

To use Axios with RTK Query, customize the `baseQuery`:

```ts
import { createApi } from '@reduxjs/toolkit/query/react';
import axios from 'axios';

const axiosBaseQuery =
  ({ baseUrl }) =>
  async ({ url, method, data }) => {
    try {
      const result = await axios({ url: baseUrl + url, method, data });
      return { data: result.data };
    } catch (error) {
      return { error: error.response?.data || error.message };
    }
  };

export const api = createApi({
  reducerPath: 'api',
  baseQuery: axiosBaseQuery({ baseUrl: '/api' }),
  endpoints: (builder) => ({
    getUsers: builder.query({ query: () => ({ url: 'users', method: 'GET' }) }),
  }),
});

export const { useGetUsersQuery } = api;
```

## Using Axios with React Query

Using Axios with React Query is straightforward:

```tsx
import { useQuery } from '@tanstack/react-query';
import axios from 'axios';

function UsersList() {
  const { data, isLoading, isError } = useQuery({
    queryKey: ['users'],
    queryFn: () => axios.get('/api/users').then((res) => res.data),
  });

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Error fetching data</p>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Conclusion

- **Fetch:** Best for minimal dependencies.
- **Axios:** Recommended for robust features like interceptors.
- **React Query:** Ideal for straightforward caching and fetching.
- **RTK Query:** Great when integrating with Redux Toolkit and managing global data cache.
