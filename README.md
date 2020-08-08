# Nuxt Routing System (a study case)

> for more info, checkout [nuxt page](https://nuxtjs.org/guides/features/file-system-routing)

## Disclaimer

In order to fully understand (at least some use cases) the routing system applied for [Nuxt.js](https://nuxtjs.org/), I felt the necessity to create this repository to store the tests I've made so far.

## Tests

### Normal route

Creating a single, normal named file, under the `pages` directory, will be structured as a single, normal route:

```
-| pages
--| index.vue

// in routes.js
routes: [
  {
    name: 'index',
    path: '/',
    component: 'pages/index.vue'
  },
]
```

### Normal + Dynamic route

Creating a normal and a dynamic file, under a directory, inside `pages`, will create a single route (for each file), unested.

The dynamic file, will generate a dynamic route with a param of the same name given to the file:

```
-| pages
--| blog
----| index.vue
----| _slug.vue

//-> Because of this structure, the `_slug` file will generate route with a required param named `:slug`(keep that in mind!).

// in routes.js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'blog',
      path: '/blog',
      component: 'pages/blog/index.vue'
    },
    {
      name: 'blog-slug',
      path: '/blog/:slug', <-- required params
      component: 'pages/blog/index.vue'
    },
  ]
}
```

### Optional dynamic route

In order to create an optional dynamic route, it's necessary to put the dynamic route file inside a directory withou the `index.vue` file.

> optional params route means that the route will render even if you don't pass the `params` - like `:id`.

```
-| pages
--| users
----| _id.vue

//-> if you have one `index.vue` inside the directory, the dynamic route will be required.

// in routes.js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'users-id',
      path: '/users/:id?', <-- optional params
      component: 'pages/users/_id.vue'
    },
  ]
}
```

### Required dynamic route

To make a dynamic route required, you can add an `index.vue` file inside the directory you want (like in the _Normal + Dynamic route_ example), but the `index.vue` file would have to be implemented, since it'll become a valid route.

The other option would be turning the dynamic route file into a directory and adding an `index.vue` inside it:

```
-| pages
--| users
----| _id
------| index.vue

// in routes.js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'users-id',
      path: '/users/:id', <-- required params
      component: 'pages/users/_id.vue'
    },
  ]
}
```

### Nested route

To create a nested route, we need to create a file and a directory (with `index.vue` and other files inside) with the **same name** and add a `nuxt-child` in the file, in order to call the nested routes:

```
-| pages
--| products.vue
--| products
----| index.vue
----| _id.vue

//-> products.vue

<template>
  <div>
    // some code
    <nuxt-child></nuxt-child>
  </div>
</template>

// in routes.js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      path: '/products',
      component: 'pages/products.vue',
      children: [{
        path: '',
        component: 'pages/products/index.vue',
        name: 'products'
      }, {
        path: ':id',
        component: 'pages/products/_id.vue',
        name: 'products-id'
      }]
    }
  ]
}
```

### Unknown dynamic route

> For more [info](https://nuxtjs.org/guides/features/file-system-routing#unknown-dynamic-nested-routes)

Sometimes, you don't know how deep your route will be or you want to avoid nesting a lot of directories, making a pain to reach there (Windows will be craaazy).

In those cases, nuxt has a wildcard that can be used to represent unknown dynamic routes (`_.vue`):

```
-| pages
--| _.vue

// in routes.js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      path: "/*",
      component: 'pages/_.vue',
      name: "all"
    }
  ]
}
```

So every route that is not being treated by any file or directory inside `pages` will be handled by this page. For example:

_example from [nuxt site](https://nuxtjs.org/guides/features/file-system-routing#unknown-dynamic-nested-routes)_

```
/about -> _.vue
/about/careers -> _.vue
/about/careers/chicago -> _.vue
```

This bring some issues that we must solve inside the file in order to validate and handle `404 errors`.

In order to show this, imagine you have a program in which you must show a lesson from a specific course and you must send the id from both the course and the lesson.

One way to solve this is to pass the id via the route, but this would enter the problem of big nested directories like:

```
-| pages
--| course
----| _courseCode
------| index.vue
------| lesson
--------| _lessonCode.vue

// imagine this going deeper and deeper ...
```

Applying the wildcard in this situation would result on the following:

```
-| pages
--| course
----| _courseCode
------| index.vue
------| _.vue

// in routes.js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      path: '/course/:courseCode',
      component: 'pages/course/_courseCode/index.vue',
      name: 'course-courseCode'
    },
    {
      path: '/course/:courseCode?/*',
      component: 'pages/course/_courseCode/_.vue',
      name: course-courseCode-all
    }
  ]
}
```

And in the `_.vue`, we use the [validate](https://nuxtjs.org/api/pages-validate) to handle the url:

```js
<script>
export default {
  validate({ params }) {
    const [, lessonCode] = params.pathMatch.split('/');
    return params.courseCode && /^\d+$/g.test(lessonCode);
  },
};
</script>
```

This would solve the issue with deep nested structures, but it's situational. If you happen to have a page for every dynamic route inside the nest, so this wouldn't be possible to use.

**obs.:** One other way to solve this problem is using route queries: `course/1/lesson?lessonCode=1`, and instead of the `_.vue` file, you would use a `lesson.vue` and catch the query inside the file. Could even use the validate method to avoid render the page if no query is passed.

## Enviroment

To check this test yourself, clone this repo and do the following:

```
yarn install // to install the dependecies (only nuxt)

yarn dev // to run the development server
```

Then go to `.nuxt/router.js` and inside the `routes` array, you'll see the routes generated there.
