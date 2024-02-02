## What is the broad vision?

We are building a "COSS Personal Finance OS" (commercial, open-source personal finance operating system)

What does this mean?

We're hard at work to figure this out, but here's a "quickstart":

- Personal finance tools that grow with your life circumstances
- A "self-service" toolbox to help you make better decisions about your money

Think of it in terms of "Maybe Core" and "Maybe Apps":

- **Core** - the "building blocks" of personal finance
  - Sign up individually and add your family
  - Add your accounts (checking, savings, credit card, loans, equity, crypto, etc.)
  - Manage your transactions (rules engine, quick search, categorization, notes, etc.)
- **Apps** - only choose what you need at your _current stage of life_
  - Insights and Reporting
    - Net worth
    - Income and expenses by category, tags, accounts, etc.
    - Key ratios (debt/income, runway, etc.)
  - Budgeting
  - Goal setting
  - Retirement planning
  - Investment tracking
  - TBD...

## Why should a contributor care about the Vision?

We're in the early days of a Rails rebuild, but we are NOT starting from _scratch_.

Maybe 1.0 was written in a different framework and ecosystem, but we built a solid data model and some solid abstractions and patterns for dealing with personal finance.

Our goal is to "lift and shift" many of these ideas to Rails.

To understand how and where to contribute, you need should be familiar with what we are planning to _change_ and what we want to _keep_.

## What is "Maybe Core"?

We spent 2 years building Maybe 1.0. During that time, our team worked through a lot of engineering decisions around how to build a personal finance app.

Some of these decisions were good. Some were not. And some are no longer relevant to an open-source codebase.

The goal of this section is to help YOU (a capable Rails dev who probably knows a bit about personal finance as well) get a quick glance of what "Maybe Core" represents and where the first iteration of this is headed.

Moving from closed-source to open-source changes our assumptions and feature set. Here are some big ideas to keep in mind.

### Core Views

Maybe "Core" is largely a backend concern, so the views required are minimal:

- Dashboard
  - `/`
  - Provide a historical net worth graph
  - Provide basic insights for user
- Accounts
  - `/accounts`
  - Allows user to see all accounts and do account-related CRUD actions
- Account
  - `/account/:id`
  - Shows historical balance graph of individual account
  - Uses delegated types pattern to show "type-specific" views (i.e. a `LoanAccount` might show data about the loan such as interest rate, remaining principal balance, etc.)
- Transactions
  - `/transactions`
  - Provides a "command center" for transactions across all accounts (explained in later section—search, filtering, sorting, pagination, rules, etc.)

### Core Concepts

#### Concept #1: Get to 100% Accuracy with a "Demo User"

In Maybe 1.0, we placed a large focus on syncing user account data from data providers such as Plaid and Finicity.

With open-source, it will be the _reverse_. In our opinion, this is an advantage. In Maybe 1.0, we spent countless hours fixing data provider bugs and this got in the way of focusing on what truly matters—**100% calculation accuracy**.

The great news with finance is there is _always a right answer_. A calculation can be either be:

- Correct
- Wrong because of bad math/programming (in our control)
- Wrong because of bad data (out of our control)

And when we have to aggregate a user's "net worth" into a single view, if there is _anything_ wrong in the underlying calculations or data, the view is wrong.

In personal finance, wealth is not the only things that compounds exponentionally—calculation mistakes do too.

With this OSS version of the app, our _primary goal_ is twofold:

1. Create a rock-solid "demo user", which serves multiple purposes:
   - Allows new users to demo the product easily
   - Gives engineers clean test data to work with (for test suites, views, etc.)
   - Gives engineers confidence that we have the "right answer" (without tracking down the source of "bad data")
2. Get the app to calculate everything with 100% accuracy for the demo account

Once we have a rock-solid "core" that can handle manual accounts, layering in data from external providers will be much easier.

#### Concept #2: "Self Service" Philosophy

