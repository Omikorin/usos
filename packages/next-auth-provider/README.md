# @usos/next-auth-provider

NextAuth.js one and only USOS provider. Unfortunately, USOS only supports deprecated OAuth 1.0a.

## Installation

To install `@usos/next-auth-provider`, use your favourite package manager:

```bash
pnpm install @usos/next-auth-provider
```

## Config
Below is an example of an `.env` file you can use for your server configuration.
It is safe to mark them all as `NEXT_PUBLIC_...`.

```sh
USOS_CLIENT_ID=""
USOS_CLIENT_SECRET=""
USOS_BASE_URL=https://apps.your-usos-instance.pl
USOS_FIELDS="id|first_name|last_name|email"
USOS_SCOPES="email"
PUBLIC_URL=http://localhost:3000
```

The `PUBLIC_URL` is a path where you should upload the USOS logo.
It should be named `usos-logo.png`. For example: `public/usos-logo.png`.

## Usage

### App router

Create a file named `auth.ts` in the project `src/lib` directory with the following code:

```ts
import UsosProvider from '@usos/next-auth-provider';
import { AuthOptions, getServerSession } from 'next-auth';
import { Provider } from 'next-auth/providers/index';

const providers: Provider[] = [];

if (process.env.USOS_CLIENT_ID && process.env.USOS_CLIENT_SECRET) {
  providers.push(
    UsosProvider({
      clientId: process.env.USOS_CLIENT_ID,
      clientSecret: process.env.USOS_CLIENT_SECRET,
      usosBaseUrl: process.env.USOS_BASE_URL,
      publicUrl: process.env.PUBLIC_URL,
      usosScopes: process.env.USOS_SCOPES,
      usosFields: process.env.USOS_FIELDS,
      allowDangerousEmailAccountLinking: true,
    }),
  );
}

if (providers.length === 0) {
  throw new Error('No authentication providers configured.');
}

const authOptions: AuthOptions = {
  pages: {
    signIn: '/auth/login',
  },
  providers,
  session: {
    strategy: 'jwt',
  },
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
      }
      return token;
    },
    async session({ session, token }) {
      if (token && session.user) {
        // use USOS account id as our user id, you can change the id to something else if you want
        session.user.id = token.id as string;
      }
      return session;
    },
  },
};

const getSession = () => getServerSession(authOptions);

declare module 'next-auth' {
  interface Session {
    user: {
      id: string; // fix types so they have our user id
      name?: string | null;
      email?: string | null;
      image?: string | null;
    };
  }
}

export { authOptions, getSession };
```

Next, expose a route handler in the `src/app/api/auth/[...nextauth]` dir:

```ts
import { authOptions } from '@/lib/auth';
import NextAuth from 'next-auth';

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

# License

This project is licensed under the terms of the [ISC license](https://github.com/Omikorin/usos/blob/main/LICENSE).
