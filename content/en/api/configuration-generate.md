---
title: "API: The generate Property"
description: Configure the generation of your universal web application to a static web application.
menu: generate
group: Configuration
groupPosition: 5
position: 10
---

- Type: `Object`

> Configure the generation of your universal web application to a static web application.

When launching `nuxt build && nuxt export`(>= v2.13) or `nuxt generate` (<= v2.12) or calling `nuxt.generate()`, Nuxt.js will use the configuration defined in the `generate` property.

nuxt.config.js 
```js
export default {
  generate: {
    ...
  }
}
```

## concurrency

- Type: `Number`
- Default: `500`

The generation of routes are concurrent, `generate.concurrency` specifies the amount of routes that run in one thread.

## crawler
 - Type: `boolean`
 - Default: true

As of Nuxt >= v2.13 Nuxt.js comes with a crawler installed that will crawl your relative links and generate your dynamic links based on these links. If you want to disable this feature you can set the value to `false`

```js
export default {
  generate: {
    crawler: false
  }
}
```

## dir

- Type: `String`
- Default: `'dist'`

Directory name created when building the web application in static mode with `nuxt build && nuxt export`(>= v2.13) or `nuxt generate` (<= v2.12) or in SPA mode with `nuxt build`.

## devtools

- Type: `boolean`
- Default: `false`

