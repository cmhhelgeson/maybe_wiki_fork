These are some frequently asked questions about moving Maybe to an open source product.

**What can I do to help? How can I contribute?**\
Welcome! Please read our [contribution guide](https://github.com/maybe-finance/maybe/blob/main/CONTRIBUTING.md). As an open source project, you're welcome to run Maybe locally and find anything you want to fix or change.

**What connection sources do you use for banks?**\
We are in the process of using [Teller](https://teller.io/) for its self-host-friendly free tier. The app previously used Plaid and Finicity, but those have been removed due to their need for an enterprise contract to be useful.

**Where is the price data for assets coming from?**\
The app currently uses [Polygon.io](https://polygon.io/).

**Have you thought about using _[data provider]_?**\
The goal is to have a wide array of data sources and connection providers to choose from to cover nearly every financial institution. This will take quite a bit of work, and PRs are highly encouraged. Keep in mind that we’re aiming to keep this self hostable, so the provider should have an accessible and useful free tier if possible.
> [!NOTE]  
> See [discussions about Data Connectors](https://github.com/maybe-finance/maybe/discussions/categories/data-connectors).

**What countries does Maybe support?**\
Currently only the United States. But with our goal of a wide array of data connectors comes global support!

**Do I need to use a bank connection or can I use manual entry?**\
Manual transactions are not yet fully supported, but are planned.

**Have you thought about rewriting *[part of the stack]* in *[stack that you prefer]*?**\
Our first priority is getting the app functioning again with the existing tech stack. Since this is an open source project you’re more than welcome to fork and re-write in whatever stack makes you happy!

**Are there any plans for adding budgeting tools?**\
Yes, after the foundations are working.

**I’m trying to run the app locally and running into errors and issues. What should I do?**\
Make sure you're up to date and then check [existing issues](https://github.com/maybe-finance/maybe/issues) to see if someone else has already reported it. If your issue hasn't previously been reported then please create an issue and provide as much detail around your machine/setup, steps performed, and debugging attempts to date.