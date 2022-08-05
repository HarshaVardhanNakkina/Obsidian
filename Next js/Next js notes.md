# Navigate between pages
## Pages in Next.js

In Next.js, a page is a React Component exported from a file in the [`pages` directory](https://nextjs.org/docs/basic-features/pages).

Pages are associated with a route based on their **file name**. For example, in development:

-   `pages/index.js` is associated with the `/` route.
-   `pages/posts/first-post.js` is associated with the `/posts/first-post` route.

Simply create a JS file under the [`pages` directory](https://nextjs.org/docs/basic-features/pages), and the path to the file becomes the URL path.

In a way, this is similar to building websites using HTML or PHP files. Instead of writing HTML you write JSX and use React Components.

**Quick Review**: If you wanted to add a new route `/authors/me`, what would the file name be (including the directory)? ***Ans:*** `pages/authors/me.js`

## Link Component
When linking between pages on websites, you use the `<a>` HTML tag.

In Next.js, you use the `Link` Component from [`next/link`](https://nextjs.org/docs/api-reference/next/link) to wrap the `<a>` tag. `<Link>` allows you to do client-side navigation to a different page in the application.

### Using `<Link>`

First, open `pages/index.js`, and import the `Link` component from [`next/link`](https://nextjs.org/docs/api-reference/next/link) by adding this line at the top:

```javascript
import Link from 'next/link'
```

Then find the `h1` tag that looks like this:

```jsx
<h1 className="title">
	Learn
```

As you can see, the `Link` component is similar to using `<a>` tags, but instead of `<a href="…">`, you use `<Link href="…">` and put an `<a>` tag inside.