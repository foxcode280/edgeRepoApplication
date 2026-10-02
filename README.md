# edgeRepoApplication

Software packages for Metronux UEM. This repository is a **read-only import
source**: UEM connects to it as a *GitHub Releases* repository, lists release
assets on **Sync**, and copies an installer into UEM storage when an admin
adds it as a package in Software Management. Deployments never download from
GitHub, so they keep working if GitHub (or the access token) is unavailable.

## How to publish a package

Installers are **release assets only**. Never commit installers to the git
tree (100 MB file limit, permanent history bloat) — `.gitignore` blocks them.

1. Create a release whose tag is `<app>-<version>`, e.g. `7zip-25.01`.
2. Attach the installer, e.g. `7z2501-x64.msi`.
3. Optionally attach `<asset>.metadata.json` per installer (e.g.
   `7z2501-x64.msi.metadata.json`, see below) to pre-fill the package form.
4. Publish the release (drafts are ignored by UEM).
5. In UEM → Repository → this connection → **Sync**.
6. In UEM → Software Management → **Add package** → pick the file.

Do not replace an asset under the same name. Publish a new release instead.
If an asset does change, UEM detects the new checksum on Sync and refuses to
import a file that no longer matches what was catalogued.

## Tag and asset naming

| Item    | Convention                         | Example            |
|---------|------------------------------------|--------------------|
| Tag     | `<app>-<version>` (lowercase app)  | `7zip-25.01`       |
| Asset   | vendor file name, arch in the name | `7z2501-x64.msi`   |

Architecture is detected from the file name (`x64`, `x86`, `arm64`); the admin
confirms it when adding the package.

## `<asset>.metadata.json` (optional)

One file per installer, named after the asset plus `.metadata.json`, so a
release can carry several installers (e.g. MSI and EXE). Pre-fills the Add
package form; the admin reviews everything before saving.
Field names match the UEM package format.

```json
{
  "name": "7-Zip",
  "version": "25.01",
  "publisher": "Igor Pavlov",
  "os": "windows",
  "arch": "x64",
  "format": "msi",
  "installCommand": "",
  "uninstallCommand": "",
  "detectionType": "msi_product_code",
  "detectionValue": "",
  "restartBehavior": "never",
  "installerFramework": "",
  "sha256": "<hex sha-256 of the installer>"
}
```

| Field                | Values                                                                 |
|----------------------|------------------------------------------------------------------------|
| `os`                 | `windows`, `linux`                                                     |
| `arch`               | `x64`, `x86`, `arm64`, `x64_x86`, `any`, `linux_x64`, `linux_arm64`     |
| `format`             | `msi`, `exe`, `msix`, `deb`, `rpm`, …                                   |
| `detectionType`      | `msi_product_code`, `registry`, `file`, `display_name`, `none`          |
| `restartBehavior`    | `never`, `if_required`, `always`                                        |
| `installerFramework` | EXE only: `nsis`, `inno`, `installshield`, `generic`, `auto`            |
| `sha256`             | If present and different from the asset's real hash, import is rejected |

Empty commands mean "use the default silent install/uninstall". In EXE
commands, `{path}` is replaced with the downloaded installer path.

## Access

UEM uses a **fine-grained personal access token** limited to this repository
with **Contents: Read-only** and **Metadata: Read-only**. The token is stored
encrypted in UEM, is never shown after saving, and is never sent to devices.
Rotate it by entering a new token on the repository connection.
