**⚠️ NOTE: This is for reference only as we work to [remove Auth0 from the codebase](https://github.com/maybe-finance/maybe/issues/16)!**

----

As authentication/authorization is a huge topic, the [Auth0 docs](https://auth0.com/docs/get-started) are tough to navigate.  The purpose of this page is to leave a few breadcrumbs that explain how **this application** leverages Auth0.

Currently, this repo uses 2 separate authentication flows.

1. [Authorization Code Flow with PKCE](https://auth0.com/docs/authorization/flows#authorization-code-flow-with-proof-key-for-code-exchange-pkce-) - This is the recommended way to authenticate and authorize users in a Single Page Application (SPA), and is how our users authenticate in this app.  We are using the [**Classic Universal Login** experience](https://auth0.com/docs/authenticate/login/auth0-universal-login/new-universal-login-vs-classic-universal-login).
2. [Authorization Code Flow](https://auth0.com/docs/authorization/flows#authorization-code-flow) - This repo hosts a [Bull](https://www.npmjs.com/package/bull) dashboard, which can only be accessed by **admin** "Roles" (i.e. Maybe Finance engineering team)

## Part 1: SPA Authentication / Authorization

### How it works

1. Unauthenticated user registers or logs in via [Auth0 Universal login](https://auth0.com/docs/login/universal-login) (via a [custom domain](https://auth0.com/docs/brand-and-customize/custom-domains), `login.maybe.co`)
2. Client is granted an **access token**, which is managed via the `@auth0/auth0-react` package (which uses [Authorization Code Flow with PKCE under the hood](https://auth0.com/docs/libraries/auth0-react#:~:text=Under%20the%20hood%2C%20it%20implements%C2%A0Universal%20Login%C2%A0and%20the%C2%A0Authorization%20Code%20Grant%20Flow%20with%20PKCE)). This package exposes the `useAuth0` hook, which allows the client to obtain [OpenID](https://auth0.com/docs/authorization/protocols/openid-connect-protocol) information about the authenticated user.
3. The [AxiosProvider](https://github.com/maybe-finance/maybe/blob/main/libs/client/shared/src/providers/AxiosProvider.tsx) uses an interceptor to silently obtain this access token and send it in the `Authorization` header of each API request.

```ts
const accessToken = await getAccessTokenSilently({
    audience: process.env.API_URL,
})

if (accessToken) {
    axiosRequestConfig.headers.Authorization = `Bearer ${accessToken}`
}
```

4. The server will then validate this access token with the [validateAuth0Jwt middleware](https://github.com/maybe-finance/maybe-app/blob/main/libs/server/shared/src/middleware/validate-auth0-jwt.ts) and attach the user to `req.user`. If the JWT is valid, the client will be able to make API requests.

### How users are associated with internal DB

All PII is stored in Auth0, but the app needs a unique identifier for each user.  On first login/registration, the user is saved to the DB and the [`auth0_id` field is populated](https://github.com/maybe-finance/maybe/blob/main/libs/server/shared/src/middleware/validate-auth0-jwt.ts#L28) with the user's unique Auth0 identifier (no collisions guaranteed).

## Part 2: Admin pages

Admin pages are served via Express static template files (ejs) and use a server-side only Auth flow via the `express-openid-connect` package.  [Here is an overview of the flow used](https://auth0.com/docs/authorization/flows/authorization-code-flow#how-it-works).

The `express-openid-connect` middleware will attach the following routes to the app:

- `/admin/login`
- `/admin/logout`
- `/admin/callback`

When the flow is complete, the user session is stored on `req.oidc`.

### Roles

To authorize admin users, the [adminClaimCheck](https://github.com/maybe-finance/maybe/blob/2d1c2c3aa293fc78b2b3f17a84d164ac4ec9ad03/apps/server/src/app/admin/admin-router.ts#L47) middleware will grab a custom claim from the decoded access token, check if the user has "Admin" as one of his/her roles, and if not, deny access to all `/admin` pages.

By default, [Auth0 Roles](https://auth0.com/docs/authorization/rbac) are not attached to the access token, and must be [added through a rule](https://auth0.com/docs/authorization/authorization-policies/sample-use-cases-rules-with-authorization#add-user-roles-to-tokens) via a [custom, namespaced claim](https://auth0.com/docs/security/tokens/json-web-tokens/create-namespaced-custom-claims).
