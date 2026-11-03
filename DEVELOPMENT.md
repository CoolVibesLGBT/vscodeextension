# Developing and publishing CoolVibes for VS Code


Install the web workspace's dependencies with Yarn, then install the extension's dependencies:

```bash
yarn install --frozen-lockfile
```

Build the latest web source into a validated, installable VSIX:

```bash
./deploy.sh
# Or select a specific web workspace:
./deploy.sh /absolute/path/to/web
```

The build discovers the sibling web workspace automatically. You can also set `COOLVIBES_WEB_ROOT`. It refreshes brand artwork, validates HTML/CSS asset references and packages the extension. Install the resulting `.vsix` using **Extensions → Install from VSIX…**.

To build and publish the same package to the **CoolVibes** Marketplace publisher:

```bash
./deploy.sh --publish
# Or:
./deploy.sh --publish /absolute/path/to/web
```

Publishing automatically uses `VSCE_PAT` when supplied, or Microsoft Entra ID through an installed Azure CLI. If a Microsoft login is needed, the script opens the sign-in flow; no Azure subscription is required. The Microsoft identity must have publishing access to the CoolVibes publisher.

Without Azure CLI, the script opens the Marketplace publisher page and reveals the VSIX for browser upload. Select the extension, choose **Update**, and upload the package. This path exits with status 2 to indicate that publication is still pending.

For token-free CLI publishing on macOS:

```bash
brew install azure-cli
COOLVIBES_PUBLISH_AUTH=azure ./deploy.sh --publish
```

Use `COOLVIBES_PUBLISH_AUTH=browser` to select browser upload, or `COOLVIBES_PUBLISH_AUTH=pat` to use an existing vsce login. Keep credentials out of source files. Increase the version in `package.json` before publishing another release.


## Verify asset integration

```bash
yarn run check
yarn run check:assets
yarn run check:assets:browser
```

Browser checks require Chrome and the web workspace’s Playwright dependencies.

## GitHub Actions Trusted Publishing

`.github/workflows/publish.yml` builds the extension and publishes the saved VSIX with OIDC. No Marketplace PAT or Azure login is used. Create this policy under the **CoolVibes** Marketplace publisher:

- Policy name: `Release workflow`
- Provider: GitHub Actions
- Repository owner: `CoolVibesLGBT`
- Repository: `vscodeextension` (not a URL)
- Workflow: `publish.yml`
- Environment: `production`

Create the `production` environment in the extension repository. If `CoolVibesLGBT/web` is private, add a read-only SSH deploy key to that repository and store its private key as the extension repository/environment secret `WEB_DEPLOY_KEY`. Public web repositories need no key. This is source checkout access, not a Marketplace publishing credential.

Push the workflow and extension changes to GitHub, and push the desired web changes to `CoolVibesLGBT/web` first: local uncommitted changes are not included by Actions. Increase `package.json` version before publishing a version already present on Marketplace. Open **Actions → Release workflow → Run workflow**, select the extension branch and enter the web branch/tag/commit (default `main`). Use a commit SHA for reproducible web sources. The workflow retains the VSIX as an artifact even if publishing fails. It runs manually and does not publish on every push.

The publishing step installs the latest vsce because the pinned packaging CLI 3.9.2 does not support `--oidc`. For private extension repositories, run the workflow within 48 hours of creating the policy to activate it.
