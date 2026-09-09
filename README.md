# Shootero — WebGL production build

Unity WebGL build of Shootero wired to the **GameBull production API**
(`https://api.g-b.store`).

Served by GitHub Pages from the repository root:
<https://alishehroz-ideo.github.io/shootero-prod/>

**This is the live URL handed to the GameBull admin panel. Never push a staging build here.**
Before committing, check the built `index.html` — its `apiBase` default must read
`https://api.g-b.store` — and grep the BUILD FILES for the staging host:

```
grep -rl "api.staging" index.html Build TemplateData StreamingAssets
```

Scoped to those paths on purpose. Grepping the whole folder matches this README,
which names the staging host twice, so the check appears to fail every time and
stops being worth running.

The staging build lives at
[alishehroz-ideo/shootero-staging](https://github.com/alishehroz-ideo/shootero-staging).

## What is in here

Unity's build output, verbatim, plus four files that live in this repo and **not** in
Unity's output:

| file | why it matters |
|---|---|
| `.nojekyll` | without it Pages runs Jekyll, which drops any path beginning with an underscore |
| `.gitattributes` | `* -text` — stops Git rewriting line endings inside the build artifacts |
| `.gitignore` | scratch files that must never be committed alongside a build |
| `README.md` | this file |

Unity rebuilds this folder from scratch, so those four are **deleted from the working tree by
every build**. Read `git status` for DELETIONS before committing, and restore them:

```
git checkout HEAD -- .nojekyll .gitattributes .gitignore README.md
```

`git add -A` stages their removal without comment, and the symptom afterwards is a Pages site
that mysteriously stops serving parts of the build.

## Build

Both halves of the environment switch together — the C# `GAMEBULL_PRODUCTION` define picks the
URL the *game* talks to, and the WebGL template's own `apiBase` picks the one the *loading
screen* queries for the per-game icon. Changing one alone is silent: the icon lookup 404s and
falls back to the static icon, which looks exactly like "the admin uploaded nothing".

With Unity **closed** (an open editor holds the project lock):

```
Unity.exe -quit -batchmode -nographics -projectPath <proj> \
  -executeMethod GameBull.EditorTools.GameBullBuildSetup.UseProduction

Unity.exe -quit -batchmode -nographics -buildTarget WebGL -projectPath <proj> \
  -executeMethod WebGLBuilder.BuildWebGLDeploy
```

Output goes to `builds/webgl-prod`. **That folder name is load-bearing** — Unity names its
artifacts after it, so it is what produces `webgl-prod.loader.js`, the exact filename
`index.html` asks for. Build to a differently-named folder and the page 404s on its own loader.

Use `BuildWebGLDeploy`, never `BuildWebGL`. The localhost entry point forces
`compressionFormat = Disabled` and `exceptionSupport = FullWithStacktrace`, both persisted to
`ProjectSettings.asset` so the change outlives the build. On this game that is 117 MB instead
of 61 MB, with the wasm going 8.2 MB → 46 MB.

## Size

Brotli, with Unity's JS decompression fallback on — GitHub Pages sends no `Content-Encoding`
of its own, so the fallback is what makes the compressed files load at all.

| | |
|---|---|
| `webgl-prod.data.unityweb` | 54.7 MB |
| `webgl-prod.wasm.unityweb` | 8.2 MB |
| total | ~61 MB |

Compare the wasm against the last build before pushing — the loading screen was tuned around
that number.
