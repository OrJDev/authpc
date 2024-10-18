# Input

```ts
import { z } from 'zod'
import { createCaller, response$ } from '@solid-mediakit/authpc'

export const withMws = createCaller
  .use(async () => {
    return {
      uno: 1,
    }
  })
  .use((input) => {
    return {
      ...input.ctx$,
      test: 1,
    }
  })

export const other = withMws.use(({ ctx$ }) => {
  return {
    ...ctx$,
    lll: 1,
  }
})

const qfn = other.use(({ ctx$ }) => {
  return {
    ...ctx$,
    test3: 1,
  }
})

export const q = qfn(
  z.object({ why: z.string() }),
  (r) => {
    return `hey ${r.input$.why}`
  },
  {
    key: 'custom-q',
    protected: true,
  },
)

export const getRequest = createCaller(
  z.object({ test: z.string() }),
  ({ input$ }) => {
    console.log('here', input$)
    return response$(
      { iSetTheHeader: true, testing: input$ },
      { headers: { 'cache-control': 'max-age=60' } },
    )
  },
  {
    method: 'GET',
  },
)

export const mutationTest3 = qfn(
  z.object({ ok: z.number(), test: z.object({ l: z.string() }) }),
  ({ input$ }) => {
    return `${input$.ok}`
  },
  {
    type: 'action',
  },
)
```

# Output

```ts
import { GET } from '@solidjs/start'
import { z } from 'zod'
import {
  createCaller,
  response$,
  callMiddleware$,
  validateZod,
} from '@solid-mediakit/authpc'

export const withMws = createCaller
export const other = withMws

const qfn = other

export const q = createCaller(
  async ({ input$: _$$payload }) => {
    'use server'
    const _$$validatedZod = await validateZod(
      _$$payload,
      z.object({
        why: z.string(),
      }),
    )
    if (_$$validatedZod instanceof Response) return _$$validatedZod
    const ctx$ = await callMiddleware$(_$$event, _$$qfn_mws)
    if (ctx$ instanceof Response) return ctx$
    return `hey ${_$$validatedZod.why}`
  },
  {
    protected: true,
    key: 'custom-q',
    method: 'POST',
    type: 'query',
  },
)

export const getRequest = createCaller(
  GET(async (_$$payload) => {
    'use server'
    const _$$validatedZod = await validateZod(
      _$$payload,
      z.object({
        test: z.string(),
      }),
    )
    if (_$$validatedZod instanceof Response) return _$$validatedZod
    console.log('here', _$$validatedZod)
    return response$(
      {
        iSetTheHeader: true,
        testing: _$$validatedZod,
      },
      {
        headers: {
          'cache-control': 'max-age=60',
        },
      },
    )
  }),
  {
    protected: false,
    key: 'getRequest',
    method: 'GET',
    type: 'query',
  },
)

export const mutationTest3 = createCaller(
  async ({ input$: _$$payload }) => {
    'use server'
    const _$$validatedZod = await validateZod(
      _$$payload,
      z.object({
        ok: z.number(),
        test: z.object({
          l: z.string(),
        }),
      }),
    )
    if (_$$validatedZod instanceof Response) return _$$validatedZod
    const ctx$ = await callMiddleware$(_$$event, _$$qfn_mws)
    if (ctx$ instanceof Response) return ctx$
    return `${_$$validatedZod.ok}`
  },
  {
    protected: false,
    key: 'mutationTest3',
    method: 'POST',
    type: 'action',
  },
)

export const _$$withMws_mws = [
  async () => {
    return {
      uno: 1,
    }
  },
  (input) => {
    return {
      ...input.ctx$,
      test: 1,
    }
  },
]

export const _$$other_mws = [
  ..._$$withMws_mws,
  ({ ctx$ }) => {
    return {
      ...ctx$,
      lll: 1,
    }
  },
]

export const _$$qfn_mws = [
  ..._$$other_mws,
  ({ ctx$ }) => {
    return {
      ...ctx$,
      test3: 1,
    }
  },
]
```
