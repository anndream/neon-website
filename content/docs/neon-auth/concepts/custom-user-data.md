---
title: Custom User Data
subtitle: How to store custom user metadata in Neon Auth
enableTableOfContents: true
tag: beta
---

> How to store custom user metadata in Neon Auth

Neon Auth allows storing additional user information through three types of metadata fields:

1. **clientMetadata**: Readable and writable from a [client](/docs/neon-auth/concepts/stack-app#client-vs-server).
2. **serverMetadata**: Readable and writable only from a [server](/docs/neon-auth/concepts/stack-app#client-vs-server).
3. **clientReadOnlyMetadata**: Readable from a client, writable only from a server.

## Client metadata

You can use the `clientMetadata` field to store non-sensitive information that both the client and server can read and write.

```tsx shouldWrap
await user.update({
  clientMetadata: {
    mailingAddress: '123 Main St',
  },
});

// On the client:
const user = useUser();
console.log(user.clientMetadata);
```

## Server-side metadata

For sensitive information, use the `serverMetadata` field. This ensures the data is only accessible and modifiable by the server.

```tsx shouldWrap
const user = await stackServerApp.getUser();
await user.update({
  serverMetadata: {
    secretInfo: 'This is a secret',
  },
});

// To read:
const user = await stackServerApp.getUser();
console.log(user.serverMetadata);
```

## Client read-only metadata

Use `clientReadOnlyMetadata` for data that clients need to read but never modify, such as subscription status.

```tsx shouldWrap
// On the server:
const user = await stackServerApp.getUser();
await user.update({
  clientReadOnlyMetadata: {
    subscriptionPlan: 'premium',
  },
});

// On the client:
const user = useUser();
console.log(user.clientReadOnlyMetadata);
```
