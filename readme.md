# Tamagui + Supabase + Solito + Next + Expo Monorepo

## ⚡️ Instant clone & deploy

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Flachlanhawthorne%2Ftamagui-supabase-todos-quickstart&root-directory=next&env=NEXT_PUBLIC_SUPABASE_URL,NEXT_PUBLIC_SUPABASE_ANON_KEY&envDescription=Supabase%20project%20URL%20and%20anon%20key&envLink=https%3A%2F%2Fapp.supabase.com%2F)

## 🔦 About

This monorepo is a starter todo app, built using Supabase + Expo + Next.js + Tamagui + Solito, adapted from the Tamagui starter (`npx create-tamagui-app@latest`). 

[Tamagui](https://tamagui.dev) is a universal design systems for React Native & Web built by [@natebirdman](https://twitter.com/natebirdman).

Many thanks to  [@FernandoTheRojo](https://twitter.com/fernandotherojo) for the Solito starter monorepo which this was initally forked from. Check out his [talk about using expo + next together at Next.js Conf 2021](https://www.youtube.com/watch?v=0lnbdRweJtA).

## 📦 Included packages

- `tamagui` for cross-platform views, themes and animations
- `solito` for cross-platform navigation
- `jotai` for state management
- `@supabase/supabase-js`
- `@supabase/supabase-auth-helpers`
- Expo SDK 44
- Next.js 12
- React Navigation 6

## 🗂 Folder layout

The main apps are:

- `expo` (native)
- `next` (web)

- `packages` shared packages across apps
  - `data-access` Supabase client, Jotai state management & user / auth context
  - `ui` includes your custom UI kit that will be optimized by Tamagui
  - `app` you'll be importing most files from `app/`
    - `features` (don't use a `screens` folder. organize by feature.)
    - `provider` (all the providers that wrap the app, and some no-ops for Web.)
    - `navigation` Next.js has a `pages/` folder. React Native doesn't. This folder contains navigation-related code for RN. You may use it for any navigation code, such as custom links.

You can add other folders inside of `packages/` if you know what you're doing and have a good reason to.

## 🏁 Start the app

- Set the environment variables for `next` and `expo`
- Install dependencies: `yarn`
- Next.js local dev: `yarn web`
  - Runs `yarn next`
- Expo local dev: `yarn native`
  - Runs `expo start`

## 💾 Setting up Supabase

- Create a new Supabase project

### Using GUI

- Create a new project 
- Navigate to the SQL Editor
- Run the following Todos migration

``` sql
create table todos (
  id bigint generated by default as identity primary key,
  user_id uuid references auth.users not null,
  task text check (char_length(task) > 3),
  is_complete boolean default false,
  inserted_at timestamp with time zone default timezone('utc'::text, now()) not null
);

alter table todos enable row level security;
alter publication supabase_realtime add table todos;

create policy "Individuals can create todos." on todos for
    insert with check (auth.uid() = user_id);
create policy "Individuals can view their own todos. " on todos for
    select using (auth.uid() = user_id);
create policy "Individuals can update their own todos." on todos for
    update using (auth.uid() = user_id);
create policy "Individuals can delete their own todos." on todos for
    delete using (auth.uid() = user_id);
```

### Using the CLI

- Install the CLI `npm install -g supabase`
- Init supabase in the project `supabase init`
- Login with an access token `supabase login`
- Link to your project `supabase link --project-ref [project-ref]`
- Link your Supabase database `supabase db remote set [database-connection-string with port 5432]`
- Create a new migration `supabase migration new todos`
- Copy the schema from `supabase-migrations/todos.sql` into the new migration at `supabase/migrations/[migration]_todos.sql`
- Run the todos migration `supabase db push`

## Developing

### Supabase

- Navigate to the `data-access` package.
- Set the `SUPABASE_REST_API_URL`, see .env.example for example
- Generate new types from your Supabase project by running `yarn supabase:types`


### Tamagui

We've added `packages/ui` to show an example of [building your own design system](https://tamagui.dev/docs/guides/design-systems).

You need to watch it to have changes propagate, we've added a root `watch` command you should run in a separate terminal alongside the apps:

```bash
yarn watch
```

If you want to see Tamagui extract, try running `yarn web:optimize` and put a debug pragma at the top of `packages/app/features/home.tsx`, like so:

```tsx
// debug
```

You'll see lots of output including the compiled HTML, CSS and all the steps it takes to get there.

## UI Kit

Note we're following the [design systems guide](https://tamagui.dev/docs/guides/design-systems) and creating our own package for components.

See `packages/ui` named `@my/ui` for how this works.

## 🆕 Add new dependencies

### Pure JS dependencies

If you're installing a JavaScript-only dependency that will be used across platforms, install it in `packages/app`:

```sh
cd packages/app
yarn add date-fns
cd ../..
yarn
```

### Native dependencies

If you're installing a library with any native code, you must install it in `expo`:

```sh
cd expo
yarn add react-native-reanimated
cd ..
yarn
```

You can also install the native library inside of `packages/app` if you want to get autoimport for that package inside of the `app` folder. However, you need to be careful and install the _exact_ same version in both packages. If the versions mismatch at all, you'll potentially get terrible bugs. This is a classic monorepo issue. I use `lerna-update-wizard` to help with this (you don't need to use Lerna to use that lib).