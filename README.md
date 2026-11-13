# sveltekit-supabase

SvelteKit app that signs a user in to [Supabase](https://supabase.com) with an email magic link, then loads and updates that user's row in a `profiles` table.

The only page is `src/routes/index.svelte`. With no session it shows the sign-in form. With a session it shows the profile form.

## Prerequisites

- Node.js 14.13 or newer and npm. `@sveltejs/kit` `1.0.0-next.240` (the version in `package-lock.json`) sets `"node": ">=14.13"`. `.npmrc` sets `engine-strict=true`, so npm refuses to install when a dependency's engines field does not match the running Node version.
- A Supabase project.
- These two environment variables, read in `src/lib/supabaseClient.js`:

  - `VITE_SUPABASE_URL`
  - `VITE_SUPABASE_ANON_KEY`

- A `profiles` table the signed-in user can read and upsert. The app uses these columns: `id` (set to the auth user id), `username`, `website`, `avatar_url`, and `updated_at`.

Vite inlines `VITE_` variables at dev-server start and at build time. Put them in a `.env` file in the project root, or export them in the shell before `npm run dev` and `npm run build`. `.gitignore` ignores `.env` and `.env.*`. A build without `VITE_SUPABASE_URL` still completes, and the served page responds with 500 and `supabaseUrl is required.`

```bash
VITE_SUPABASE_URL=<your Supabase project URL>
VITE_SUPABASE_ANON_KEY=<your Supabase anon key>
```

## Install, build, run, and test

Install dependencies:

```bash
npm install
```

Start the dev server:

```bash
npm run dev
```

Choose a port:

```bash
npm run dev -- --port 3001
```

Create a production build. This uses `@sveltejs/adapter-auto` from `svelte.config.js`. Outside a supported host, the build finishes and prints that it could not detect a supported production environment.

```bash
npm run build
```

Serve that build locally. The default port is 3000.

```bash
npm run preview
```

`package.json` has no test script.

`npm run package` is `svelte-kit package`. In this repo it exits with an error: `svelte2tsx` and `typescript` are not installed, and the command cannot find package `svelte2tsx`.

`.gitpod.yml` runs `npm install && npm run build`, then `npm run dev`.

## Usage

1. Open the dev server URL. The page heading is "Supabase + Svelte".
2. Enter an email and submit **Send magic link**. The form calls `supabase.auth.signIn({ email })`. On success it alerts `Check your email for the login link!`. On failure it alerts `error.error_description` or `error.message`.
3. After the session exists, `src/routes/index.svelte` stores `supabase.auth.user()` and updates that store from `supabase.auth.onAuthStateChange`. The profile form replaces the sign-in form.
4. The profile form shows the session email (disabled), a name field (`username`), and a website field. Submitting **Update** upserts `{ id, username, website, avatar_url, updated_at }` into `profiles` with `returning: 'minimal'`. `updated_at` is `new Date()`. `avatar_url` is loaded with the profile and sent back on update. The form has no control for it.
5. While loading the profile, an error whose status is 406 is ignored and the name and website fields stay empty. Any other error is alerted.
6. **Sign Out** calls `supabase.auth.signOut()`. The page shows the profile form while the user store is set.

## Layout

```
src/app.html              HTML shell. The app hydrates #svelte.
src/global.d.ts           SvelteKit type reference
src/routes/index.svelte   session check and the only route
src/lib/Auth.svelte       magic-link form
src/lib/Profile.svelte    profile load, update, and sign-out
src/lib/sessionStore.js   Svelte store for the current user
src/lib/supabaseClient.js Supabase client
src/lib is imported as $lib (see jsconfig.json)
static/favicon.png
svelte.config.js          @sveltejs/adapter-auto, target #svelte
package.json              scripts and dependencies
package-lock.json
.npmrc                    engine-strict=true
.gitpod.yml               install, build, then dev
```

Dependencies: `svelte` 3, `@sveltejs/kit` and `@sveltejs/adapter-auto` at the `next` dist-tags pinned by the lockfile, `@supabase/supabase-js` 1.29.3, and `vite` 2.7.13.

## License

This repository has no license file.
