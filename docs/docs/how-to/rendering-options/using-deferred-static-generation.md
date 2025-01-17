---
title: Using Deferred Static Generation (DSG)
---

## Introduction

Deferred Static Generation (DSG) is one of [Gatsby's rendering options](/docs/conceptual/rendering-options/) and allows you to defer non-critical page generation to user request, speeding up build times. Instead of generating _every_ page at build time, you can decide to build certain pages up front and others only when a user accesses the page at run time. For large sites, with content that is infrequently visited (e.g. old blog posts or certain content types), this can dramatically reduce build times.

In this guide, you'll learn how to modify your `createPage` calls to only build your preferred pages up-front and leave the rest deferred to the first user request.

For full documentation on all options, see the [Reference Guide on Deferred Static Generation](/docs/reference/rendering-options/deferred-static-generation/).

## Prerequisites

Before you begin, you should already have:

- An existing Gatsby site. (Need help creating one? Follow the [Quick Start](/docs/quick-start/).)
- A `gatsby-node.js` file where you're creating pages with the [`createPages`](/docs/reference/config-files/gatsby-node#createPages) API. (The [File System Route API](/docs/reference/routing/file-system-route-api) isn't yet supported, but it will be soon!)

## Directions

The general process for using DSG looks like this:

- Adding `defer: true` to your `createPage` call.

  ```js
  createPage({
    path: "page-path",
    component: "component-path",
    context: {},
    defer: true, // highlight-line
  })
  ```

For the purpose of this guide, let's assume you have a blog powered by MDX and have blog posts dating back years (with 1000 blog posts in total). Via your analytics tracking, you're seeing that only the latest 100 posts are regularly visited. The rest gets occasional or no visits at all.

The `gatsby-node.js` file:

```js:title=gatsby-node.js
const blogPostTemplate = require.resolve(`./src/templates/blog-post.js`)

export.createPages = async ({ graphql, actions, reporter }) => {
  const { createPage } = actions

  const result = await graphql(`
    query {
      allMdx(sort: { fields: frontmatter___date, order: DESC }) {
        nodes {
          slug
        }
      }
    }
  `)

  if (result.errors) {
    reporter.panicOnBuild(`There was an error loading posts`, result.errors)
    return
  }

  const posts = result.data.allMdx.nodes

  posts.forEach(post => {
    createPage({
      path: post.slug,
      component: blogPostTemplate,
      context: {
        slug: post.slug,
      },
    })
  })
}
```

For this example, it's important that inside the `gatsby-node.js` the result is sorted by date and in an descending order. Depending on your use case you might need to adjust the query to sort differently or query additional information (e.g. a "category" field from the frontmatter) besides the `slug`.

This way you can use the `index` in the `forEach` to apply `defer` to all but the latest 100 posts:

```js:title=gatsby-node.js
// Rest of gatsby-node.js

const posts = result.data.allMdx.nodes

posts.forEach((post, index) => {
  createPage({
    path: post.slug,
    component: blogPostTemplate,
    context: {
      slug: post.slug,
    },
    // index is zero-based index
    defer: index + 1 > 100, // highlight-line
  })
})
```

The first 100 pages will receive `defer: false`, the other 900 pages receive `defer: true`.

## Additional Resources

- [API Reference guide](/docs/reference/rendering-options/deferred-static-generation/)
- [Conceptual Guide](/docs/conceptual/rendering-options/)
