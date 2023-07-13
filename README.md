# sbom-artifact-context

GitHub Action to generate SBOM in SPDX format and generate artifact context YAML.

* Does not keep the GitHub dependency graph updated
* Should be triggered by on_release
  * Generates SBOM and uploads it as a release artifact
  * Generates artifact context YAML which user can choose whether to upload as release artifact.
