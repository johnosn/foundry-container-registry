# Local Copy Changelog

## Comparison Basis

- **Reference:** [`multi-cloud-image-support`](https://github.com/johnosn/foundry-container-registry/tree/multi-cloud-image-support), commit `7458a121dd398532d515d136bc35bd68f32cae2d`.
- **Local copy:** compared with a temporary shallow clone of the reference branch because this workspace copy has no `.git` metadata.
- **Result:** 16 shared files have content changes, 30 files are local-only, and no files from the reference branch are missing locally.
- Build output, dependencies, Playwright reports, and local validation artifacts were excluded. The local `e2e/.env` was not read and no secret values are included here.

## Registry Synchronization

- **Adds multi-cloud and cloud-specific image sources** for Falcon Sensor, Container Sensor, Image Assessment, and Kubernetes Admission Controller images. Regional variants are labeled `(Regional)`; other existing image sources remain available.
- **Retains valid older tags and sorts tags by semantic version.** Invalid/non-semver sensor tags are filtered. The README documents the supported older-tag behavior for multi-cloud Falcon Sensor and regional Container Sensor images.
- **Adds optional image build timestamps.** The sync function reads each image's creation date and returns it as `buildDate` when available; failures to read a date do not prevent that tag from being returned.
- **Handles unavailable registry variants more gracefully.** A missing or empty variant does not prevent synchronization when its counterpart is available. If no usable variant exists for a component, synchronization fails before replacing the stored image list.
- **Stops serializing registry credentials** in collection image records and removes the associated fields from the collection schema.
- **Uses Foundry function identity** (`Request.FnID`) to decide when to write results to the Foundry collection. A local invocation returns the fetched data without writing, even when an access token is present.
- **Checks collection-write responses** for empty responses and API payload errors instead of treating every successful HTTP call as a successful write.

**Files:** `functions/syncimages/main.go`, `functions/syncimages/falcon/falcon.go`, `functions/syncimages/registry/registry.go`, `functions/syncimages/go.mod`, `functions/syncimages/go.sum`, `collections/images.json`.

## Registry UI

- Groups multi-cloud and regional records under one normalized image name, labels each registry type, and sorts groups alphabetically without case sensitivity.
- Filters dated builds older than 18 calendar months, retains undated or invalid-date tags, and recalculates the latest tag and digest from the remaining tags. An empty result displays a “No recent images” state with a sync action.
- Expanding an image shows separate details and tag tables for each available registry variant, including architecture, build date, and digest.
- Places image paths beside “Latest tag” in each registry section when both variants exist. For a single registry, keeps the latest tag and image path alongside the image name and description.
- Adds responsive layout rules for registry details and updates the image model to include optional build-date metadata.
- Updates the UI dependencies, including PatternFly React Table, React, TypeScript, Webpack, and related loaders/plugins.

**Files:** `ui/pages/src/app/Dashboard/ImageItem.tsx`, `ui/pages/src/app/Dashboard/ImageList.tsx`, `ui/pages/src/app/app.css`, `ui/pages/src/app/types/Image.ts`, `ui/pages/package.json`, `ui/pages/package-lock.json`.

## Automation and App Configuration

- Adds a daily scheduled image-sync workflow at 07:38 UTC and prevents overlapping runs.
- Registers the workflow in `manifest.yml`, updates Foundry app/capability identifiers, and adds ignore rules for local UI/E2E dependencies, environment files, test output, and package locks.
- Adds a root `.gitignore` entry for the local E2E environment file.

**Files:** `workflows/container-registry-daily-image-sync.yml`, `manifest.yml`, `.gitignore`.

## Documentation

- Documents multi-cloud and regional image availability, older-tag handling, variant fallback behavior, and the requirement to sync after deploying an updated function.
- Clarifies that collection writes depend on Foundry function identity, not merely the presence of an access token.

**Files:** `README.md`, `docs/DEVELOPER.md`.

## Local-Only Additions

- **Scheduled workflow:** `workflows/container-registry-daily-image-sync.yml`.
- **End-to-end test setup:** `e2e/.env.sample`, `e2e/.gitignore`, `e2e/README.md`, `e2e/package.json`, `e2e/package-lock.json`, `e2e/playwright.config.ts`, `e2e/tsconfig.json`, `e2e/tests/auth.setup.ts`, and `e2e/tests/registry.spec.ts`. A local `e2e/.env` also exists but was intentionally not inspected.
- **Backend tests:** `functions/syncimages/main_test.go` and `functions/syncimages/falcon/falcon_test.go`.
- **UI tests:** `ui/pages/tests/imageGroups.test.js` and `ui/pages/tests/imageItem.test.js`.
- **JavaScript companion files:** `ui/pages/src/index.js`, `ui/pages/src/app/index.js`, `ui/pages/src/app/Dashboard/Dashboard.js`, `ui/pages/src/app/Dashboard/ImageItem.js`, `ui/pages/src/app/Dashboard/ImageList.js`, `ui/pages/src/app/types/Image.js`, `ui/pages/src/app/types/ImageCollectionResponse.js`, `ui/pages/src/app/utils/imageGroups.js`, and `ui/pages/src/app/utils/useDocumentTitle.js`.
- **Local tooling/configuration:** `.foundryignore`, root `package-lock.json`, `ui/pages/.github/modernize/code-migration/.gitignore`, and `ui/pages/.tsupgrader/runtime-validation/{baseline-result.json,eval-plan.json,postupgrade-result.json}`.

## Changed-File Inventory

The content-diff inventory contains the files listed in the four sections above, plus these dependency/configuration changes: `functions/syncimages/go.mod` upgrades `foundry-fn-go` from `v0.23.2` to `v0.24.0`; UI dependencies and lockfiles were refreshed as noted above. The remaining local-only additions are listed separately to distinguish new local files from edits to files shared with the branch.