Configure whether to allow [vue-devtools](https://github.com/vuejs/vue-devtools) inspection.

If you already activated through nuxt.config.js or otherwise, devtools enable regardless of the flag.

## exclude

- Type: `Array`
  - Items: `String` or `RegExp`

It accepts an array of string or regular expressions and will prevent generation of routes matching them. The routes will still be accessible when `generate.fallback` is used.

Taking this examples of structure:
```bash
-| pages/
---| index.vue
---| admin/
-----| about.vue
-----| index.vue
```

By default, running `nuxt build && nuxt export`(>= v2.13) or `nuxt generate` (<= v2.12) a file will be created for each route.

```bash
-| dist/
---| index.html
---| admin/
-----| about.html
-----| item.html
```

When adding a regular expression which matches all routes with "ignore", it will prevent the generation of these routes.

nuxt.config.js 
```js
export default {
  generate: {
    exclude: [
      /^\/admin/ // path starts with /admin
    ]
  }
}
```

```bash
-| dist/
---| index.html
```

You can also exclude a specific route by giving a string:

```js
export default {
  generate: {
    exclude: ['/my-secret-page']
  }
}
```

## fallback

- Type: `String` or `Boolean`
- Default: `200.html`

```js
export default {
  generate: {
    fallback: '404.html'
  }
}
```

The path to the fallback HTML file. It should be set as the error page, so that also unknown routes are rendered via Nuxt.
If unset or set to a falsy value, the name of the fallback HTML file will be `200.html`. If set to `true`, the filename will be `404.html`. If you provide a string as a value, it will be used instead.

When running a SPA it is more idiomatic to use a `200.html`, as it's the only file necessary as no other routes are generated.

```js
fallback: false
```

If working with statically generated pages then it is recommended to use a `404.html` for error pages and for those covered by [excludes](https://nuxtjs.org/api/configuration-generate#exclude) (the files that you do not want generated as static pages).

```js
fallback: true
```

However, Nuxt allows you to configure any page you like so if you don't want to use the `200.html` or `404.html` you can add a string and then you just have to make sure you redirect to that page instead. This is of course not necessary and is best to redirect to `200.html`/`404.html`.

```js
fallback: 'fallbackPage.html'
```

 *Note: Multiple services (e.g. Netlify) detect a `404.html` automatically. If you configure your webserver on your own, please consult it's documentation to find out how to set up an error page (and set it to the `404.html` file) * 
 
## interval

- Type: `Number`
- Default: `0`

Interval between two render cycles to avoid flooding a potential API with API calls from the web application.

## minify

- **Deprecated!**
- Use [build.html.minify](/api/configuration-build#html-minify) instead

## routes

- Type: `Array`

<div class="Alert Alert-blue">

As of Nuxt v2.13 there is a crawler installed that will now crawl your link tags and generate your routes when using the command `nuxt export` based on those links. 

</div>

<div class="Alert Alert--orange">

**Warning:** If you using Nuxt >= v2.13 and have pages that have no links such as secret pages and you would like these to also be generated then you can use the `generate.routes` property.

**Warning:** dynamic routes are ignored by the `generate` command when using Nuxt <= v2.12 

</div>

Example:

```bash
-| pages/
---| index.vue
---| users/
-----| _id.vue
```

Only the route `/` will be generated by Nuxt.js.

If you want Nuxt.js to generate routes with dynamic params, you need to set the `generate.routes` property to an array of dynamic routes.

We add routes for `/users/:id` in `nuxt.config.js`:

```js
export default {
  generate: {
    routes: [
      '/users/1',
      '/users/2',
      '/users/3'
    ]
  }
}
```

Then when we launch `nuxt generate`:

```bash
[nuxt] Generating...
[...]
nuxt:render Rendering url / +154ms
nuxt:render Rendering url /users/1 +12ms
nuxt:render Rendering url /users/2 +33ms
nuxt:render Rendering url /users/3 +7ms
nuxt:generate Generate file: /index.html +21ms
nuxt:generate Generate file: /users/1/index.html +31ms
nuxt:generate Generate file: /users/2/index.html +15ms
nuxt:generate Generate file: /users/3/index.html +23ms
nuxt:generate HTML Files generated in 7.6s +6ms
[nuxt] Generate done
```

Great, but what if we have **dynamic params**?

1. Use a `Function` which returns a `Promise`.
2. Use a `Function` with a `callback(err, params)`.

### Function which returns a Promise

`nuxt.config.js`

```js
import axios from 'axios'

export default {
  generate: {
    routes () {
      return axios.get('https://my-api/users')
        .then((res) => {
          return res.data.map((user) => {
            return '/users/' + user.id
          })
        })
    }
  }
}
```

### Function with a callback

`nuxt.config.js`

```js
import axios from 'axios'

export default {
  generate: {
    routes (callback) {
      axios.get('https://my-api/users')
        .then((res) => {
          const routes = res.data.map((user) => {
            return '/users/' + user.id
          })
          callback(null, routes)
        })
        .catch(callback)
    }
  }
}
```

### Speeding up dynamic route generation with `payload`

In the example above, we're using the `user.id` from the server to generate the routes but tossing out the rest of the data. Typically, we need to fetch it again from inside the `/users/_id.vue`. While we can do that, we'll probably need to set the `generate.interval` to something like `100` in order not to flood the server with calls. Because this will increase the run time of the generate script, it would be preferable to pass along the entire `user` object to the context in `_id.vue`. We do that by modifying the code above to this:

`nuxt.config.js`

```js
import axios from 'axios'

export default {
  generate: {
    routes () {
      return axios.get('https://my-api/users')
        .then((res) => {
          return res.data.map((user) => {
            return {
              route: '/users/' + user.id,
              payload: user
            }
          })
        })
    }
  }
}
```

Now we can access the `payload` from `/users/_id.vue` like so:

```js
async asyncData ({ params, error, payload }) {
  if (payload) return { user: payload }
  else return { user: await backend.fetchUser(params.id) }
}
```

## subFolders

- Type: `Boolean`
- Default: `true`

By default, running `nuxt build && nuxt export`(>= v2.13) or `nuxt generate`(<= v2.12) Nuxt.js will create a directory for each route & serve an `index.html` file.

Example:

```bash
-| dist/
---| index.html
---| about/
-----| index.html
---| products/
-----| item/
-------| index.html
```

When set to false, HTML files are generated according to the route path:

nuxt.config.js 
```js
export default {
  generate: {
    subFolders: false
  }
}
```

```bash
-| dist/
---| index.html
---| about.html
---| products/
-----| item.html
```


