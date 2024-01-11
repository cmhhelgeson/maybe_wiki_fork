**Note: This is potentially outdated as it was written for the original version of Maybe that we're now working on updating. It should give _some_ insight to help guide you, though.**

## `AccountConnection`

Represents a connection to an institution (i.e. Plaid)

- `type` - A single table inheritance value that can be `plaid | some-other-institution`
- `status` - Identifies the state of the connection with the institution provider, such as Plaid.
  - `OK` - default state, no connection issues
  - `ERROR` - the provider (i.e. Plaid) has detected an issue with the user's connection that requires some sort of user action to resolve (login in again, sync again, etc.)
  - `DISCONNECTED` - the user has manually disconnected (but not deleted) this connection.  Provider details are stored, but an account with this state will not continue to sync transactions, holdings, etc.
- `sync_status` - Determines the stage of syncing this connection is in.  
  - `IDLE` - Not syncing any underlying accounts
  - `PENDING` - An `AccountConnection` will be pending when it has first been created, but not yet synced
  - `SYNCING` - When *any* of the underlying `Account`s associated with this connection are syncing

## `Account`

An `Account` is main concept of value within the system, and can either be directly linked to `User` via `user_id`, or directly linked to `AccountConnection` via `account_connection_id`.

**Some fields are omitted below** 

- `start_date` - Represents the user-defined (or system defined) date in which account valuations should start being populated.  If `start_date` is `2021-01-01`, any date *prior to* this date will have a `0` balance.
- `type` - A single table inheritance value that can be `plaid | property | vehicle | other`, which subsequently determines which fields have non-null values in the table.
- `current_balance` - the "all-in" account balance (what you see when you login to your institution).  This will be different depending on the type of account.  For a checking account, this is the total amount before including any pending transactions.  For an investment account, this would be cash + investments value.
- `available_balance` - This is also different depending on the account type.  For a checking account, this is the balance after incorporating pending transactions.  For an investment account, this represents the brokerage cash available to spend.
- `valuation_type` - tells us *how* the account is being valued
- `valuation_source` (computed) - populated based on the following logic:

```sql
ADD COLUMN  "valuation_source" TEXT GENERATED ALWAYS AS (
  CASE 
    WHEN "valuation_type" = 'PLAID_TRANSACTION'
        OR "valuation_type" = 'PLAID_INVESTMENT_TRANSACTION'
        OR "valuation_type" = 'PLAID_VALUATION'
        THEN 'plaid'
    WHEN "valuation_type" = 'USER_VALUATION' THEN 'user'
    WHEN "valuation_type" = 'KBB_VALUATION' THEN 'kbb'
    WHEN "valuation_type" = 'ZILLOW_VALUATION' THEN 'zillow'
    ELSE 'other'
  END
) STORED;
```

- `valuation_method` (computed) - determines the sync method that is used to value this account and populate historical balances

```sql
ADD COLUMN  "valuation_method" TEXT GENERATED ALWAYS AS (
  CASE 
    WHEN "valuation_type" = 'PLAID_TRANSACTION' THEN 'transaction'
    WHEN "valuation_type" = 'PLAID_INVESTMENT_TRANSACTION' THEN 'investment-transaction'
    ELSE 'valuation'
  END
) STORED;
```

- `classification` - either `asset` or `liability`
- `category` (computed) - user defined, falls back to Plaid category (if `type` is `plaid`)

```sql
ALTER TABLE "account"
ADD COLUMN  "category" TEXT GENERATED ALWAYS AS (
  CASE 
    WHEN "type" = 'plaid' AND "plaid_type" IN ('depository') THEN 'cash'
    WHEN "type" = 'plaid' AND "plaid_type" IN ('investment' ,'brokerage') THEN 'investment'
    WHEN "type" = 'plaid' AND "plaid_type" IN ('loan') THEN 'loan'
    WHEN "type" = 'plaid' AND "plaid_type" IN ('credit') THEN 'credit'
    WHEN "type" = 'property' THEN 'property'
    WHEN "type" = 'vehicle' THEN 'vehicle'
    WHEN "type" = 'other' THEN 'other'
  END
) STORED;
```

- `subcategory` (computed) 

```sql
ALTER TABLE "account"
ADD COLUMN  "subcategory" TEXT GENERATED ALWAYS AS (
  CASE 
    WHEN "subcategory_override" IS NOT NULL THEN "subcategory_override"
    WHEN "type" = 'plaid' THEN "plaid_subtype"
    WHEN "type" = 'property' THEN 'property'
    WHEN "type" = 'vehicle' THEN 'vehicle'
    ELSE 'other'
  END
) STORED;
```

- `subcategory_override` - If defined, `subcategory` will assume this value
- `sync_status` - either `idle | pending | syncing`.
  - `pending` - An `Account` is pending after it is created, but before its balances have synced
  - `syncing` - An `Account` is explicitly syncing when marked, OR implicitly when its parent `AccountConnection.sync_status` is equal to `SYNCING`
  - `idle` - default state