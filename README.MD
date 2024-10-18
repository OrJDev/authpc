# @solid-mediakit/authpc

Type-safe utility library for Solid

## createCaller

Use this method to interact with the api, you can choose wether to use a `query`` or a `mutation `and also choose wether to use`GET``or`POST`.

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

// get request
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
```
