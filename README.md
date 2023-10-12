# generate-sbom-yaml

This GitHub Action uses the GitHub dependency graph to generate a SBOM in SPDX format and an OpenContext artifact YAML definition for the SBOM.

**NOTE**: This GitHub action does not keep the GitHub dependency graph updated.
## Usage

In general you will need to do the following to make use of this GitHub action:

- Make sure your GitHub dependency graph is up to date. See [GitHub](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/using-the-dependency-submission-api) for more information.
- Make sure the workflow that uses this GitHub action is triggered by the `release` event

### Input

- `upload_yaml` - Upload OpenContext SBOM artifact YAML to release? Possible values are `true` or `false`. If true the OpenContext YAML will be uploaded as a release artifact alongside the SBOM. If false the OpenContext YAML will be uploaded as a workspace artifact.

### Outputs

- `sbom_path` - The path of the SBOM generated.
- `sbom_url` - The release URL where the SBOM can be downloaded from.
- `artifact_path` - The path of the OpenContext SBOM artifact YAML.
- `artifact_url` - The release URL of the OpenContext SBOM artifact YAML or the text `WORKSPACE_ARTIFACT` if the input `upload_yaml` is `false`

### Generate and upload both the SBOM and OpenContext YAML definition to the release

```
name: Upload SBOM and artifact yaml on release
on:
  release:
    types: [created]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - id: sbom
        uses: opencontextinc/sbom-artifact-context@fixAction
        with:
          upload_yaml: true
      - name: Check outputs
        run: |
          # i.e. SBOM PATH = /home/runner/work/retail-app/retail-app/spdx-68bb4376-acf0-4ed7-9eb4-926892a3950a.spdx.json
          echo "SBOM PATH = ${{ steps.sbom.outputs.sbom_path }}"
          # i.e. SBOM URL = https://github.com/scatter-ly/retail-app/releases/download/v0.0.8/retail-app-f2797e3-sbom-spdx.json
          echo "SBOM URL = ${{ steps.sbom.outputs.sbom_url }}"
          # i.e. Artifact YAML PATH = /home/runner/work/retail-app/retail-app/oc-artifact-yaml.tgz
          echo "Artifact YAML PATH = ${{ steps.sbom.outputs.artifact_path }}"
          # i.e. Artifact YAML URL = https://github.com/opencontextinc/retail-app/releases/download/v0.0.8/oc-artifact-yaml.tgz
          echo "Artifact YAML URL = ${{ steps.sbom.outputs.artifact_url }}"
```

### Generate both the SBOM and the OpenContext YAML definition and upload the SBOM to the release and upload the YAML definition to the workspace

```
name: Upload SBOM and generate artifact yaml on release
on:
  release:
    types: [created]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - id: sbom
        uses: opencontextinc/sbom-artifact-context@fixAction
        with:
          upload_yaml: false
      - name: Check outputs
        run: |
          # i.e. SBOM PATH = /home/runner/work/retail-app/retail-app/spdx-68bb4376-acf0-4ed7-9eb4-926892a3950a.spdx.json
          echo "SBOM PATH = ${{ steps.sbom.outputs.sbom_path }}"
          # i.e. SBOM URL = https://github.com/scatter-ly/retail-app/releases/download/v0.0.8/retail-app-f2797e3-sbom-spdx.json
          echo "SBOM URL = ${{ steps.sbom.outputs.sbom_url }}"
          # i.e. Artifact YAML PATH = /home/runner/work/retail-app/retail-app/oc-artifact-yaml.tgz
          echo "Artifact YAML PATH = ${{ steps.sbom.outputs.artifact_path }}"
          # i.e. Artifact YAML URL = WORKSPACE_ARTIFACT
          echo "Artifact YAML URL = ${{ steps.sbom.outputs.artifact_url }}"
```

### Using the YAML file with OpenContext
In order for OpenContext to process these files you will need to do the following:

- **De-duplicate the files generated**. If the same file is found multiple times then there will be a conflict, and the artifact or bot described in the YAML file will not appear in the catalog. This GitHub action will generate the same filename for the same artifact and bot. As long as the YAML generated is not modified after generation, you should be able to just replace the old file with a new one if a new one is generated.
- **Save these files to a locations known to OpenContext**:
  - Commit these files to a repository that is for OpenContext YAML. For example, see the [opencontext repository](https://github.com/scatter-ly/opencontext) or the [scatter.ly repository](https://github.com/scatter-ly/scatter.ly) in our demo GitHub organization [scatter-ly](https://github.com/scatter-ly).
  - Concatenate the contents of the files generated into an `oc-catalog.yaml` file and commit it to the root of your current repository. For example, see the [retail-app repository](https://github.com/scatter-ly/retail-app) in [scatter-ly](https://github.com/scatter-ly), our demo GitHub organization.
  - Upload these files to the catalog files location for your tenant. See our [docs](https://docs.opencontext.com/docs/getting-started/client-portal#catalog-files) for more information.
