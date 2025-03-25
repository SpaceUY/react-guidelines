---
title: Data fetching
layout: default
nav_order: 5
---

# Data fetching

**Technology Review**  
_Written by: Federico Castañares, Facundo Panizza_

## Article description
I will try to cover all ways with pro and crons. I will start with the basic implementation to the most usefull tool

### Fetch
This example demonstrates fetching data from a public API and displaying it in a list.

```jsx
import React, { useState, useEffect } from 'react';

const DataFetchingComponent = () => {
  const [data, setData] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [isError, setIsError] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts');
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const data = await response.json();
        setData(data);
      } catch (error) {
        setIsError(true);
      } finally {
        setIsLoading(false);
      }
    };

    fetchData();
  }, []);

  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (isError) {
    return <p>Error fetching data</p>;
  }

  return (
    <div>
      <h1>Data List</h1>
      <ul>
        {data.map((item) => (
          <li key={item.id}>{item.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default DataFetchingComponent;
```

#### Explanation:
• State Management: We use useState to manage the data, loading, and error states.
• Data Fetching: The fetchData function is defined inside the useEffect hook to fetch data when the component mounts.
• Error Handling: If the fetch request fails, an error is caught, and the error state is set to true.
• Loading State: While the data is being fetched, a loading message is displayed.
• Rendering Data: Once the data is fetched successfully, it is displayed in a list.

### Axios -> where fetch have superpowers

The most important thing of axios are:
• Interceptors: Interceptor is the way for call always the same query builder. For example when you call to your API and you need add a token in the headers or when the response is unauthorized problaby you want redirect to login url.
You can create your interceptor like this:

```jsx
import { getLocalStorage } from '@/utils/localstorage';
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';

export const baseURL = import.meta.env.VITE_BACK_URL ?? 'http://localhost:4000';

export const api: AxiosInstance = axios.create({
  baseURL,
  withCredentials: false,
});

api.interceptors.request.use((config) => {
  const token = getLocalStorage('token');
  config.headers['Authorization'] = `Bearer ${token}`;

  return config;
});

api.interceptors.response.use(
  (response) => response,
  async (error: { response: AxiosResponse; config: AxiosRequestConfig }) => {
    if (error.response?.status === 401) {
      document.location.href = '/login';
    }

    return Promise.reject(error);
  },
);
```


## React-query & Modern redux RTK
### Modern Redux, RTK, Redux Query

**Pros:**

- Avoids a lot of unnecessary boilerplate.
- Has a fairly easy-to-use cache management, as well as cache invalidation.
- Supports error handling and loading states.

**Cons:**

- Not necessary, but it is recommended to know the conventional way of Redux for a better understanding of how it works.

#### Example:

Query

```
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: [Users],
  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => 'users',
      providesTags: ['Users'],
    }),
    createUser: build.mutation({
      query: (body) => ({
        url: 'users',
        method: 'POST',
        body,
      }),
      invalidatesTags: ['Users'],
    })
  }),
});

export const { useGetUsersQuery } = api;
```

<br />
This will create a cache for the query “getUsers” and when a new user is created is going to refresh that cache.
<br />
<br />

```
import { useGetUsersQuery } from './api';

const UsersList = () => {
  const { data, isLoading, isError } = useGetUsersQuery();

  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (isError) {
    return <p>Error fetching data</p>;
  }

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

### Conventional Redux

#### Create async thunk

The idea of Redux is to centralize the state of our application in our store and separate our state from our components, providing persistence in the data.

As mentioned earlier, the implementation of Modern Redux requires less code and comes with built-in methods that are often necessary to write if we don't have them.

Nevertheless, let's take a look at the flow.

**Pros**:

- Separation of concerns; on one hand, we have the actions, on the other hand, the services, and finally, how the combination of these two changes the state of our application.

**Cons**:

- Requires writing a considerable amount of boilerplate.
- Initially, communication between each responsibility may seem confusing.

### The Flow

Dispatch in our components to an action:
dispatch(createCashierAction(CashierLoad));

### Action

We create our action that will call the service, then from our slice, we will listen to the response whether it fails or not. If it fails, we have the ability to send the error message to the slice. In the example below, we return the error message with the error response structure of a service with NestJS. We can use this or use our own messages or use just one generic.
