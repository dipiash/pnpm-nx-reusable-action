# Action for cache PNPM and NX dependencies

## Cache paths
- ~/.pnpm-store
- **/node_modules
- node_modules/.cache/nx

## How to use

Real example you can see [here](https://github.com/dipiash/nx-ts-vite-react-graphql-styled-monorepo-example).

For example workflow action for PR checks.

```yaml
name: Check Pull Request
on:
  push:
    branches:
      - main
  pull_request:
jobs:
  cache-and-install:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Cache data for pnpm and node_modules
        uses: actions/cache@v3
        with:
          path: |
            ~/.pnpm-store
            **/node_modules
          key: ${{ runner.os }}-${{ hashFiles('**/pnpm-lock.yaml') }}
          restore-keys: |
            ${{ runner.os }}-

      - uses: pnpm/action-setup@v2.1.0
        with:
          version: 7.1.5
          run_install: true

  type-check:
    runs-on: ubuntu-latest
    needs:
      - cache-and-install
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Reusable Action for PNPM and NX
        uses: dipiash/pnpm-nx-reusable-action@v2

      - name: type-check PR
        if: github.ref != 'refs/heads/main'
        run: npx nx affected --target=type-check --base=origin/main --head=HEAD --with-deps
      - name: type-check Main
        if: github.ref == 'refs/heads/main'
        run: npx nx affected --target=type-check --base=origin/main~1 --head=origin/main --with-deps
```
