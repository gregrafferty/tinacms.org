---
title: Creating Remark Forms
---

There are three ways to register remark forms with the CMS, depending on the component:

- [`useRemarkForm`](/guides/gatsby/git/create-remark-form#1-the-hook-useremarkform) - A [React Hook](https://reactjs.org/docs/hooks-intro.html) — useful for any function component. You can use this with components that source data from a [static query](https://www.gatsbyjs.org/docs/static-query/#how-staticquery-differs-from-page-query) using Gatsby's `useStaticQuery` hook.
- [`RemarkForm`](/guides/gatsby/git/create-remark-form#2-the-render-props-component-remarkform) — A [Render Props](https://reactjs.org/docs/render-props.html#use-render-props-for-cross-cutting-concerns) component — useful for any class component. Can be used with components sourcing data from a [static query](https://www.gatsbyjs.org/docs/static-query/#how-staticquery-differs-from-page-query) using Gatsby's [`StaticQuery`](https://www.gatsbyjs.org/docs/static-query/) render props component.
- [`remarkForm`](/guides/gatsby/git/create-remark-form#3-the-higher-order-component-remarkform) — A [Higher-Order Component](https://reactjs.org/docs/higher-order-components.html) — useful for [page components](https://www.gatsbyjs.org/docs/recipes/#creating-pages-automatically) (function or class), that source data from a [page query](https://www.gatsbyjs.org/docs/page-query/).

All of these options can only take data (transformed by `gatsby-transformer-remark`) from a `markdownRemark` query. If you need more information on using Markdown in Gatsby, refer to [this documentation](https://www.gatsbyjs.org/docs/adding-markdown-pages/).

### 1. The Hook: useRemarkForm

This hook connects the `markdownRemark` data with Tina to be made editable. It is useful in situations where you need to edit on non-page components, or just prefer working with hooks or static queries. You can also use this hook with functional page components.

#### Usage:

`useRemarkForm(remark, options): [values, form]`

#### Arguments:

- `remark`: The data returned from a Gatsby `markdownRemark` query.
- `options`: A configuration object that can include [form options](https://tinacms.org/guides/gatsby/git/customize-form) or form actions (such as the [`DeleteAction`](https://tinacms.org/guides/gatsby/creating-new-files#deleting-files))— optional.

#### Return:

- `[values, form]`
  - `values`: The current values to render in the template. This has the same shape as the `markdownRemark` data.
  - `form`: A reference to the `Form`. Most of the time you won't need to directly work with the `Form`.

```javascript
/*
 ** example component --> src/components/Title.js
 */

// 1. import useRemarkForm and usePlugin
import { useRemarkForm } from 'gatsby-tinacms-remark'
import { usePlugin } from 'tinacms'
import { useStaticQuery } from 'gatsby'

const Title = data => {
  // 2. Add required GraphQL fragment
  const data = useStaticQuery(graphql`
    query TitleQuery {
      markdownRemark(fields: { slug: { eq: "song-of-myself" } }) {
        ...TinaRemark
        frontmatter {
          title
        }
      }
    }
  `)

  // 3. Call the hook and pass in the data
  const [markdownRemark, form] = useRemarkForm(data.markdownRemark)

  // 4. Register the form plugin
  usePlugin(form)

  return <h1>{markdownRemark.frontmatter.title}</h1>
}

export default Title
```

To use this hook, you'll first need to import it from `gatsby-tinacms-remark`. Then you'll need to add the GraphQL fragment `...TinaRemark` to your query. The fragment adds these parameters: `id`, `fileRelativePath`, `rawFrontmatter`, and `rawMarkdownBody`. Finally you'll call the hook and pass in the `markdownRemark` data.

The form will populate with default text fields. To customize it, you can pass in a config options object as the second parameter. Jump ahead to learn more on [customizing the form](http://tinacms.org/guides/gatsby/git/customize-form).

### 2. The Render Props Component: RemarkForm

`RemarkForm` is a thin wrapper around `useRemarkForm` and `usePlugin`. Since React [Hooks](https://reactjs.org/docs/hooks-intro.html) are only available within [function components](https://reactjs.org/docs/components-and-props.html) you will need to use `RemarkForm` instead of calling those hooks directly working with a [class component](https://reactjs.org/docs/components-and-props.html).

#### Props:

- `remark`: the data returned from a Gatsby `markdownRemark` query.
- `render(renderProps): JSX.Element`: A function that returns JSX elements
  - `renderProps.markdownRemark`: The current values to be displayed. This has the same shape as the `markdownRemark` data that was passed in.
  - `renderProps.form`: A reference to the [`Form`](http://localhost:8000/docs/forms/).

You can use this with both page and non-page components in Gatsby. Below is an example of using `RemarkForm` in a non-page component using [`StaticQuery`](https://www.gatsbyjs.org/docs/static-query/).

```javascript
/*
 ** example component --> src/components/Title.js
 */
import { StaticQuery, graphql } from 'gatsby'

// 1. import RemarkFrom
import { RemarkForm } from 'gatsby-tinacms-remark'

class Title extends React.Component {
  render() {
    return (
      <StaticQuery
        // 2. add ...TinaRemark fragment to query
        query={graphql`
          query TitleQuery {
            markdownRemark(fields: { slug: { eq: "song-of-myself" } }) {
              ...TinaRemark
              frontmatter {
                title
              }
            }
          }
        `}
        render={data => (
          /*
           ** 3. Return RemarkForm, pass in the props
           **    and then return the JSX this component
           **    should render
           */
          <RemarkForm
            remark={data.markdownRemark}
            render={({ markdownRemark }) => {
              return <h1>{markdownRemark.frontmatter.title}</h1>
            }}
          />
        )}
      />
    )
  }
}

export default Title
```

Here is another example using `RemarkForm` with a page component:

```js
/*
 ** src/templates/blog-post.js
 */

// 1. import RemarkForm
import { RemarkForm } from '@tinacms/gatsby-tinacms-remark'

class BlogPostTemplate extends React.Component {
  render() {
    /*
     ** 2. Return RemarkForm, pass in markdownRemark
     **    as props and return the jsx this component
     **    should render
     */
    return (
      <RemarkForm
        remark={this.props.data.markdownRemark}
        render={({ markdownRemark }) => {
          return <h1>{markdownRemark.frontmatter.title}</h1>
        }}
      />
    )
  }
}

export default BlogPostTemplate

// 3. Add ...TinaRemark fragment to query
export const pageQuery = graphql`
  query {
    markdownRemark(fields: { slug: { eq: $slug } }) {
      ...TinaRemark
      frontmatter {
        title
      }
    }
  }
`
```

Learn how to customize the fields displayed in the form [below](/guides/gatsby/git/customize-form).

### 3. The Higher-Order Component: remarkForm

The `remarkForm` [higher-order component](https://reactjs.org/docs/higher-order-components.html) (HOC) let's us register forms with `Tina` on Gatsby page components.

There are 3 steps to making a Markdown file editable with `remarkForm`:

1. Import the `remarkForm` HOC
2. Wrap your template with `remarkForm`
3. Add `...TinaRemark` to the GraphQL query

> Required fields used to be queried individually: `id`, `fileRelativePath`, `rawFrontmatter`, & `rawMarkdownBody`. The same fields are now being queried via `...TinaRemark`

**Example: src/templates/blog-post.js**

```jsx
// 1. Import the `remarkForm` HOC
import { remarkForm } from 'gatsby-tinacms-remark'

function BlogPostTemplate(props) {
  return <h1>{props.data.markdownRemark.frontmatter.title}</h1>
}

// 2. Wrap your template with `remarkForm`
export default remarkForm(BlogPostTemplate)

// 3. Add the required fields to the GraphQL query
export const pageQuery = graphql`
  query BlogPostBySlug($slug: String!) {
    markdownRemark(fields: { slug: { eq: $slug } }) {
      id
      html
      frontmatter {
        title
        date
        description
      }
      ...TinaRemark
    }
  }
`
```

You should now see text inputs for each of your front matter fields and for the Markdown body. Try changing the title and see what happens!

### Queries aliasing 'markdownRemark'

NOTE: If your query uses an alias for 'markdownRemark', then you will have to use the 'queryName' option to specify the alias name.

**Example: src/templates/blog-post.js**

```jsx
/// ...

// Use 'queryName' to specify where markdownRemark is found.
export default remarkForm(BlogPostTemplate, { queryName: 'myContent' })

// Aliasing markdownRemark as 'myContent'
export const pageQuery = graphql`
  query BlogPostBySlug($slug: String!) {
    myContent: markdownRemark(fields: { slug: { eq: $slug } }) {
      // ...
    }
  }
`
```
