This document explains how this application system handles, stores, and formats money.

## Transactions

### Primer

There are 2 primary ways to represent transaction amounts in an application system:

1. Using signed amounts (i.e. `{ amount: -20 }`)
2. Using unsigned amounts plus a transaction type (i.e. `{ amount: 20, type: 'INFLOW' }`)

This repository uses a **combination of the two approaches**.  See the DB schema below:

| amount | type |
| -- | -- |
| 20 | outflow |
| -20 | inflow |

In the table above, `type` is a *computed column* based on the signage of `amount`.  If `amount < 0`, the transaction is an *inflow* and vice-versa.  By keeping the signage in the `amount` column, it becomes easier to write aggregation queries such as:

```
-- Get net transaction amounts
SELECT sum(amount) FROM ...

-- Get inflows/outflows
SELECT 
  abs(sum(amount)) FILTER (WHERE type = 'inflow'),
  sum(amount) FILTER (WHERE type = 'outflow')
  ...
```

## Account Balances

No matter what approach is used, account balances are generally represented as **positive values in their "normal state"**. Account balances _can_ be negative, unlike transaction amounts, which are always positive.

| Account Type            | Amount | Description                                                         | "State"  |
| ----------------------- | ------ | ------------------------------------------------------------------- | -------- |
| Savings (Asset)         | $500   | "I have $500 in my savings account"                                 | Normal   |
| Savings (Asset)         | -$500  | "My account is overdrafted because I spent money I didn't have"     | Abnormal |
| Credit Card (Liability) | $200   | "I carry a credit card balance of $200 and need to pay it off soon" | Normal   |
| Credit Card (Liability) | -$200  | "I overpaid my credit card, so now I have a negative liability"     | Abnormal |

### Mutating Account Balances

Because we store account balances as positive values in their "normal" state, "inflows" and "outflows" have a different effect on account balance based on the account type.

#### Base Formula

##### Assets

An "inflow" to an asset account will increase the balance of that account (i.e. you have "more assets").

- `endBalance = startBalance + inflows - outflows`
- `startBalance = endBalance - inflows + outflows`

##### Liabilities

An "inflow" to a liability account will **decrease** the balance of that account (i.e. money went **into** the account, and therefore, you have **less of a liability** now)

- `endBalance = startBalance - inflows + outflows`
- `startBalance = endBalance + inflows - outflows`

## Storing and Formatting Money

All monetary values are **stored in major currency units (i.e. "dollars and cents" for USD)** as a `Decimal` DB type.  [Decimal.js](https://mikemcl.github.io/decimal.js/) is used throughout the codebase for computations, and the `Number.Intl` web API is used for formatting.