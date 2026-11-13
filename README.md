# sveltekit-supabase

SvelteKit app (`package.json` name `sveltekit-supabase`, version `0.0.1`) that signs a person in with a Supabase email magic link and reads or updates one row in a Supabase `profiles` table.

`src/routes/index.svelte` stores the current Supabase user and renders `Auth.svelte` or `Profile.svelte`.

## Prerequisites

- Node.js `>=14.13` and npm. The locked `@sveltejs/kit` release `1.0.0-next.240` declares that range, and `.npmrc` sets `engine-strict=true`.
- A Supabase project.
- The two variables read by `src/lib/supabaseClient.js`:
  - `VITE_SUPABASE_URL`
  - `VITE_SUPABASE_ANON_KEY`
- A `profiles` table the signed-in user can read and upsert. The client selects `username`, `website`, and `avatar_url` where `id` equals the user id, and upserts `id`, `username`, `website`, `avatar_url`, and `updated_at`.

`.gitignore` ignores `.env` and `.env.*`, and it leaves `.env.example` unignored. This repo has no `.env.example`. Put the variables in a root `.env` file:

```bash
VITE_SUPABASE_URL=<supabase project url>
VITE_SUPABASE_ANON_KEY=<supabase anon key>
```

Or export them in the shell before the dev server or the production build:

```bash
export VITE_SUPABASE_URL="<supabase project url>"
export VITE_SUPABASE_ANON_KEY="<supabase anon key>"
```

Vite inlines `import.meta.env.VITE_*` when `npm run dev` or `npm run build` starts. If `VITE_SUPABASE_URL` is unset, opening the app throws `supabaseUrl is required.`

## Install, build, run, and test

Install dependencies:

```bash
npm install
```

Start the dev server:

```bash
npm run dev
```

The server listens on port 3000 and prints `local: http://localhost:3000`. To open a browser tab, the `svelte-kit dev` CLI accepts `--open`:

```bash
npm run dev -- --open
```

Build:

```bash
npm run build
```

`svelte.config.js` uses `@sveltejs/adapter-auto`. The build completes. When no supported production environment is detected, adapter-auto prints that message and still exits successfully.

Preview the build (default port 3000):

```bash
npm run preview
```

Set the Supabase variables before `npm run build`. Preview serves the values that were inlined into that build.

`package.json` has no `test` script. `npm test` exits with `Missing script: "test"`.

`npm run package` runs `svelte-kit package`. That command exits with an error in this repo because `svelte2tsx` is not installed. The CLI also prints that `svelte2tsx` and `typescript` are required to generate type definitions.

`.gitpod.yml` runs `npm install && npm run build` on init, then `npm run dev`.

## Usage

1. Open `http://localhost:3000` with the dev server or preview running and the Supabase variables set.
2. On first load the page calls `supabase.auth.user()`. With no session it shows the sign-in form. The heading is `Supabase + Svelte`.
3. Enter an email and submit `Send magic link`. The form calls `supabase.auth.signIn({ email })`. On success the browser alerts `Check your email for the login link!`. While the request runs, the button label is `Loading` and the button is disabled.
4. `supabase.auth.onAuthStateChange` stores `session.user`. When that store value is set, the page shows the profile form.
5. The profile form loads the `profiles` row for that user id. Email comes from the session and the input is disabled. Name edits `username`. Website edits `website`. `avatar_url` is loaded and sent back on update. The form only has inputs for email, name, and website.
6. Submit `Update` to upsert the row. `updated_at` is `new Date()`, and the call uses `returning: 'minimal'`.
7. `Sign Out` calls `supabase.auth.signOut()`.

A select error whose status is `406` is ignored, so name and website stay empty. Any other error from sign-in, profile load, update, or sign-out is passed to `alert`.

## Layout

`jsconfig.json` maps `$lib` to `src/lib`.

```text
src/app.html              HTML shell. SvelteKit hydrates #svelte
src/global.d.ts           SvelteKit type reference
src/routes/index.svelte   Home route
src/lib/Auth.svelte       Magic-link sign-in form
src/lib/Profile.svelte    Profile load, update, and sign out
src/lib/sessionStore.js   Svelte store for the current user
src/lib/supabaseClient.js Supabase client
static/favicon.png        Favicon
svelte.config.js          adapter-auto and hydrate target #svelte
package.json              npm scripts
```

## License

This repository does not include a license file.
