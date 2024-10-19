> uncompleted docs, demo purposes

- check out [how it works](./Testing.md)
- check out [transform](./transform.md)

# @solid-mediakit/authpc

Type-safe utility library for Solid

## createCaller

Use this method to interact with the api, you can choose wether to use a `query` or a `mutation` (default is query) and also choose wether to use `GET`/`POST` (default is POST).

```ts
import { createCaller, response$ } from '@solid-mediakit/authpc'
import { z } from 'zod'

const mySchema = z.object({ name: z.string() })

createCaller(mySchema, ({ input$, session$ }) => {
  console.log('User logged in?', session$)
  return `Hey there ${input$.name}`
})

// protected server function
createCaller(
  mySchema,
  ({ input$, session$ }) => {
    console.log('User logged in!!!', session$)
    return `Hey there ${input$.name}`
  },
  {
    protected: true,
  },
)

// this will be called using GET method
export const getRequest = createCaller(
  () => {
    return response$(
      { iSetTheHeader: true },
      { headers: { 'cache-control': 'max-age=60' } },
    )
  },
  {
    method: 'GET',
  },
)

// this will be called using GET method, and is a mutation
export const mutation = createCaller(
  () => {
    return response$(
      { iSetTheHeader: true },
      { headers: { 'cache-control': 'max-age=60' } },
    )
  },
  {
    method: 'GET',
    type: 'action',
  },
)
mutation.mutate()
```

## Middleware & Merging

You can combine multiple callers / middlewares and import them to different files.

### file1.ts

```ts
import { createCaller } from '@solid-mediakit/authpc'

export const withMw1 = createCaller.use(() => {
  return {
    myFile1: 1,
  }
})

export const action1 = withMw1(({ ctx$ }) => {
  return `hey ${ctx$.myFile1} `
})
```

becomes

```ts
import { createCaller, callMiddleware$ } from '@solid-mediakit/authpc'

export const withMw1 = createCaller

export const action1 = createCaller(
  async ({ input$: _$$payload }) => {
    'use server'
    const ctx$ = await callMiddleware$(_$$event, _$$withMw1_mws)
    if (ctx$ instanceof Response) return ctx$
    return `hey ${ctx$.myFile1} `
  },
  {
    protected: false,
    key: 'action1',
    method: 'POST',
    type: 'query',
  },
)

export const _$$withMw1_mws = [
  () => {
    return {
      myFile1: 1,
    }
  },
]
```

### file2.ts

```ts
import { withMw1 } from './file1'

export const withMw2 = withMw1.use(({ ctx$ }) => {
  return {
    ...ctx$,
    myFile2: 2,
  }
})

export const action2 = withMw2(({ ctx$ }) => {
  return `hey ${ctx$.myFile1} ${ctx$.myFile2}`
})
```

becomes:


```ts
import { createCaller, callMiddleware$ } from '@solid-mediakit/authpc'
import { withMw1, _$$withMw1_mws } from './file1'

export const withMw2 = withMw1

export const action2 = createCaller(
  async ({ input$: _$$payload }) => {
    'use server'
    const ctx$ = await callMiddleware$(_$$event, _$$withMw2_mws)
    if (ctx$ instanceof Response) return ctx$
    return `hey ${ctx$.myFile1} ${ctx$.myFile2}`
  },
  {
    protected: false,
    key: 'action2',
    method: 'POST',
    type: 'query',
  },
)

export const _$$withMw2_mws = [
  ..._$$withMw1_mws,
  ({ ctx$ }) => {
    return {
      ...ctx$,
      myFile2: 2,
    }
  },
]
```

- check out [how it works](./Testing.md)
- check out [transform](./transform.md)
