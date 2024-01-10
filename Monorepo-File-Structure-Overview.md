This repository uses the monorepo management tools from NRWL called `nx`. The NX dev tools come with pre-configured TS, ESLint, Prettier, Jest, Cypress and more, and provide an extensive suite of commands to lint, test, serve, and build applications. [Here's a great 10 minute video](https://nx.dev/l/r/getting-started/intro#10-minute-nx-overview) that walks through what nx can do.

### Structure

**Notes**

Through ESLint, the following rules apply:

- libs/shared can be imported to any app OR lib
- libs/server can be imported to server and workers apps
- libs/client can be imported to client app
- libs/client/shared can be imported to any libs/client/features
- libs/server/shared can be imported to any libs/server/features

```
apps/
  client/
  server/
  workers/
libs/
  design-system/
  client/
    src/
      features/
        {feature1-name}/
        {feature2-name}/
      shared/
  server/
    src/
      features/
        {feature1-name}/
        {feature2-name}/
      shared/
  shared/
    src/
      features/
        {feature1-name}/
        {feature2-name}/
      utils/
      types/
```

### Decision Tree

![folder-structure-diagram](https://github.com/maybe-finance/maybe/assets/35243/fbe702b0-a6a7-41b1-a677-80ff7fe7f656)