Moving forward, the app will not deal with "advising". This was something we did in Maybe 1.0 that required a lot of overhead in terms of both compliance and costs.

In Maybe 2.0, we should be thinking in terms of a "self-service" model.

It will be much more of a "here's your data presented in several different ways so you can make good choices".

Things to consider here:

- Giving user option to include/exclude "apps" (i.e. "grab what apps you need for your current stage of life")
- Building an editable "rules engine" for classifying transactions

#### Concept #3: Dealing with Money (Globally this time)

With an OSS community, we're better equipped to tackle global currency concerns. In Maybe 1.0, everything was centered around USD.

This time around, we'll support global accounts and currencies.

Since we didn't implement this in 1.0, we'll need to build it fresh. Here are some things to consider:

##### Storing Monetary Values

You can't have "10 monies". 10 USD is very different than 10 CAD. To determine value of money, we need to store:

- `amount_cents` - stored in _minor currency units_ (e.g. for USD, we'd store `100` "cents" to represent $1.00). Generally, a `BigInt` type is good here as it is large enough for any currency (think cryptocurrencies that have minted an extremely high number of coins).
- `currency_iso` - the ISO currency code

By having both pieces of information, we can integrate directly with libraries like `money-rails` to seamlessly convert in/out of minor currency units, do formatting, and with "bolt on" an exchange rate system via something like [Open Exchange Rates](https://openexchangerates.org/).

##### Signage of Money

Some approaches talk about storing all amounts as positive numbers and add a separate `INFLOW/OUTFLOW` field.

The _simpler_ approach is to just store money as signed values (both negative and positive).

With this approach, a positive value is an "inflow" and a negative value is an "outflow".

Depending on `Account.type`, this will either increase or decrease the value of the account (i.e. "debits" and "credits"):

- Checking Account
  - Positive value _increases_ account value
  - Negative value _decreases_ account value
- Credit Card Account
  - Positive value _decreases_ account value (a "payment" made to account)
  - Negative value _increases_ account value (i.e. "spending" with your CC)

##### Formatting Money in the UI

A "default currency" should likely be set at either the `User` or `Family` level. An obvious default here would be `USD`.

For something like a net-worth graph where there _could_ be transactions with multiple currency types, we'll need a way to determine what "default" currency to show the graph in.

For something like a _transaction_, the formatting is simpler because we just use what's stored in the DB for that individual transaction.

The `money-rails` gem makes this much easier:

```rb
# Directly integrates to model
class Account < ApplicationRecord
  # Easy conversion in and out of minor currency units
  monetize :balance_cents, as: "balance", with_currency: :currency
end

# Mutates both `amount` and `currency` fields in one line
account = Account.create(balance: Money.new(50000, "GBP")) # £500.00

# Easy formatting in the view:
<%= account.balance.format %> <!-- Output will be something like: $10.00 -->
```

#### Concept #4: Transactions "Command Center"

Whether it's a basic credit card transaction for your Chipotle meal or a more complex brokerage transaction with `qty`, `shares`, `price`, `cusip`, the concept of a "Transaction" is the basic "building block" of a personal finance app.

Especially since we're starting out "100% manual", users will need a "command center" for viewing and managing transactions across accounts.

This includes but is not limited to things like:

- Searching (full text search)
- Sorting
- Filtering (view “All”, “by account”, “by tag”, “by category”, “by date”, etc.)
- CRUD operations (single or bulk)
- Transaction Import/Exports
- Customizable "Rules Engine" for classifying transactions
  - Reviewing “auto classified” transactions (once 3rd party data is integrated)

#### Concept #5: Charts (long-term)

Having a standardized and composable charts module will be a long-term priority.

Maybe 1.0 was known for having clean and interactive charts to view historical data. We previously used Visx (a React library on top of D3.js) which made this a bit easier.

Now that we're in Rails, a vanilla D3.js integration would be a great addition for building Maybe-branded, "standardized" charts.
