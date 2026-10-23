# EZ-Tree on Tofu

EZ-Tree is Daniel Greenheck's browser-based procedural tree generator. It lets users adjust tree parameters, choose presets, and export generated trees as 3D models, images, or saved settings. Rendering and generation run in the visitor's browser; the upstream name, logo, favicon, and author credits are preserved.

## Run on Tofu

- Framework preset: `vite`.
- Project root: repository root. The app's HTML source is `src/app/index.html`; `vite.app.config.js` configures that source root.
- Package manager: npm, using the existing `package-lock.json`.
- Install: `npm ci`.
- Build: `npm run build` (builds the tree library first, then the application).
- Output directory: `dist/`, with `dist/index.html` served at `/`.
- Local production preview: `npm run preview -- --host 127.0.0.1 --port 47923 --strictPort`.
- Required environment variables: none.
- Database: none.
- Accounts, API keys, and Tofu credentials: none.
- Runtime: a browser with WebGL support. There is no application server or cloud persistence; exports are downloaded locally.

The upstream Docker files remain available but are not used for this gallery build. Choose the Vite preset above.

## Adaptations and reasons

1. Added the conventional `build` script to run `build:lib` followed by `build:app`. The production app imports the built library, so both steps are required on a clean checkout.
2. Removed the upstream Google Analytics scripts from the HTML. Copies do not send usage to the upstream analytics account.
3. Removed the paid-asset promotion footer from the settings panel. The original author credits and social links remain.
4. Materialized all 40 Git LFS texture pointers as their exact upstream bytes and disabled LFS filtering for those texture paths on the `tofu` branch. Each file's SHA-256 and size were checked against its upstream pointer. This makes GitHub source archives complete without requiring Git LFS or a build-time asset download. The textures total 47,934,738 bytes.
5. Added this document to describe the gallery build and its verification limits.

The browser still fetches public presentation assets from Google Fonts and cdnjs (Font Awesome), and a Draco decoder from Google's `www.gstatic.com` CDN. These require no account or key. The upstream social links and sharing metadata remain. The application is not an offline bundle.

## Local verification

Verified with Node.js `v22.22.3` and npm `10.9.8`:

```sh
npm --prefix /private/tmp/gallery-batch2/ez-tree ci --no-audit --no-fund
npm --prefix /private/tmp/gallery-batch2/ez-tree run build
npm --prefix /private/tmp/gallery-batch2/ez-tree run preview -- --host 127.0.0.1 --port 47923 --strictPort
curl --fail --silent --show-error --output /dev/null --write-out 'HTTP %{http_code}\n' http://127.0.0.1:47923/
```

Installation and both build steps exited successfully. The production preview returned HTTP 200 for `/`, `/styles.css`, `/favicon.ico`, `/images/logo.png`, `/models/rock1.glb`, `/textures/leaves/ash.png`, and `/textures/bark/Bark001_1K-JPG/Bark001_1K-JPG_Color.jpg`. The page contained the expected EZ-Tree title and no Google Analytics loader. All 40 built textures matched the materialized source bytes. The preview process was stopped after verification.

The build printed non-fatal warnings about API Extractor's bundled TypeScript version, `eval` in two Three.js addon modules processed by Rollup, and the application's JavaScript chunk size (about 786 kB before gzip). These checks do not verify interactive WebGL rendering or export behavior in a real browser. No Tofu deployment was performed.

## Provenance and licensing

- Upstream: https://github.com/dgreenheck/ez-tree
- Upstream branch: `main`.
- Base upstream commit: `dcf309bd86bd521083d9c70f01f2de45fdc7c457`.
- Gallery fork: https://github.com/tofu-gallery/ez-tree
- Adaptation branch: `tofu`.
- Verified gallery tag: `pin/2026-10-23`.
- Application license: MIT, copyright Daniel Greenheck; the root `LICENSE` remains unchanged.
- Bark textures: CC0 from ambientCG; `src/app/public/textures/LICENSE.md` remains unchanged.
- Other bundled assets retain the upstream notices and licensing. Dependency licenses remain with their packages.

The fork's `main` branch mirrors the upstream source and receives no adaptation commits. When updating the `tofu` branch, materialize and verify any new LFS textures before repeating the build and creating a new pin.
