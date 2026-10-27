# Docker Project Setup Guide — macOS

Laravel 13 · React 19 · TypeScript · Inertia 3 · Livewire 4 · Next.js · Flutter · Dart · C · React Native

Prepared: October 27, 2026. This is a local development guide, not a production deployment recipe.

Run commands one at a time. Finish a section's checkpoint before continuing. Each project gets its own folder and its own `compose.yaml`.

## Contents

1. [Install tools and understand Docker](#1-install-tools-and-understand-docker)
2. [Plain Laravel 13 with MySQL and Redis](#2-plain-laravel-13-with-mysql-and-redis)
3. [Manual React 19 + TypeScript + Inertia — no Fortify](#3-manual-react-19--typescript--inertia--no-fortify)
4. [Official Laravel React starter kit — includes Fortify](#4-official-laravel-react-starter-kit--includes-fortify)
5. [Laravel + Livewire — no Fortify](#5-laravel--livewire--no-fortify)
6. [phpMyAdmin](#6-phpmyadmin)
7. [Next.js + TypeScript](#7-nextjs--typescript)
8. [Flutter web in Docker](#8-flutter-web-in-docker)
9. [Dart command-line projects](#9-dart-command-line-projects)
10. [C projects](#10-c-projects)
11. [React Native with Expo](#11-react-native-with-expo)
12. [Everyday commands and recovery](#12-everyday-commands-and-recovery)
13. [Save projects and this guide on GitHub](#13-save-projects-and-this-guide-on-github)
14. [Official references and validation notes](#14-official-references-and-validation-notes)

## Choose your path

| You want | Follow |
|---|---|
| Laravel backend / Blade only | Sections 1 → 2 |
| Laravel + React + TypeScript + Inertia, no authentication | Sections 1 → 2 → 3 |
| Laravel React starter kit with login/register | Sections 1 → 4; do not create the plain project first |
| Laravel + Livewire, no authentication | Sections 1 → 2 → 5 |
| Database browser for Laravel | Add section 6 |
| Next.js | Sections 1 → 7 |
| Flutter browser app | Sections 1 → 8 |
| Dart console app | Sections 1 → 9 |
| C console app | Sections 1 → 10 |
| React Native / Expo | Sections 1 → 11 |

React and Livewire are separate frontend choices here. Do not combine sections 3, 4, and 5 in the same fresh project.

## 1. Install tools and understand Docker

### 1.1 Install Docker Desktop

Open Terminal with **Command + Space → Terminal**.

```bash
uname -m
```

`arm64` means Apple silicon; `x86_64` means Intel. Download the matching [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/), drag it into Applications, and open it. Wait for the engine to start.

```bash
docker --version
```

```bash
docker compose version
```

```bash
docker run --rm hello-world
```

Checkpoint: versions print and the last command prints `Hello from Docker!`.

### 1.2 Check Git and install an editor

```bash
git --version
```

If Git is missing, install Apple's command-line tools:

```bash
xcode-select --install
```

Use your preferred editor or install [VS Code](https://code.visualstudio.com/).

### 1.3 Create a workspace

```bash
mkdir -p ~/Developer
```

```bash
cd ~/Developer
```

`mkdir` creates folders; `cd` changes folders; `pwd` prints your current folder; `exit` leaves a container shell.

### 1.4 Docker vocabulary

| Term | Meaning |
|---|---|
| Image | Software and instructions used to create a container |
| Container | A running environment created from an image |
| Bind mount | Shares a Mac folder with a container, such as your code |
| Named volume | Docker-managed storage, such as MySQL records |
| Dockerfile | Instructions for building an image |
| Compose | Describes and connects a project's services |
| `8000:80` | Mac port 8000 forwards to container port 80 |
| `127.0.0.1` | This computer; use it in your browser |
| `0.0.0.0` | Listen on all interfaces; usually a server binding, not your browser URL |

PHP, Composer, Node.js, Dart, and GCC run inside containers in this guide. Native mobile tools are explained separately.

## 2. Plain Laravel 13 with MySQL and Redis

This creates plain Laravel without Fortify, login, or registration screens. The framework's default `users` migration may exist; a users table does not mean authentication screens are installed.

Use `my-laravel` as the example folder name. Replace it consistently when starting another project. The destination must not already exist. Keep old projects as backups instead of deleting them.

### 2.1 Open a temporary PHP container

**Mac Terminal:**

```bash
cd ~/Developer
```

Copy this one-time command as a single line:

```bash
docker run --rm -it -v "$PWD:/app" -w /app --entrypoint sh laravelsail/php84-composer:latest
```

This supplies PHP 8.4 and Composer. `-v` shares your current folder, `-w` selects the container folder, and `--rm` removes the temporary container when you exit. Project files stay on your Mac.

### 2.2 Create Laravel

**Inside the temporary container:**

```bash
export COMPOSER_ALLOW_SUPERUSER=1
```

```bash
composer create-project laravel/laravel my-laravel "^13.0" --no-scripts
```

The version constraint selects Laravel 13. `--no-scripts` defers setup until the full Docker environment exists.

```bash
cd my-laravel
```

```bash
cp .env.example .env
```

```bash
composer require laravel/sail --dev --no-scripts
```

```bash
php artisan package:discover
```

```bash
php artisan sail:install --with=mysql,redis
```

If asked to install frontend packages now, choose **No**. The temporary container is only for PHP setup.

```bash
exit
```

### 2.3 Configure the application

**Back in Mac Terminal:**

```bash
cd ~/Developer/my-laravel
```

```bash
alias sail='./vendor/bin/sail'
```

The shortcut lasts for this Terminal session. Repeat it in a new Terminal. Run Sail commands from the project root.

Open this project folder in your editor. In `.env`, replace matching entries with the following and add missing entries. Do not create duplicates:

```dotenv
APP_NAME=MyLaravel
APP_URL=http://127.0.0.1:8000
APP_PORT=8000

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=my_laravel
DB_USERNAME=sail
DB_PASSWORD=password
FORWARD_DB_PORT=3307

REDIS_CLIENT=phpredis
REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
FORWARD_REDIS_PORT=6380
```

These database credentials are only for local development. Inside Docker, Laravel reaches `mysql:3306` and `redis:6379`. From Mac database clients, MySQL uses `127.0.0.1:3307`.

Set the database name before the first MySQL startup. Changing `DB_DATABASE` later does not automatically create another database in an existing volume.

### 2.4 Start the environment

```bash
sail up -d --build
```

The first build can take several minutes. `-d` runs the services in the background.

```bash
sail ps
```

Wait until MySQL is healthy.

```bash
sail artisan key:generate
```

Generate the key only for a new app. Keep an existing application's `APP_KEY` when rebuilding containers.

```bash
sail artisan config:clear
```

```bash
sail artisan migrate
```

```bash
sail artisan --version
```

Checkpoint: the version is `13.x`, and migrations succeed.

### 2.5 Build the plain frontend

If you are immediately continuing to section 3, you can do that section's frontend installation instead. Otherwise:

```bash
sail npm install --include=optional
```

```bash
sail npm run build
```

Open **http://127.0.0.1:8000**. You should see Laravel's welcome page.

Verify Fortify is absent:

```bash
sail composer show laravel/fortify
```

Expected: Composer reports that the package is not found. That is the desired result for this path.

## 3. Manual React 19 + TypeScript + Inertia — no Fortify

Start from the working **plain Laravel project in section 2**. Keep the containers running. This path adds no authentication package.

### 3.1 Install the adapters and frontend tools

```bash
sail composer require inertiajs/inertia-laravel:^3.0
```

```bash
sail artisan inertia:middleware
```

```bash
sail npm install react@19 react-dom@19 @inertiajs/react@^3
```

```bash
sail npm install -D typescript @types/react@19 @types/react-dom@19 @vitejs/plugin-react @inertiajs/vite@^3
```

Laravel is the server, Inertia connects server routes to React pages, and TypeScript checks frontend types. This recipe uses the Inertia 3 Vite plugin and Vite 7/8, not the older Inertia 2 setup.

### 3.2 Register the middleware

Open `bootstrap/app.php`. Keep the existing routing and exception configuration. Add this import near the top:

```php
use App\Http\Middleware\HandleInertiaRequests;
```

Inside the existing `withMiddleware` callback, add:

```php
$middleware->web(append: [
    HandleInertiaRequests::class,
]);
```

That part of the file should read:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        HandleInertiaRequests::class,
    ]);
})
```

Do not append a second callback or replace unrelated application configuration.

### 3.3 Configure Vite

Replace `vite.config.js` with:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';
import inertia from '@inertiajs/vite';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/js/app.tsx'],
            refresh: true,
        }),
        react(),
        inertia(),
    ],
    server: {
        host: '0.0.0.0',
        port: 5173,
        strictPort: true,
        hmr: { host: '127.0.0.1' },
    },
});
```

For this minimal example, we use ordinary CSS instead of Tailwind. This avoids extra styling setup.

### 3.4 Create the HTML entry

Create `resources/views/app.blade.php`:

```blade
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        @viteReactRefresh
        @vite('resources/js/app.tsx')
        <x-inertia::head />
    </head>
    <body>
        <x-inertia::app />
    </body>
</html>
```

### 3.5 Create the React entry and page

Create `resources/js/app.tsx`:

```tsx
import '../css/app.css';
import { createInertiaApp } from '@inertiajs/react';

createInertiaApp({
    pages: {
        path: './pages',
        extension: '.tsx',
    },
});
```

The Inertia Vite plugin resolves and mounts pages. No manual `createRoot` is needed with this setup.

Create the page folder:

```bash
mkdir -p resources/js/pages
```

Create `resources/js/pages/Hello.tsx`:

```tsx
import { Head } from '@inertiajs/react';
import { useState } from 'react';

type Props = { message: string };

export default function Hello({ message }: Props) {
    const [count, setCount] = useState(0);

    return (
        <main>
            <Head title="My application" />
            <h1>{message}</h1>
            <p>Laravel + React 19 + TypeScript + Inertia</p>
            <button onClick={() => setCount(count + 1)}>
                Clicked {count} times
            </button>
        </main>
    );
}
```

Replace `resources/css/app.css` with:

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 3rem;
    background: #f4f6f8;
    color: #172033;
}

button {
    padding: 0.75rem 1rem;
    cursor: pointer;
}
```

### 3.6 Add TypeScript configuration

Create `tsconfig.json` in the project root:

```json
{
    "compilerOptions": {
        "target": "ES2022",
        "lib": ["ES2022", "DOM", "DOM.Iterable"],
        "module": "ESNext",
        "moduleResolution": "Bundler",
        "jsx": "react-jsx",
        "strict": true,
        "skipLibCheck": true,
        "noEmit": true,
        "isolatedModules": true,
        "allowSyntheticDefaultImports": true,
        "esModuleInterop": true,
        "types": ["vite/client"]
    },
    "include": ["resources/js/**/*.ts", "resources/js/**/*.tsx"]
}
```

### 3.7 Add the Laravel page route

Replace `routes/web.php` in this fresh project with:

```php
<?php

use Illuminate\Support\Facades\Route;
use Inertia\Inertia;

Route::get('/', fn () => Inertia::render('Hello', [
    'message' => 'Hello from Laravel!',
]));
```

Laravel sends `message` as a React prop. The page name and filename must match, including capitalization.

### 3.8 Check and run

```bash
sail npx tsc --noEmit
```

```bash
sail npm run build
```

Open **http://127.0.0.1:8000**. The heading and clickable counter should appear. There should be no login or registration screens.

For automatic updates while editing:

```bash
sail npm run dev
```

Keep this Terminal open. Edit the heading to confirm live updates. Stop Vite with **Control + C**. Old `resources/js/app.js` and `bootstrap.js` files may remain unused; this recipe loads `app.tsx` only.

## 4. Official Laravel React starter kit — includes Fortify

This is an alternative to sections 2–3. Use it only when you want the starter kit's authentication. Deselecting authentication features does not remove the Fortify package.

### 4.1 Create a separate project

**Mac Terminal:**

```bash
cd ~/Developer
```

```bash
docker run --rm -it -v "$PWD:/app" -w /app --entrypoint sh laravelsail/php84-composer:latest
```

**Inside the container:**

```bash
export COMPOSER_ALLOW_SUPERUSER=1
```

```bash
composer create-project laravel/react-starter-kit my-react-kit dev-main --no-scripts
```

`dev-main` uses the current official branch. It currently targets Laravel 13; it is a moving branch, so check the version after installation and commit the resulting lock files. Do not assume an old starter-kit tag targets Laravel 13.

```bash
cd my-react-kit
```

```bash
cp .env.example .env
```

```bash
php artisan package:discover
```

```bash
php artisan sail:install --with=mysql,redis
```

Choose **No** if asked to install frontend dependencies in this temporary container.

```bash
exit
```

### 4.2 Configure and start

**Mac Terminal:**

```bash
cd ~/Developer/my-react-kit
```

```bash
alias sail='./vendor/bin/sail'
```

Apply the `.env` values from section 2.3. Set a distinct `APP_NAME` and database name. Stop another project using the same published ports before starting this one.

```bash
sail up -d --build
```

```bash
sail artisan key:generate
```

```bash
sail artisan --version
```

Checkpoint: `13.x`. Then run the starter kit's setup inside the full application container:

```bash
sail artisan install:features
```

Use arrow keys to move, Space to toggle, and Enter to confirm. For a basic local login demo, enable registration and password confirmation; leave email verification, passkeys, and two-factor authentication disabled until you intend to configure them.

```bash
sail artisan migrate
```

```bash
sail npm install --include=optional
```

```bash
sail npm run build
```

Open **http://127.0.0.1:8000** and register. For development, run `sail npm run dev` and keep the Terminal open.

Do not run `install:features` on the plain Laravel/manual Inertia path. It belongs to this starter kit and may change in future versions.

## 5. Laravel + Livewire — no Fortify

Start from a separate plain Laravel project using section 2. Livewire uses PHP and Blade for interactive screens; it does not require React or Inertia.

### 5.1 Install Livewire

```bash
sail composer require livewire/livewire:^4.0
```

```bash
sail artisan make:livewire Counter --class
```

The `--class` option explicitly chooses a PHP class plus a separate Blade view, rather than Livewire 4's other component formats.

### 5.2 Add the component logic

Replace `app/Livewire/Counter.php` with:

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public int $count = 0;

    public function increment(): void
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

Replace `resources/views/livewire/counter.blade.php` with:

```blade
<div>
    <h1>Count: {{ $count }}</h1>
    <button wire:click="increment">Add one</button>
</div>
```

### 5.3 Create the page

Create `resources/views/home.blade.php`:

```blade
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Livewire demo</title>
        @livewireStyles
    </head>
    <body>
        <livewire:counter />
        @livewireScripts
    </body>
</html>
```

Replace `routes/web.php` in this fresh project with:

```php
<?php

use Illuminate\Support\Facades\Route;

Route::view('/', 'home');
```

Open **http://127.0.0.1:8000** and click **Add one**. The counter should change without a full page refresh.

This page needs neither Vite nor npm because it loads Livewire's assets directly and uses no bundled styles. Livewire includes Alpine; do not load another Alpine copy on this page. No Fortify is installed by this section.

## 6. phpMyAdmin

Use this with a Sail project that already has MySQL.

### 6.1 Add a service

Open the project's `compose.yaml`. Add this **under the existing `services:` key**, aligned with `mysql:` and `laravel.test:`. Do not create a second `services:` key.

```yaml
    phpmyadmin:
        image: phpmyadmin:latest
        ports:
            - '127.0.0.1:8080:80'
        environment:
            PMA_HOST: mysql
            PMA_PORT: 3306
        depends_on:
            - mysql
        networks:
            - sail
```

The host binding makes this browser tool available only from your Mac. The `sail` network must match the one generated in your existing file.

### 6.2 Start and log in

```bash
docker compose up -d phpmyadmin
```

Open **http://127.0.0.1:8080**.

| Field | Value for this guide |
|---|---|
| Username | `sail` |
| Password | `password` |
| Database | The value of `DB_DATABASE`, such as `my_laravel` |

Use your actual `.env` credentials if they differ. Select a database, select a table, then choose **Browse**.

phpMyAdmin uses the internal MySQL port `3306`, not your Mac's `3307`. An empty database needs `sail artisan migrate` to create tables.

## 7. Next.js + TypeScript

This is a separate Node.js project. It does not require Laravel or Inertia.

### 7.1 Create it inside a Node container

**Mac Terminal:**

```bash
cd ~/Developer
```

```bash
docker run --rm -it -v "$PWD:/workspace" -w /workspace node:24-bookworm-slim sh
```

**Inside the container:**

```bash
npx create-next-app@latest my-next
```

Choose the recommended defaults: TypeScript, App Router, and the offered standard styling/linting options. Decline optional changes you do not need. The current generator may phrase prompts differently.

```bash
exit
```

**Mac Terminal:**

```bash
cd ~/Developer/my-next
```

### 7.2 Add development Compose

Create `compose.yaml`:

```yaml
services:
  web:
    image: node:24-bookworm-slim
    working_dir: /app
    ports:
      - '127.0.0.1:3000:3000'
    volumes:
      - .:/app
      - next_modules:/app/node_modules
      - next_cache:/app/.next
    command: npm run dev -- --hostname 0.0.0.0

volumes:
  next_modules:
  next_cache:
```

The named volumes keep Linux dependencies and Next's cache separate from your Mac folders.

### 7.3 Install and run

```bash
docker compose run --rm web npm ci
```

```bash
docker compose up -d
```

Open **http://127.0.0.1:3000**. Edit `app/page.tsx` or `src/app/page.tsx`, depending on your generator choices.

```bash
docker compose logs --tail=50 web
```

Checkpoint: the page loads and an edited heading appears.

### 7.4 Check a production build locally

Stop the development server first so it does not share build output concurrently:

```bash
docker compose down
```

```bash
docker compose run --rm web npm run build
```

```bash
docker compose run --rm --service-ports web npm run start -- --hostname 0.0.0.0
```

Open port 3000. Press Control + C to stop. This checks a production build locally; it is not a hardened production container. Use Next's standalone-output Docker example when preparing a deployment.

## 8. Flutter web in Docker

Flutter includes Dart. This section builds a Linux Flutter toolchain from the official Flutter repository and runs a **browser app**. Docker does not provide an iOS simulator or macOS desktop build environment.

### 8.1 Prepare a folder

```bash
cd ~/Developer
```

```bash
mkdir my-flutter
```

```bash
cd my-flutter
```

### 8.2 Add the toolchain image

Create `Dockerfile`:

```dockerfile
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl git unzip xz-utils zip libglu1-mesa \
    && rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --uid 1000 developer
USER developer
WORKDIR /home/developer
RUN git clone --depth 1 --branch stable https://github.com/flutter/flutter.git
ENV PATH="/home/developer/flutter/bin:${PATH}"
RUN flutter config --no-analytics --enable-web
RUN flutter precache --web

WORKDIR /workspace
```

Create `compose.yaml`:

```yaml
services:
  flutter:
    build: .
    platform: linux/amd64
    working_dir: /workspace
    ports:
      - '127.0.0.1:8082:8080'
    volumes:
      - .:/workspace
    command: flutter run -d web-server --web-hostname 0.0.0.0 --web-port 8080
```

This example deliberately uses an x86 Linux toolchain for predictable SDK compatibility. Docker Desktop emulates it on Apple silicon, which is slower. `stable` moves over time; record the installed Flutter version and pin a release tag/commit for a long-lived project.

### 8.3 Create the app in a subfolder

```bash
docker compose build
```

```bash
docker compose run --rm flutter flutter create --platforms=web demo
```

The source lives under `demo/`, keeping generated app files separate from the Docker setup.

Change `working_dir` in `compose.yaml` to:

```yaml
    working_dir: /workspace/demo
```

### 8.4 Run and check

```bash
docker compose run --rm flutter flutter pub get
```

```bash
docker compose run --rm flutter flutter analyze
```

```bash
docker compose up -d
```

Open **http://127.0.0.1:8082**. Edit `demo/lib/main.dart`. This web-server workflow has more limited browser debugging than running Flutter with Chrome locally; restart the service and refresh if changes are not reflected:

```bash
docker compose restart flutter
```

Build static web files:

```bash
docker compose down
```

```bash
docker compose run --rm flutter flutter build web
```

Output: `demo/build/web/`. Keep `demo/pubspec.lock` in Git; ignore generated build and `.dart_tool` directories.

### 8.5 Android and iOS development

The image above contains no Android SDK. It cannot run `flutter build apk` or build iOS apps as written.

For native mobile development on your Mac:

1. Install the [Flutter SDK](https://docs.flutter.dev/install).
2. For Android, install Android Studio, its SDK tools, and an emulator using the [Android setup guide](https://docs.flutter.dev/platform-integration/android/setup).
3. For iOS, install Xcode, its platform runtimes, and required plugin tooling using the [iOS setup guide](https://docs.flutter.dev/platform-integration/ios/setup).
4. Open an emulator/simulator, then run these **on your Mac**, not in the Linux container:

```bash
cd ~/Developer/my-flutter/demo
```

```bash
flutter doctor
```

```bash
flutter create --platforms=android,ios .
```

```bash
flutter devices
```

```bash
flutter run
```

`flutter doctor` lists missing native requirements. Android builds can be containerized with an additional Android SDK/JDK image, but device/emulator access on Docker Desktop needs separate networking setup. iOS builds require macOS/Xcode; a Linux Docker container cannot replace them.

## 9. Dart command-line projects

Use this for Dart console programs, not Flutter screens.

### 9.1 Prepare the project

```bash
cd ~/Developer
```

```bash
mkdir my-dart
```

```bash
cd my-dart
```

Create `compose.yaml`:

```yaml
services:
  dart:
    image: dart:stable
    working_dir: /workspace
    volumes:
      - .:/workspace
```

### 9.2 Create, check, and run

```bash
docker compose run --rm dart dart create --template=console demo
```

Change `working_dir` to `/workspace/demo` in `compose.yaml`. Then:

```bash
docker compose run --rm dart dart pub get
```

```bash
docker compose run --rm dart dart analyze
```

```bash
docker compose run --rm dart dart run
```

Checkpoint: the generated program prints output. Edit `demo/bin/demo.dart` and rerun it.

## 10. C projects

This compiles and runs a Linux C executable inside Docker. The generated binary does not run directly on macOS.

### 10.1 Prepare the project

```bash
cd ~/Developer
```

```bash
mkdir my-c
```

```bash
cd my-c
```

Create `main.c`:

```c
#include <stdio.h>

int main(void) {
    puts("Hello from C inside Docker!");
    return 0;
}
```

Create `compose.yaml`:

```yaml
services:
  c:
    image: gcc:14
    working_dir: /app
    volumes:
      - .:/app
    command: sh -c 'mkdir -p build && gcc -std=c17 -Wall -Wextra -Werror main.c -o build/hello && ./build/hello'
```

### 10.2 Compile and run

```bash
docker compose run --rm c
```

The command makes an output directory, compiles with warnings enabled, and runs the binary only if compilation succeeds.

Checkpoint: `Hello from C inside Docker!` prints. Repeat the command after editing the code.

### 10.3 Optional self-contained image

Create `Dockerfile`:

```dockerfile
FROM gcc:14
WORKDIR /app
COPY main.c .
RUN gcc -std=c17 -Wall -Wextra -Werror main.c -o hello
CMD ["./hello"]
```

```bash
docker build -t my-c-app .
```

```bash
docker run --rm my-c-app
```

Rebuild after source changes. This simple example keeps the compiler in the final image; a smaller runtime image is a separate deployment optimization.

## 11. React Native with Expo

Docker runs the JavaScript toolchain and Metro development server. Your phone or simulator runs the native app. This is an Expo-managed React Native workflow, not a bare native build setup.

### 11.1 Create the app

**Mac Terminal:**

```bash
cd ~/Developer
```

```bash
docker run --rm -it -v "$PWD:/workspace" -w /workspace node:24-bookworm-slim sh
```

**Inside the container:**

```bash
npx create-expo-app@latest my-mobile
```

Use the default TypeScript template. If the generator prints a specific SDK recommendation for Expo Go, follow that recommendation for your installed Expo Go version.

```bash
exit
```

**Mac Terminal:**

```bash
cd ~/Developer/my-mobile
```

### 11.2 Add the development image

Create `Dockerfile`:

```dockerfile
FROM node:24-bookworm-slim
WORKDIR /app
RUN npm install -g @expo/ngrok
```

Create `compose.yaml`:

```yaml
services:
  expo:
    build: .
    working_dir: /app
    ports:
      - '127.0.0.1:8081:8081'
    volumes:
      - .:/app
      - expo_modules:/app/node_modules
    stdin_open: true
    tty: true
    command: npx expo start --tunnel

volumes:
  expo_modules:
```

The tunnel lets a physical phone reach Metro without relying on the Docker container's private network address. It uses a third-party service and creates a public development URL; stop it when you finish testing.

### 11.3 Install and run

```bash
docker compose build
```

```bash
docker compose run --rm expo npm ci
```

```bash
docker compose run --rm expo npx expo-doctor
```

```bash
docker compose up
```

Install **Expo Go** on your phone. Scan the displayed QR code using your phone's supported Expo Go flow. The phone and computer need internet access for this tunnel.

Checkpoint: the starter application opens on your phone. Edit the generated home screen under `app/` or `src/app/`, according to the template layout.

If Expo Go reports an SDK mismatch, use the template/SDK compatible with your Expo Go version or create a development build. Native modules unsupported by Expo Go need a development build.

### 11.4 Optional browser preview

Stop the running Expo process with **Control + C**. Install Expo's web dependencies:

```bash
docker compose run --rm expo npx expo install react-dom react-native-web @expo/metro-runtime
```

```bash
docker compose run --rm --service-ports expo npx expo start --web --localhost
```

Open **http://127.0.0.1:8081**. A container cannot open your Mac browser automatically. Web preview does not validate native-only features.

### 11.5 Native builds and simulators

Do not press `i` or `a` inside this Linux container expecting it to launch your Mac's simulator. Local iOS builds need Xcode on macOS; Android builds need the Android SDK/JDK. A physical phone with Expo Go avoids local native compilation for supported features.

Cloud development/store builds are a separate option through [Expo EAS Build](https://docs.expo.dev/build/introduction/). They require an Expo account and platform-specific signing/setup. The local Docker commands above do not build an iOS binary.

## 12. Everyday commands and recovery

### 12.1 Start and stop

Run commands from the relevant project's folder.

| Action | Laravel Sail | Other Compose projects |
|---|---|---|
| Start | `sail up -d` | `docker compose up -d` |
| Stop | `sail down` | `docker compose down` |
| Status | `sail ps` | `docker compose ps` |
| Logs | `sail logs --tail=50 laravel.test` | `docker compose logs --tail=50` |
| Rebuild | `sail up -d --build` | `docker compose up -d --build` |

Flutter and Expo may use attached `docker compose up` when interactive output is needed. C and Dart console examples use one-off `run --rm` commands.

### 12.2 Laravel daily workflow

```bash
cd ~/Developer/my-laravel
```

```bash
alias sail='./vendor/bin/sail'
```

```bash
sail up -d
```

For React/Vite development:

```bash
sail npm run dev
```

Keep that Terminal open. Laravel still opens at **http://127.0.0.1:8000**, not Vite's port 5173.

### 12.3 Browser says connection refused

```bash
sail ps
```

```bash
sail port laravel.test 80
```

For this guide, expect a host address ending in `:8000`. If it ends in `:80`, set `APP_PORT=8000` in `.env`, then:

```bash
sail up -d --force-recreate laravel.test
```

```bash
sail artisan config:clear
```

If the application container has exited, inspect:

```bash
sail logs --tail=50 laravel.test
```

Laravel may correctly log `0.0.0.0:80` inside its container while you visit `127.0.0.1:8000` on your Mac. `APP_URL` does not publish a Docker port; `APP_PORT` does so through the generated Compose configuration.

### 12.4 npm is not found

You may still be in the temporary PHP/Composer container. Exit it and use the full running Sail environment:

```bash
exit
```

In the project folder on your Mac:

```bash
./vendor/bin/sail npm --version
```

Do not install local Node.js just to fix this bootstrap issue.

### 12.5 Missing native binding / Linux ARM64 npm package

First try a clean install inside the correct container:

```bash
sail npm ci --include=optional
```

```bash
sail npm run build
```

`npm ci` replaces `node_modules` and respects the lock file. If the same native package is still missing, the lock file may omit the platform dependency. Keep a backup before regenerating it:

```bash
cp package-lock.json package-lock.backup.json
```

```bash
sail shell
```

**Inside the application container, at `/var/www/html`:**

```bash
pwd
```

Only after confirming this is your project directory:

```bash
rm -rf node_modules
```

```bash
rm package-lock.json
```

```bash
npm install --include=optional
```

```bash
exit
```

```bash
sail npm run build
```

This removes downloaded packages, not application source. Regenerating the lock file can update package versions; review its diff and commit the new lock only after the build works. Keep the backup outside Git or remove it after review.

### 12.6 Images or containers were deleted

With your project source, `.env`, `vendor/`, and Compose file intact:

```bash
sail up -d --build
```

If you also deleted the database volume, the old records are gone unless backed up. Recreate tables:

```bash
sail artisan migrate
```

Re-register test accounts only on the authentication starter-kit path. Do not regenerate an existing `APP_KEY` just because a container was deleted.

If `vendor/` is missing, Sail cannot run yet. Open the temporary PHP container from the project's parent folder, enter the project, and run:

```bash
composer install --no-scripts
```

```bash
php artisan package:discover
```

Exit, start Sail, and complete any remaining setup in the application container. Do not overwrite your existing `.env` with `.env.example`.

### 12.7 Port already allocated

Stop the other project using the same host port or assign a different host port. For example, change Laravel's `APP_PORT` to `8001` and `APP_URL` to `http://127.0.0.1:8001`, then recreate its application container.

MySQL, Redis, phpMyAdmin, and Vite can also conflict when multiple projects publish the same ports. Change the relevant host-side mapping or forwarding variable; internal service ports normally stay the same.

### 12.8 Keep data safe

`docker compose down` normally preserves named volumes. `docker compose down -v` removes them. Deleting volumes in Docker Desktop also removes their data. Images can be downloaded again; database records require a backup.

For a Sail MySQL database backup, open a project Terminal and run:

```bash
mkdir -p backups
```

```bash
sail exec -T mysql sh -c 'MYSQL_PWD="$MYSQL_PASSWORD" mysqldump --no-tablespaces -u "$MYSQL_USER" "$MYSQL_DATABASE"' > backups/database.sql
```

The password is read inside the container, not written literally into the command. Check that the command succeeded and the SQL file contains data. Keep backups outside Git. A backup is only proven when a restore has been tested against a separate disposable database.

## 13. Save projects and this guide on GitHub

### 13.1 Ignore local/generated files

Keep the framework's generated `.gitignore`. Add missing patterns appropriate to the project; do not replace useful existing rules:

```gitignore
.env
.env.*
!.env.example
node_modules/
vendor/
.next/
.expo/
.dart_tool/
build/
backups/
package-lock.backup.json
.DS_Store
```

For Flutter/Dart apps under `demo/`, keep the generated `demo/.gitignore` too.

Commit `compose.yaml`, Dockerfiles, source, `composer.lock`, `package-lock.json`, and application `pubspec.lock` files. Commit `.env.example` with non-secret sample values. Never commit your real `.env`, signing keys, database backups, or service-account credentials.

### 13.2 Save this guide

Create a repository on GitHub, such as `development-guides`. Upload this Markdown file through **Add file → Upload files**, then commit it. GitHub renders the contents and navigation automatically.

### 13.3 Optional Git commands

For a new project repository:

```bash
git init
```

```bash
git status
```

Confirm `.env`, dependencies, backups, and generated build files are ignored before staging:

```bash
git add .
```

```bash
git diff --cached --stat
```

```bash
git commit -m "Add Docker development setup"
```

Create an empty repository on GitHub and copy its real URL. Add that URL with `git remote add origin YOUR_REPOSITORY_URL`, then use GitHub's displayed push instructions. Replace the placeholder; do not paste it literally. GitHub authentication is separate from Docker.

### 13.4 Make future installs reproducible

Commands using `latest`, `stable`, or `dev-main` select moving versions. Record tool versions after setup, commit lock files, and pin Docker image tags/digests and Flutter release commits when maintaining a real project. Prefer `composer install` and `npm ci` when restoring an existing repository rather than creating a new project again.

## 14. Official references and validation notes

These recipes were assembled against the official documentation/source available on October 27, 2026. Future generators can change prompts and requirements. Source and configuration examples were reviewed, but full image builds and end-to-end runs for all stacks were not performed while authoring this guide. Treat each checkpoint as your local validation before continuing.

| Topic | Reference |
|---|---|
| Docker installation | [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/) |
| Laravel 13 | [Installation](https://laravel.com/docs/13.x/installation) |
| Sail | [Laravel Sail](https://laravel.com/docs/13.x/sail) |
| React starter kit | [Official source](https://github.com/laravel/react-starter-kit) |
| Laravel frontend tooling | [Laravel 13 package manifest](https://github.com/laravel/laravel/blob/13.x/package.json) |
| Inertia server | [Server-side setup](https://inertiajs.com/docs/v3/installation/server-side-setup) |
| Inertia client | [Client-side setup](https://inertiajs.com/docs/v3/installation/client-side-setup) |
| Livewire | [Installation](https://livewire.laravel.com/docs/4.x/installation), [components](https://livewire.laravel.com/docs/4.x/components) |
| Next.js | [Installation](https://nextjs.org/docs/app/getting-started/installation), [deployment](https://nextjs.org/docs/app/getting-started/deploying) |
| Flutter | [Install](https://docs.flutter.dev/install), [platform setup](https://docs.flutter.dev/platform-integration), [official SDK source](https://github.com/flutter/flutter) |
| Dart | [Official Docker image](https://hub.docker.com/_/dart) |
| C / GCC | [Official Docker image](https://hub.docker.com/_/gcc) |
| Expo | [Create a project](https://docs.expo.dev/get-started/create-a-project/), [CLI and tunnels](https://docs.expo.dev/more/expo-cli/) |
| React Native native setup | [Environment setup](https://reactnative.dev/docs/set-up-your-environment) |
| phpMyAdmin | [Official image](https://hub.docker.com/_/phpmyadmin) |

When asking for help, include the section/step, exact command, full error, and relevant container status. Do not post passwords or the contents of `.env`.
