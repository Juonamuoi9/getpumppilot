set -e

git clone https://github.com/Juonamuoi9/getpumppilot.git
cd getpumppilot
git switch -c perf/quick-wins

cat > perf.patch <<'PATCH'
diff --git a/package.json b/package.json
index 0000000..0000000 100644
--- a/package.json
+++ b/package.json
@@ -1,12 +1,14 @@
 {
   "name": "tanstack_start_ts",
   "private": true,
   "sideEffects": false,
   "type": "module",
   "scripts": {
+    "build:core": "vite build",
+    "ci:full": "bun run sync:route-seo && bun run gen:asset-og && bun run gen:social-hashes && bun run gen:sitemap && bun run test:security && bun run test:seo:social && vite build && bun run ping:indexing",
     "dev": "vite dev",
     "build": "bun run sync:route-seo && bun run gen:asset-og && bun run gen:social-hashes && bun run gen:sitemap && bun run test:security && bun run test:seo:social && vite build && bun run ping:indexing",
     "preview": "vite preview",
     "lint": "eslint .",
     "format": "prettier --write .",
     "cap:sync": "cap sync",
diff --git a/tsconfig.json b/tsconfig.json
index 0000000..0000000 100644
--- a/tsconfig.json
+++ b/tsconfig.json
@@ -1,15 +1,17 @@
 {
   "include": ["src/**/*.ts", "src/**/*.tsx", "vite.config.ts", "eslint.config.js"],
   "compilerOptions": {
     "target": "ES2022",
     "jsx": "react-jsx",
     "module": "ESNext",
     "lib": ["ES2022", "DOM", "DOM.Iterable"],
     "types": ["vite/client"],
 
     /* Bundler mode */
     "moduleResolution": "Bundler",
     "allowImportingTsExtensions": true,
     "verbatimModuleSyntax": false,
     "noEmit": true,
+    "incremental": true,
+    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo",
 
     /* Linting */
     "skipLibCheck": true,
     "strict": true,
     "noUnusedLocals": false,
PATCH

git apply perf.patch

mkdir -p .github/workflows
cat > .github/workflows/perf-ci.yml <<'YAML'
name: perf-ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: 'npm'

      - name: Cache bun + node deps
        uses: actions/cache@v4
        with:
          path: |
            ~/.bun
            ~/.cache/bun
            node_modules
          key: ${{ runner.os }}-deps-${{ hashFiles('**/package-lock.json','**/bun.lock') }}
          restore-keys: |
            ${{ runner.os }}-deps-

      - name: Install dependencies
        run: npm install

      - name: Build core
        run: npm run build:core
YAML

npm install
npm run build:core

git add package.json tsconfig.json .github/workflows/perf-ci.yml
git commit -m "perf: add fast build, TS incremental cache, and CI cache"

echo "Local changes committed successfully."
echo "Next:"
echo "git push --set-upstream origin perf/quick-wins"
echo "gh pr create --title \"perf: quick wins\" --body \"Add a fast local build path, incremental TypeScript caching, and CI cache.\" --base main"
