---
title: "Enhancing SvelteKitAuth with Custom Type Additions"
seoTitle: "Custom Type Enhancements in SvelteKitAuth"
seoDescription: "Guide to enhancing SvelteKitAuth with custom type additions for improved IDE recognition and smoother development"
datePublished: Sun Jul 07 2024 19:48:13 GMT+0000 (Coordinated Universal Time)
cuid: clybyuaiq000508l228gofz3a
slug: enhancing-sveltekitauth-with-custom-type-additions
tags: types, sveltekitauth, authjs

---

Often, your IDE might complain that certain types are unrecognizable. This is common in SvelteKitAuth. To fix this, we need to add custom typings. Here is how we do it:

Step 1: Create a file named *types/auth.d.ts* at the same level as *src*. Here, we will extend the core `Session` type with our custom type.

```typescript
import type { DefaultSession } from '@auth/core/types';

declare module '@auth/core/types' {
	interface Session {
		user: DefaultSession['user'] & Partial<IUser>;
	}
}

interface IUser {
	name: string;
	email: string;
	image: string;
	picture: string;
	sub: string;
	access_token: string;
	id_token: string;
	scope: string;
	expires_in: number;
	token_type: string;
	expires_at: number;
	provider: string;
	type: string;
	providerAccountId: string;
	nickname: string;
	updated_at: string;
	email_verified: boolean;
	iss: string;
	aud: string;
	iat: number;
	exp: number;
	sid: string;
	jti: string;
	fav_num: string;
}
```

Step 2: Update the *tsconfig.json* file's *types* and *include* properties.

```json
{
  "extends": "./.svelte-kit/tsconfig.json",
  "compilerOptions": {
    ...,
    "types": ["vite/client", "svelte", "@auth/core", "@auth/sveltekit"]
  },
  "include": [
    ...,
    "types/**/*.ts"
  ]
}
```

Step 3: Restart your IDE.

In conclusion, enhancing SvelteKitAuth with custom type additions is a straightforward process that can significantly improve your development experience. By creating a custom typings file and updating your TypeScript configuration, you can ensure that your IDE recognizes all necessary types, reducing errors and improving code quality. Remember to restart your IDE after making these changes to apply the updates effectively. This small but crucial step can make a big difference in maintaining a smooth and efficient development workflow.

Here is the link to the [**GitHub repository**](https://github.com/aakash14goplani/SvelteKitAuth) with the codebase. In the next section, we will delve into initiating [client-side sign-in and sign-out flows](https://blog.aakashgoplani.in/streamlining-client-side-sign-in-and-sign-out-processes), further streamlining the user authentication journey.