---
title: Data fetching
layout: default
nav_order: 5
---

## Data fetching

**Technology Review**  
_Written by: Federico Castañares, Facundo Panizza_

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
