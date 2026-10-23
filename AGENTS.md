# Day.js agent guide

Day.js is a small, immutable date-time library with a Moment.js-style API.
Keep core small; optional functionality belongs in opt-in plugins.

## Source map

- `src/index.js`: `dayjs` factory, `Dayjs` class, locale registry, plugin loading.
- `src/constant.js`, `src/utils.js`: core constants and shared helpers.
- `src/plugin/<name>/index.js`: plugins, activated with `dayjs.extend(plugin)`.
- `src/locale/*.js`: locale data; `types/index.d.ts`, `types/plugin/`, and
  `types/locale/` contain the source declarations.
- `test/*.test.js`, `test/plugin/`, `test/locale/`, `test/issues/`: core,
  plugin, locale, and isolated regression tests.
- `build/index.js`, `build/rollup.config.js`: UMD core/plugin/locale builds;
  `build/esm.js`: ESM declaration copying and import rewriting.
- `docs/`: multilingual documentation, not runtime source.

`dev` is the default development branch. CI tests pushes/PRs to `dev`;
`master` triggers release automation.

## Commands (from the repository root)

```sh
npm install
npm run lint
TZ=UTC npm test
TZ=UTC npx jest test/plugin/isLeapYear.test.js --coverage=false
NODE_OPTIONS=--openssl-legacy-provider npm run build && npm run babel
node -e "const dayjs = require('./dayjs.min.js'); console.log(dayjs('2026-10-22').add(1, 'day').format('YYYY-MM-DD'))"
```

The focused Jest command is an example; substitute the relevant test path.
`npm test` runs four core timezone passes, four timezone-plugin passes, then
all Jest tests with 100% global line coverage enforced. Set `TZ=UTC` so the
last pass does not inherit the workstation timezone. A focused test does not
replace that matrix. The legacy Jest stack can print `fs.F_OK` deprecation
and Handlebars prototype-access warnings even when tests pass; check the
final pass counts and exit status before treating them as failures.
The release workflow uses the legacy OpenSSL provider;
`npm run build` also checks the 2.99 KB gzipped core limit. There is no dev
server or `npm run dev`. `npm run prettier` formats only `docs/**/*.md`,
not JavaScript.

## Conventions

- ES modules, extensionless relative imports, two spaces, single quotes, no
  semicolons or trailing commas; Airbnb-based ESLint is authoritative.
- Core operations return cloned/wrapped values. Mutation is intentional only
  in the opt-in `badMutable` plugin.
- Put plugin changes and tests in matching `src/plugin/<name>/index.js` and
  `test/plugin/<name>.test.js`; update `types/plugin/` for public API changes.
- Follow existing Moment.js comparisons where used. For clock-dependent tests,
  use `MockDate` with `beforeEach`/`afterEach` and always reset it.
- Prefer the closest existing test file; use `test/issues/` for isolated
  plugin-order regressions. Keep 100% line coverage and check size after core edits.
- Locale filenames are lowercase tags such as `pt-br.js`. Preserve their
  leading `// Language [tag]` comment: the build parses it into `locale.json`.
- Use semantic commit messages, as requested in `CONTRIBUTING.md`.

## Guardrails

- Edit declarations in `types/`; do not hand-edit generated `dayjs.min.js`,
  root `index.d.ts`, `locale/`, `plugin/`, `esm/`, `locale.json`, or `coverage/`.
- Do not edit `node_modules/` or casually change `patches/`; the tracked
  semantic-release patch belongs to the release dependency workflow.
- Avoid runtime dependencies and unrelated docs formatting: the tiny core
  and separately loaded plugins are defining constraints.
