# Standalone Hypit snapshot

This repository imports the working tree of
[`hypit-ai/hypit`](https://github.com/hypit-ai/hypit) at
`e4abc8b246e72600fc186d881fa831074e1060f5` into
`Planeshiffhter/openseedance`, without upstream Git history or tags.
Hypit.AI attribution, product names, licenses, workspace identities and runtime adapters
are retained. This is a source snapshot, not a new npm publication of `@hypit/*`.

## Native development, including Windows

| Tool | Exact repository selection |
| --- | --- |
| Node.js | `.node-version`: **24.14.1**; root engine minimum **22.15.0**. Use the pinned version for development and CI. |
| pnpm | `packageManager`: **10.33.0**; use the intact `pnpm-lock.yaml`. |
| Python | WhisperX and OpenCV `.python-version` files select **3.13**, with no exact patch pin. WhisperX and yt-dlp manifests allow **>=3.10,<3.14**; OpenCV requires **>=3.13,<3.14**. Use **3.13** for all local Python programs. |
| uv | No exact version pin; upstream documents **latest**. Use frozen sync with each runtime's `uv.lock`. |
| FFmpeg / FFprobe | No exact version pin; upstream requires a **recent stable** release on `PATH`. Windows CI uses `choco install ffmpeg --yes --no-progress`. |
| Chrome | Local HTML Provider recommends Chrome for Testing / Headless Shell **152.0.7928.2** (`packages/provider-html-local/package.json`). Puppeteer Core is **25.10.0** and `@puppeteer/browsers` is **3.2.2**. Explicit Runtime preparation downloads the selected version. |

Install Node 24.14.1, then use the following commands in Bash or PowerShell. If Corepack
is unavailable, `npm install --global pnpm@10.33.0` provides the same pinned package manager.

```sh
git clone https://github.com/Planeshiffhter/openseedance.git
cd openseedance
corepack enable
pnpm install --frozen-lockfile
pnpm check
pnpm test
npx skills add Planeshiffhter/openseedance -g
node bin/hypit.mjs --help
```

The Skill and executable install separately. `npx skills add` installs this repository's
Skill; it does not build or install a standalone CLI. To use the CLI from this checkout,
run `node <absolute-checkout-path>/bin/hypit.mjs <arguments>` (or `pnpm hypit <arguments>`
from the checkout). `npm install -g @hypit/hypit` selects the upstream npm distribution,
not this repository's source snapshot.

For local distribution packaging, run `npm run pack:distribution`. The tarball is written
under `dist/release/`; its independent `@hypit/*` dependencies still use ordinary npm
registry fetches. `npm run pack:release-candidate` also builds those independent packages;
`npm run check:release-candidate` installs and executes the complete candidate outside the
checkout, including browser preparation and an actual video render. No publish step is needed.
These commands preserve the upstream packaging scripts and public package names.

There are no Dockerfiles or Compose configurations in the imported snapshot. Local CLI,
Studio and Provider programs run natively; no container or prebuilt image is required.

## Projects, credentials and Runtime

Use a separate video-project directory whose `package.json` declares
`"hypit": { "project": true }`. Install that project's component dependencies normally.
The commands below assume `hypit` refers to the CLI installation you selected; when using
this checkout, replace `hypit` with `node <absolute-checkout-path>/bin/hypit.mjs`.

```sh
hypit paths
hypit runtime init
hypit runtime status
hypit auth --help
```

`runtime init` preserves existing configuration and creates a starter `hypit.runtime.json`.
Review its endpoints and bindings before preparation. `hypit runtime use <profile>` selects
another profile; `--runtime <profile>` selects one for a single invocation. Use actual
endpoint names from that profile in the commands below.

* **Hosted HypiHub:** the starter uses `hypihub.default`, with service base URL
  `https://hypit.ai/v1`. Run `hypit auth status hypihub.default`, then
  `hypit auth login hypihub.default` for its OAuth flow. Its default credential reference
  is the writable local-store key `hypihub.oauth`. This requires your own HypiHub account;
  the standalone repository supplies no account, credit or upstream organization secret.
* **Custom API or local model gateway:** select the appropriate Provider package, explicit
  endpoint URL and credential reference in the Runtime Profile. Use
  `hypit auth status <endpoint>` / `hypit auth login <endpoint>` for writable local-store
  API keys. For `@hypit/credential-store-env`, select that store and reference the exact
  environment-variable name you supply, e.g. `credentialRef("env", "IMAGE_API_KEY")`.
  There is no universal required `OPENAI_API_KEY` or `HYPIHUB_API_KEY`; the chosen Profile
  determines variable names and credentials. Local unauthenticated Providers need no key.
  A gateway must implement the selected Provider's protocol; a URL alone does not adapt an API.
* **Local media / rendering:** FFmpeg and FFprobe must be on `PATH`, or selected explicitly
  through the Profile's `ffmpegPath` / `ffprobePath`. Run
  `hypit programs prepare --endpoint html.local` to prepare Chrome, or
  `hypit runtime up --endpoint media.local --endpoint html.local` to prepare the selected
  programs and start the Worker. Inspect with `hypit doctor` and `hypit programs status`.
  Chrome is selected through `browserVersion` (plus optional `browserCacheDirectory` /
  `browserDownloadBaseUrl`) or a mutually exclusive explicit `chromePath`, not by an
  arbitrary browser environment variable. `pnpm install` intentionally downloads no browser.
* **Local WhisperX alignment:** select `@hypit/provider-whisperx-local` instead of the hosted
  alignment route, bind the alignment capability to that endpoint, and set
  `alignmentLanguages` before `hypit programs prepare --endpoint whisperx.local`.
  `hypit programs up --endpoint whisperx.local` starts its warm loopback service; default
  `http://127.0.0.1:8765`, model `small`, device `cpu`, compute `int8`, batch size `8`.
  Python dependencies use the preserved frozen lock and **WhisperX 3.8.6**. Model weights,
  alignment weights and NLTK data download only during explicit preparation. They are
  runtime state outside Git, and local service requests are path-based and loopback-only.

No environment variable or paid credential is required to install JavaScript dependencies,
type-check, run default offline tests or open Studio. Optional host/download settings:

| Variable | Use |
| --- | --- |
| `HYPIT_STATE_HOME` | Override the CLI's machine state directory. |
| `NODE_OPTIONS` | CI caps each Node heap at `--max-old-space-size=4096`. |
| `UV_PYTHON_INSTALL_MIRROR` | Optional Python distribution mirror. |
| `HF_ENDPOINT`, `HF_HOME`, `TORCH_HOME` | Optional model download route / caches for local inference. |
| `HYPIT_WHISPERX_PORT`, `HYPIT_WHISPERX_MODEL`, `HYPIT_WHISPERX_DEVICE`, `HYPIT_WHISPERX_COMPUTE`, `HYPIT_WHISPERX_BATCH_SIZE` | Normally supplied by the Managed Program from Profile settings. |
| `HYPIT_WHISPERX_ALIGNMENT_LANGUAGES`, `HYPIT_WHISPERX_MODEL_CACHE`, `HYPIT_WHISPERX_NLTK_DATA` | Local preparation resource selections, normally supplied by the Managed Program. |
| `HYPIT_WHISPERX_INPUT_ROOTS`, `HYPIT_WHISPERX_MAX_REQUEST_BYTES`, `HYPIT_WHISPERX_MAX_AUDIO_BYTES` | Optional service input-path and size bounds. |

See the [WhisperX service README](./packages/provider-whisperx-local/runtime/README.md)
for every service default and explicit trusted-proxy preparation. Keep credentials out of
Git and Profiles; use the selected credential store. Changing an existing Worker's inherited
environment requires an idle restart; inspect active work first.

Open Studio for an existing project Run:

```sh
hypit studio --run ./build.svrun
```

`hypit studio` requires `--run`; the default local port is **5179**. Select another port
with `--port <number>`. Studio can inspect sources without generation credentials; live
Builds still require the selected Runtime programs and service credentials. Stop Studio
with Ctrl+C; stop your idle Worker with `hypit runtime down` and helpers with
`hypit programs down` when finished.

## GitHub Actions

All six upstream workflow files remain present.

| Workflow | Standalone behavior |
| --- | --- |
| `ci.yml` | `main` pushes, pull requests and manual dispatch; upstream path ignores retained. Linux/Windows type-check and default tests, plus the reusable package build/consumer verification. Node follows `.node-version`; heap capped at 4 GiB; 20-minute test job timeout. |
| `npm-package.yml` | Called by CI or manually dispatched; package without publishing, then verify outside the checkout on Linux and Windows. 15/25-minute timeouts and 4 GiB heap cap. |
| `automation-check.yml` | Relevant `main` pushes and pull requests, plus manual dispatch; offline automation tests on Ubuntu. |
| `publish-npm.yml` | Job restricted to `hypit-ai/hypit`; skipped here even on manual dispatch or release events. Upstream npm OIDC authority is not reusable by this repository. |
| `sync-hypihub-docs.yml` | Job restricted to `hypit-ai/hypit`; upstream dispatch step additionally requires a nonempty `HYPIHUB_SYNC_TOKEN`. |
| `repository-analysis.yml` | Existing manual-only, upstream-repository job guard retained; `DEEPSEEK_API_KEY` is not needed here. |

Core CI uses standard GitHub-hosted runners and requires no upstream org secrets.
The active push workflows do not create tags, releases or npm publications. Packaging
artifacts are CI outputs, not committed dependencies. Optional live browser, OpenCV,
WhisperX and scale tests retain upstream opt-in behavior; see [Testing](./docs/guide/testing.md).

The native-frame evidence test fixture includes a silent 0.6-second audio track so the
container duration remains explicit across FFmpeg versions. Its four variable-rate video
timestamps and all half-open-range/grid assertions are unchanged; this corrects fixture
construction without weakening the test or changing production media code.

## Import validation

The import was checked on Linux with Node 24.14.1, pnpm 10.33.0 and FFmpeg 7.1.5:

* `pnpm install --frozen-lockfile` and `pnpm check`: passed.
* `pnpm test`: 1,157 tests, 1,135 passed, 22 optional skips, zero failures or cancellations.
* `node --test .github/automation/analysis.test.mjs`: all six passed.
* JSON, YAML and Python-manifest parsing, and actionlint 1.7.11: passed.
* `npm run pack:release-candidate` (including `pack:distribution`): passed; 58 tarballs.
* Studio example component build, source compilation, HTML response and compiled UI-module
  response: passed on the default local port 5179.
* `npm run check:release-candidate`: clean consumer installation, public type checking,
  CLI startup and source compilation passed. Live render verification was blocked when the
  cloud proxy denied `storage.googleapis.com`, the official managed Chrome download host.
  The render assertions remain enabled in CI; they were not skipped or replaced.

GitHub-hosted Linux/Windows results were not observable from the import session because
`api.github.com` returned HTTP 403. Check the repository's Actions tab for hosted outcomes.
These cloud network limitations do not establish a failure on GitHub-hosted runners.

The imported snapshot has no LFS objects, submodules or direct GitHub URL dependencies in
JavaScript/Python dependency declarations. Its largest tracked file is
`packages/provider-whisperx-local/runtime/uv.lock` at 635,384 bytes. Every upstream source
and media file is retained; the license, Git attributes, workspace manifest and all lockfiles
are byte-identical to upstream. Generated dependencies, virtual environments, build output,
model weights and browsers are excluded from Git.

## License

The original [LICENSE](./LICENSE) is retained byte for byte. It is **Apache License 2.0
with additional conditions**, not unmodified Apache-2.0: multi-tenant / hosted offerings
for third parties and commercial redistribution require written Hypit.AI authorization;
forking and publishing under the same license without commercial gain is permitted.
Preserve names, logos and copyright in surfaced CLI, reports, manifests and derived UI.
The license also includes contributor terms allowing producer license changes and
commercial use of contributions. The producer claims no rights in generated output;
third-party providers' terms may additionally apply. Read the complete license for its
definitions, permissions and exceptions. © 2026 Hypit.AI.
