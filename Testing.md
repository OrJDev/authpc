## What Is AuthPC

AuthPC is a utility library that combines both 
- Authentication (`Auth`)
- Better Solid'S RPC (`PC`)

AuthPC supports two Auth Providers: Clerk & AuthJS
You can change the entire behavior of the app with single line of change (changes both the types & functionality)

AuthPC also allows you to throw type-safe errors, redirect the user, modify headers, set cookies and most importantly, you can choose to use either `GET` (allowing the use of HTTP cache-control headers) or `POST`.

That means you can create server actions completly type safe, cached, auth/session protected and all in simple function declaration, many might think but wait this is very simple to achieve in other frameworks, what took you so long to create this one, In this doc i'm actually going to explain the behind the scenes of `AuthPC`.

## How Is It Being Used

First, lets understand what does the `createCaller` method accepts

- createCaller(schema,fn,opts?)
- createCaller(fn,opts?)

### fn

while `fn` is something like:

```ts
({ input, event$, ctx$, session }) => {
    console.log('user-agent', event$.request.headers.get('user-agent'))
    console.log('Welcome', session.user?.name)
    return `hey ${input.test}`
  }
```

- input: is sent from the client side and then validated using provided schema.
- event$: is returned from `getRequestEvent()` and transformed in plugin.
- ctx$: is returned from a middleware which you can attach using the `.use` method on createCaller.
- session: is returned from `getSession()` and transformed in plugin.


A Full example would be:

```ts
import { z } from 'zod'
import { createCaller } from '@solid-mediakit/authpc'

export const r = createCaller(
  z.object({
    test: z.string(),
  }),
  ({ input, event$, session }) => {
    console.log('user-agent', event$.request.headers.get('user-agent'))
    console.log('Welcome', session.user.name)
    return `hey ${input.test}`
  },
  {
    protected: true,
    method: "POST"
  },
)
```

## What makes it so complex?

well to make something a in Solid run on server we use the servr pragma:

```ts
'use server';
```

And then Solid/Vinxi using a babel plugin to transform this 'use server' to an actual server RPC call.

so if we want to create a server action utility library, we can't just do:

```ts
const fnToServer = <Fn extends (...args: any[]) => any>(fn: Fn) => {
  return (...args: Parameters<Fn>) => {
    'use server'
    return fn(...args)
  }
}
```

That is for two reasons:

1. Solid compiles your code using a babel plugin, it parses the files in your projects and transform them accordinigly **exactly as they are**.
2. Closures aren't allowed from client to server, meaning you cannot pass a function from your client file to this function and expect it to become server function.

```ts
import { fnToServer } from './test'

const Home = () => {
  const serverFn = fnToServer(() => {
    console.log('on server')
  })
  return <button onClick={() => serverFn()}>call</button>
}

export default Home
```

This infact will not work.

## Solution

Well since you cannot use existing functions to transform a client function to server, how does AuthPC still support it? AuthPC also uses a babel plugin to transform.

A simple transformation will be:

```ts
const fn = createCaller(()=>{
    console.log("run on server")
})
```

This using a babel plugin becomes:

```ts
const fn = createCaller(()=>{
    'use server';
    console.log("run on server")
})
```

how does it actually look like?

```ts
import { z } from 'zod'
import { createCaller } from '@solid-mediakit/authpc'

const withCtx = createCaller.use(() => {
  const num = 1
  return {
    num,
  }
})

export const callTest = withCtx(
  z.object({
    test: z.string(),
  }),
  ({ input$, event$, session$, ctx$ }) => {
    console.log('user-agent', event$.request.headers.get('user-agent'))
    console.log('Welcome', session$.user.name)
    console.log('ctx', ctx$)
    return `hey ${input$.test}`
  },
  {
    protected: true,
  },
)
```

Becomes:

```ts
import { authOpts } from '~/server/auth'
import { getSession } from '@solid-mediakit/auth'
import { getRequestEvent } from 'solid-js/web'
import { cache } from '@solidjs/router'
import { z } from 'zod'
import {
  createCaller,
  callMiddleware$,
  error$,
  validateZod,
} from '@solid-mediakit/authpc'

const withCtx = createCaller

export const callTest = createCaller(
  cache(async ({ input$: _$$payload }) => {
    'use server'
    const _$$validatedZod = await validateZod(
      _$$payload,
      z.object({
        test: z.string(),
      }),
    )
    if (_$$validatedZod instanceof Response) return _$$validatedZod
    const _$$event = getRequestEvent()
    const session$ = await getSession(_$$event.request, authOpts)
    if (!session$) {
      return error$('This is a protected route')
    }
    const ctx$ = await callMiddleware$(_$$event, _$$withCtx_mws)
    if (ctx$ instanceof Response) return ctx$
    console.log('user-agent', _$$event.request.headers.get('user-agent'))
    console.log('Welcome', session$.user.name)
    console.log('ctx', ctx$)
    return `hey ${_$$validatedZod.test}`
  }, 'callTest'),
  {
    protected: true,
    key: 'callTest',
  },
)

export const _$$withCtx_mws = [
  () => {
    const num = 1
    return {
      num,
    }
  },
]

```

so `input` calls becomes `_$$payload` or `_$$validatedZod`
`event$` calls becomes `_$$event` using getRequestEvent
`ctx` becomes `ctx$` using callMiddleware$
