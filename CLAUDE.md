# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Vue 2 + LESS single-page app that renders multiple resume templates from `resume/data.yml` and exports them to PDF via Puppeteer. The app serves a gallery of previews on the home page and renders individual resumes at `/resume/:id`.

## Commands

```bash
npm install                              # install dependencies
npm run dev                              # start dev server on :8080 with HMR
npm run export                           # start dev server + Puppeteer export all resumes to pdf/
npm run preview                          # convert PDFs in pdf/ to PNG previews
npm run docs                             # webpack production build to docs/
npm run lint                             # ESLint on src/ and scripts/
npm run lint:fix                         # auto-fix lint issues
npm test                                 # run all e2e test suites sequentially
npm run test:export                      # test PDF export pipeline
npm run test:preview                     # test preview generation
npm run test:docs                        # test docs build
npm run test:e2e                         # dev + TestCafe e2e tests
```

Test a single e2e file directly:
```bash
npx testcafe chromium test/e2e/resumes.js
```

Run a single mocha script:
```bash
npx mocha test/scripts/export.js
```

## Architecture

**Entry**: `src/main.js` → `src/App.vue` → `src/router/index.js` routes to `Home` (`/`) or `Resume` (`/resume/:resumeid`).

**Templates**: Each resume is a Vue component in `src/resumes/`. They all follow the pattern from `src/resumes/template.vue`:

```js
import { getVueOptions } from './options';
export default Vue.component('NAME', getVueOptions('NAME'));
```

New templates must be imported in `src/resumes/resumes.js` to be registered with vue-router and picked up by the export script.

**Data**: `resume/data.yml` is parsed at runtime (js-yaml) and passed to each resume component. Replace `resume/id.jpg` for the profile picture.

**Languages**: i18n strings live in `src/lang/*.js`. Each file exports a translation map.

**Export pipeline** (`scripts/export.js`): waits for the dev server to respond on port 8080, then launches Puppeteer, navigates to `/#/resume/<name>` for every `.vue` file in `src/resumes/`, and saves an A4 PDF to `pdf/`. Box shadows require the chrome-shadow-fixer workaround (see DEVELOPER.md).

**Build output**: `npm run docs` compiles to `docs/` configured for GitHub Pages deployment at `/best-resume-ever/`.

**Tests**: TestCafe e2e tests live in `test/e2e/`. Mocha helper scripts live in `test/scripts/`. Tests start the dev server themselves where needed.
