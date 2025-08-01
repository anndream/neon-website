---
title: Custom Pages
subtitle: How to build custom authentication pages with Neon Auth
enableTableOfContents: true
---

Custom pages allow you to take full control over the layout and logic flow of authentication pages in your application. Instead of using the default pages provided by Neon Auth, you can build your own using our built-in components or low-level functions.

By default, `StackHandler` creates all authentication pages you need, however, you can replace them with your own custom implementations for a more tailored user experience.

<Steps>
## Simple Example

For example, if you want to create a custom sign-in page with a customized title on the top, you can create a file at `app/signin/page.tsx`:

```tsx title="app/signin/page.tsx"
import { SignIn } from '@stackframe/stack';

export default function CustomSignInPage() {
  return (
    <div>
      <h1>My Custom Sign In page</h1>
      <SignIn />
    </div>
  );
}
```

Then you can instruct the Stack app in `stack.ts` to use your custom sign in page:

```tsx title="stack.ts"
export const stackServerApp = new StackServerApp({
  // ...
  // add these three lines
  urls: {
    signIn: '/signin',
  },
});
```

You are now all set! If you visit the `/signin` page, you should see your custom sign in page. When users attempt to access a protected page or navigate to the default `/handler/sign-in` URL, they will automatically be redirected to your new custom sign-in page.

## Building From Scratch

While the simple approach above lets you customize the layout while using Stack's pre-built components, sometimes you need complete control over both the UI and authentication logic.

We also provide the low-level functions powering our components, so that you can build your own logic. For example, to build a custom OAuth sign-in button, create a file at `app/signin/page.tsx`:

```tsx title="app/signin/page.tsx"
'use client';
import { useStackApp } from '@stackframe/stack';

export default function CustomOAuthSignIn() {
  const app = useStackApp();

  return (
    <div>
      <h1>My Custom Sign In page</h1>
      <button
        onClick={async () => {
          // This will redirect to the OAuth provider's login page.
          await app.signInWithOAuth('google');
        }}
      >
        Sign In with Google
      </button>
    </div>
  );
}
```

Again, edit the Stack app in `stack.ts` to use your custom sign in page:

```tsx title="stack.ts"
export const stackServerApp = new StackServerApp({
  // ...
  // add these three lines
  urls: {
    signIn: '/signin',
  },
});
```

As above, visit the `/signin` page to see your newly created custom OAuth page.
</Steps>
