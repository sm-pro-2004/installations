# CI YAML File

```

name: FleetFlow CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: ci-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  backend:
    name: Laravel tests
    runs-on: ubuntu-24.04
    timeout-minutes: 10

    env:
      APP_ENV: testing
      DB_CONNECTION: sqlite
      DB_DATABASE: ':memory:'
      DB_URL: ''
      CACHE_STORE: array
      SESSION_DRIVER: array
      QUEUE_CONNECTION: sync
      MAIL_MAILER: array
      INERTIA_SSR_ENABLED: 'false'

    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false

      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.5'
          extensions: mbstring, dom, xml, pdo_sqlite, sqlite3
          coverage: none
          tools: composer:v2

      - name: Locate Composer cache
        id: composer
        run: echo "dir=$(composer config cache-files-dir)" >> "$GITHUB_OUTPUT"

      - name: Cache PHP downloads
        uses: actions/cache@v5
        with:
          path: ${{ steps.composer.outputs.dir }}
          key: composer-${{ runner.os }}-php85-${{ hashFiles('composer.lock') }}
          restore-keys: composer-${{ runner.os }}-php85-

      - name: Install PHP dependencies
        run: composer install --no-interaction --prefer-dist --no-progress

      - name: Prepare Laravel
        run: |
          cp .env.example .env
          php artisan key:generate --no-interaction
          php artisan config:clear

      - name: Run Laravel tests
        run: |
          mkdir -p test-results
          php artisan test --compact --log-junit test-results/laravel.xml

      - name: Save test report
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: laravel-test-results
          path: test-results/laravel.xml
          if-no-files-found: warn
          retention-days: 7

  frontend:
    name: React tests and build
    runs-on: ubuntu-24.04
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false

      - uses: actions/setup-node@v7
        with:
          node-version: '24'
          cache: npm
          cache-dependency-path: package-lock.json

      - name: Install frontend dependencies
        run: npm ci --include=optional --no-audit --no-fund

      - name: Check TypeScript
        run: npm run typecheck

      - name: Run React and Zod tests
        run: npm test

      - name: Build frontend
        run: npm run build

      - name: Save frontend build
        uses: actions/upload-artifact@v4
        with:
          name: frontend-build
          path: public/build/
          if-no-files-found: error
          retention-days: 7
```
          
