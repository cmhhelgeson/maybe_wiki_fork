*This page may not represent the current state of the API and is primarily here as a quick reference to see everything in one spot.*

### Users

* `GET /users`
* `GET /users/net-worth`
* `GET /users/:id/net-worth` - admin only
* `GET /users/net-worth/:date`
* `GET /users/:id/net-worth/:date` - admin only
* `GET /users/:id/account-rollup`
* `POST /users/link-accounts` - links 2 Auth0 identities into one
* `POST /users/unlink-account`
* `POST /users/resend-verification-email` - resends the Auth0 verification email
* `PUT /users`
* `PUT /users/change-password`
* `DELETE /users`

### Account connections

* `GET /connections/:id`
* `POST /connections/:id/plaid/link-update-completed`
* `POST /connections/:id/disconnect`
* `POST /connections/:id/reconnect`
* `POST /connections/:id/link-token`
* `POST /connections/:id/sync`
* `POST /connections/:id/sync/:sync-method`
* `POST /connections/:id/plaid/sandbox/fire-webhook` - local dev only
* `POST /connections/:id/sandbox/item-reset-login` - local dev only
* `DELETE /connections/:id`
* `DELETE /connections` - local dev only

### Accounts

* `GET /accounts`
* `GET /accounts/:id`
* `GET /accounts/:id/balances`
* `GET /accounts/:id/transactions`
* `GET /accounts/:id/holdings`
* `GET /accounts/:id/valuations`
* `POST /accounts`
* `POST /accounts/:id/valuations`
* `POST /accounts/:id/sync`
* `PUT /accounts/:id`
* `DELETE /accounts/:id`

### Webhooks

* `POST /webhooks/plaid/webhook`
* `POST /webhooks/finicity/webhook`

### Account rollups

* `GET /account-rollups` - the sidebar account rollups

### Valuations

* `GET /valuations/:id`
* `PUT /valuations/:id`
* `DELETE /valuations/:id`

### Institutions

* `GET /institutions` - gets our internal DB of institutions (combined from Plaid, Finicity)
* `POST /institutions/sync` - syncs all institutions from Plaid and Finicity APIs

### Plaid

* `POST /plaid/link-token`
* `POST /plaid/exchange-public-token`
* `POST /plaid/sandbox/quick-add` - dev only
