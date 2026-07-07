# vaul-vue SSR crash repro

Minimal Nuxt 4 app using `vaul-vue@latest` (0.4.1 from npm, unmodified) that
reproduces an SSR crash: `document is not defined`.

## Run it

```bash
npm install
npm run build
node .output/server/index.mjs
curl -i http://localhost:3000/
```

Expect `HTTP/1.1 500` with body:

```json
{
  "error": true,
  "statusCode": 500,
  "message": "document is not defined"
}
```

## Why it crashes

`app/app.vue` renders a drawer with `<DrawerRoot :default-open="true">` and
`<DrawerPortal force-mount>`. `force-mount` is required to make the drawer's
content actually render during SSR — by default reka-ui's `TeleportPrimitive`
guards the whole portal behind `v-if="isMounted || forceMount"`, so with no
`forceMount` the drawer subtree (and the bug) is silently skipped server-side.

Once the content does render on the server, `DrawerContent`'s `setup()`
unconditionally calls `useScaleBackground()`, which reads
`document.body.style.backgroundColor` synchronously — not inside
`onMounted`/`watchEffect` — so it throws on the server where `document`
doesn't exist:

```
document is not defined
  at bt (node_modules/vaul-vue/dist/index.js:750:137)   <- useScaleBackground
  at setup (node_modules/vaul-vue/dist/index.js:806:5)  <- DrawerContent
  ...
  at renderComponentVNode (@vue/server-renderer)
```

See `../reka-ssr-test` for the same example built on reka-ui's native
`Drawer` primitive, which renders cleanly under identical conditions.
