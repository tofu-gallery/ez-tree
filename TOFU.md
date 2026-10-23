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
4. Materialized all 40 Git LFS texture pointers as their exact upstream bytes and disabled LFS filtering for those texture paths on the `tofu` branch. Each file's SHA-256 and size were checked against its upstream pointer. This makes GitHub source archives complete without requiring Git LFS or a build-time asset download. The textures totalled 47,934,738 bytes at that step (see 5).
5. Re-encoded the 33 bark maps under `src/app/public/textures/bark/` so the GitHub source archive of the pin stays under Tofu's 50,000,000-byte import limit: the archive of `pin/2026-10-23` was 52,105,798 bytes. These are the Color, NormalGL and Roughness maps of all 11 bark sets. ambientCG ships them as JPEG quality 100; they are now JPEG quality 97 with full-resolution color (4:4:4, no chroma subsampling) and optimized Huffman tables, written with Pillow 9.4. The resolution is unchanged (1024x1024; the Bark008 maps are 512x1024), and the embedded color profiles of the seven files that carry one were kept byte for byte. Sizes: Color maps 16,500,066 to 9,915,488 bytes; NormalGL maps 23,112,594 to 16,298,757 bytes; Roughness maps 6,779,393 to 5,236,839 bytes; all 33 bark maps 46,392,053 to 31,451,084 bytes. Quality 97 is the highest single quality that meets the gallery's 40,000,000-byte archive target; at quality 98 the archive would still be about 41.3 MB. No texture was deleted, because the app uses every one: `src/app/textures.js` loads the Color, NormalGL and Roughness map of each bark set listed in `BarkType`, and the bark Type menu offers all 11 sets; `src/app/ground.js` loads the three ground textures, and `LeafType` covers the four leaf textures. The ground and leaf textures are unchanged.
6. Added this document to describe the gallery build and its verification limits.

The browser still fetches public presentation assets from Google Fonts and cdnjs (Font Awesome), and a Draco decoder from Google's `www.gstatic.com` CDN. These require no account or key. The upstream social links and sharing metadata remain. The application is not an offline bundle.

## Local verification

Verified for `pin/2026-10-23.1` with Node.js `v22.22.3` and npm `10.9.8`, from the repository root:

```sh
npm ci --no-audit --no-fund
npm run build
python3 -I -m http.server 47931 --bind 127.0.0.1 --directory dist
curl --silent --output /dev/null --write-out 'HTTP %{http_code}\n' http://127.0.0.1:47931/
```

Installation and both build steps exited successfully, `/` returned HTTP 200, and the textures in `dist/textures/` are identical to the source files. In a real browser (Chromium) the page rendered the scene with a generated tree. Switching the bark Type menu through all 11 bark sets loaded all 33 bark maps with HTTP 200, the trunk's bark changed visibly, and the console showed no errors and no warnings (the app logs a warning when a bark map fails to load). The server was stopped after verification.

The build printed non-fatal warnings about API Extractor's bundled TypeScript version, `eval` in two Three.js addon modules processed by Rollup, and the application's JavaScript chunk size. Export behavior was not exercised. No Tofu deployment was performed.

The earlier pin, `pin/2026-10-23`, was verified the same way with `npm run preview` and HTTP checks only, before the bark maps were re-encoded.

## Provenance and licensing

- Upstream: https://github.com/dgreenheck/ez-tree
- Upstream branch: `main`.
- Base upstream commit: `dcf309bd86bd521083d9c70f01f2de45fdc7c457`.
- Gallery fork: https://github.com/tofu-gallery/ez-tree
- Adaptation branch: `tofu`.
- Verified gallery tag: `pin/2026-10-23.1`. The earlier tag `pin/2026-10-23` still points to the commit before the bark maps were re-encoded.
- Application license: MIT, copyright Daniel Greenheck; the root `LICENSE` remains unchanged.
- Bark textures: CC0 from ambientCG, re-encoded as described in adaptation 5; `src/app/public/textures/LICENSE.md` remains unchanged.
- Other bundled assets retain the upstream notices and licensing. Dependency licenses remain with their packages.

The fork's `main` branch mirrors the upstream source and receives no adaptation commits. When updating the `tofu` branch, materialize and verify any new LFS textures, re-encode new or changed bark maps the same way (adaptation 5), and check the size of the source archive (`git archive --format=zip HEAD | wc -c`) before repeating the build and creating a new pin.
