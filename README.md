<!--
 * Copyright (C) 2025 Contributors to the Eclipse Foundation
 * 
 * This program and the accompanying materials are made available under the
 * terms of the Apache License, Version 2.0 which is available at
 * https://www.apache.org/licenses/LICENSE-2.0.
 * 
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
 * WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
 * License for the specific language governing permissions and limitations
 * under the License.
 * 
 * SPDX-License-Identifier: Apache-2.0
-->
# QUality EVEnt Engine (quevee) github action

This action accepts a set of URLs and metadata referring to release artifacts of various categories (like 'documentation' or 'testing'), and creates an output manifeste that summarizes project, release and artifact metadata and download URLs. The goal is to have a summary document with information about project release artifacts that are relevant for assessment of project quality aspects, and present them for evaluation and archival in quality assessment processes.

## quevee v1 vs v2

First generation quevee (v1 versions) were mostly a proof of concept, and so worked with minimal input (release URLs only, no metadata) producing a simple toml output file. While this served to illustrate the idea of generating a release software-quality artifact listing as part of a CI workflow, it quickly became apparent that a more fleshed out quality process needs more details beyond just artifact download URLs and some repository metadata. Therefore, quevee v2 was created, which is mostly a rewrite of the original quevee, with two important differences from a usage perspective:

1. Extended input data: in addition to release artifact download URLs, quevee v2 accepts additional information on each artifact, like it's name, a description, and a tags field.
2. Extended output data: to better accomodate the more complex input data, improve handling in scripts and other tooling, quevee v2 generates an extended json ouput in addition to the original toml manifest.

On the output side, quevee v2 behaves identically to the first-generation versions: it will still create the original toml manifest, with the same name, accessible via the same GitHub action ouput parameter. The new json output is available as an additional ouput (refer to documentation below). This is done to ensure backwards compatibility with the existing Eclipse Foundation quality badge process/tooling.

On the input side, quevee v2 expecs a slightly expanded format for artifact entries - in addition to the download URL, there should now be name, description and tags strings that provide additional information about an artifact. Also, quevee v2 attempts to construct a package URL ([PURL](https://github.com/package-url/purl-spec/blob/3fa4ebdd4fda67bd9adf058b480789021b26ccc1/PURL-SPECIFICATION.rst)) as for the release component as part of it's ouput, which requires a purl type and potentially a purl namespace. This information can be provided via new input parameters (refer to documentation below). For information about PURL types, refer to the [PURL specification](https://github.com/package-url/purl-spec/blob/3fa4ebdd4fda67bd9adf058b480789021b26ccc1/PURL-TYPES.rst).

NOTE: PURL creation in quevee v2 is somewhat basic at this time, and does not support qualifiers and subpaths. This might change in future quevee releases.

The following secions will address both v1 and v2 variants of quevee. Please be aware that v1 will be deprecated as soon as possible, so existing users should aim to move to v2 as soon as possible.

## Inputs

### quevee v1

This section describes input parameters and value formats for quevee v1 versions. This is deprecated, do not use for new setups!

#### `artifacts_documentation` (v1)

Comma-separated list of URLs of documentation artifacts.

#### `artifacts_licensing` (v1)

Comma-separated list of URLs of licensing information.

#### `artifacts_readme` (v1)

Comma-separated list of URLs of readme files.

#### `artifacts_requirements` (v1)

Comma-separated list of URLs of requirement files.

#### `artifacts_testing` (v1)

Comma-separated list of URLs of test results.

#### `release_url` (v1)

URL of the release this manifest refers to.

### quevee v2

This section describes input parameters and value formats for quevee v2 versions.

#### `artifacts_documentation` (v2)

Multi-line list of documentation artifacts, each line must be in the form `url|name|description|tags`

#### `artifacts_licensing` (v2)

Multi-line list of URLs containing licensing information, each line must be in the form `url|name|description|tags`

#### `artifacts_readme` (v2)

Multi-line list of URLs of readme files, each line must be in the form `url|name|description|tags`

#### `artifacts_requirements` (v2)

Multi-line list of URLs of requirement files, each line must be in the form `url|name|description|tags`

#### `artifacts_testing` (v2)

Multi-line list of URLs of test results, each line must be in the form `url|name|description|tags`

#### `json_filename` (v2)

File name of json manifest output

#### `log_level` (v2)

Quevee log level, given as an integer number; valid values are 0 (ERROR), 1 (WARN), 2 (INFO, default), or 3 (DEBUG)

#### `purl_namespace` (v2)

The package URL [(PURL) namespace](https://github.com/package-url/purl-spec/blob/3fa4ebdd4fda67bd9adf058b480789021b26ccc1/PURL-TYPES.rst) for the component; e.g. the user or organization in case of a github-based package

#### `purl_type` (v2)

Provides the package URL [(PURL) type](https://github.com/package-url/purl-spec/blob/3fa4ebdd4fda67bd9adf058b480789021b26ccc1/PURL-TYPES.rst); e.g. "cargo" for a Rust component (defaults to "undefined")

#### `release_url` (v2)

URL of the release this manifest refers to.

#### `toml_filename` (v2)

File name of toml manifest output

## Outputs

### quevee v1

This section describes outpu parameters and value formats for quevee v1 versions. This is deprecated, do not use for new setups!

#### `manifest_file` (v1)

Process artifacts manifest file name

### quevee v2

This section describes ouput parameters and value formats for quevee v2 versions.

#### `manifest_file` (v2)

Process artifacts manifest file name; identical to v1 toml output

#### `manifest_file_v2` (v2)

Extended process artifacts manifest file name; v2 json output

## Simple example

### quevee v1

Conceptually, this is how quevee v1 is used:

```yaml
- name: Collect quality artifacts
  uses: eclipse-dash/quevee@v0.4
  with:
    artifacts_readme: <readme-url>
    artifacts_requirements: <req-url1>,<req-url2>
```

This invocation will generate the following manifest structure:

```toml
[metadata]
<repository and release-specific information>

readme = [
    "readme-url",
]

requirements = [
    "req-url1", 
    "req-url2",
]
```

### quevee v2

Conceptually, this is how quevee v2 is used:

```yaml
- name: Collect quality artifacts
  uses: eclipse-dash/quevee@v2.0
  with:
    release_url: release-url
    purl_namespace: purl-namespace
    purl_type: purl-type
    artifacts_readme: |
      readme-artifact-url|readme-artifact-name|readme-artifact-description|readme-artifact-tags
    artifacts_requirements: |
      requirements-artifact1-url|requirements-artifact1-name|requirements-artifact1-description|requirements-artifact1-tags
      requirements-artifact2-url|||
```

This invocation will generate the following manifest structure:

```json
{
  "repo-url": github-url,
  "created": some-timestamp,
  "by-action": some-actionname,
  "git-hash": some-hash,
  "project": some-projectname,
  "components": [
    {
      "name": some-componentname,
      "tag": some-tag,
      "purl": some-purl,
      "releaseUrl": some-url,
      "evidence": [
        {
          "created": some-timestamp,
          "type": "README",
          "name": "readme-artifact-name",
          "description": "readme-artifact-description",
          "tsfAssertions": "readme-artifact-tags",
          "url": "readme-artifact-url"
        },
        {
          "created": some-timestamp,
          "type": "REQUIREMENTS",
          "name": "requirements-artifact1-name",
          "description": "requirements-artifact1-description",
          "tsfAssertions": "requirements-artifact1-tags",
          "url": "requirements-artifact1-url"
        },
        {
          "created": some-timestamp,
          "type": "REQUIREMENTS",
          "name": "",
          "description": "",
          "tsfAssertions": "",
          "url": "requirements-artifact1-url"
        },
      ]
    }
  ]
}
```

NOTE: As described above, quevee v2 will additionally generate the v1 toml output to ensure backwards compabitility.

## Realistic example (v2)

A more complete example, applied to the quevee repository:

```yaml
jobs:
  quality_artifacts_job:
    name: A job to collect quality artifacts
    runs-on: ubuntu-latest
    permissions: write-all
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Create release
        uses: softprops/action-gh-release@v2
        id: create_release
      - name: Show release URL
        run: |
          echo ${{ steps.create_release.outputs.url }}

      # Upload artifacts to release
      - name: Upload README to release
        uses: svenstaro/upload-release-action@v2
        id: upload_readme
        with:
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          file: README.md
          tag: ${{ github.ref }}
      - name: Upload LICENSE to release
        uses: svenstaro/upload-release-action@v2
        id: upload_license
        with:
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          file: LICENSE
          tag: ${{ github.ref }}

      - name: Collect quality artifacts
        uses: eclipse-dash/quevee@v2
        id: quevee
        with:
          release_url: ${{ steps.create_release.outputs.url }}
          purl_namespace: ${{ github.repository_owner }}
          purl_type: github
          artifacts_readme: |
            ${{ steps.upload_readme.outputs.browser_download_url }}|Project README|README documentation including usage, build and contribution instructions|
          artifacts_requirements: |
            ${{ steps.upload_license.outputs.browser_download_url }}|Project License|Project License text|
            https://yet.another.org/example/artifact.bz2|Example Artifact|Another dummy example artifact link|example_tag

      - name: Upload quality manifest to release
        uses: svenstaro/upload-release-action@v2
        id: upload_manifest
        with:
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          file: ${{ steps.quevee.outputs.manifest_file }}
          tag: ${{ github.ref }}
      - name: Store quality manifest as workflow artifact
        uses: actions/upload-artifact@v4
        id: store_manifest
        with:
          name: quality-artifacts-manifest
          path: ${{ steps.quevee.outputs.manifest_file }}
          
      - name: Upload extended quality manifest to release
        uses: svenstaro/upload-release-action@v2
        id: upload_manifest_v2
        with:
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          file: ${{ steps.quevee.outputs.manifest_file_v2 }}
          tag: ${{ github.ref }}
      - name: Store extended quality manifest as workflow artifact
        uses: actions/upload-artifact@v4
        id: store_manifest_v2
        with:
          name: quality-artifacts-manifest_v2
          path: ${{ steps.quevee.outputs.manifest_file_v2 }}
````

This invocation will generate the following manifest structure (test run on the quevee repository):

```json
{
  "repo-url": "https://github.com/EclipseDash/quevee",
  "created": "2025-04-16T10:47:24Z",
  "by-action": "quevee",
  "git-hash": "65c58eef5accdb6f61e80bbd77a2cc6e9b275221",
  "project": "EclipseDash",
  "components": [
    {
      "name": "quevee",
      "tag": "v2.0.0",
      "purl": "pkg:github/EclipseDash/quevee@2.0.0",
      "releaseUrl": "https://github.com/EclipseDash/quevee/releases/tag/v2.0.0",
      "evidence": [
        {
          "created": "2025-04-16T10:47:24Z",
          "type": "REQUIREMENTS",
          "name": "Project License",
          "description": "Project License text",
          "tags": "",
          "url": "https://github.com/EclipseDash/quevee/releases/download/v2.0.0/LICENSE"
        },
        {
          "created": "2025-04-16T10:47:24Z",
          "type": "REQUIREMENTS",
          "name": "Example Artifact",
          "description": "Another dummy example artifact link",
          "tags": "example_tag",
          "url": "https://yet.another.org/example/artifact.bz2"
        },
        {
          "created": "2025-04-16T10:47:24Z",
          "type": "README",
          "name": "Project README",
          "description": "README documentation including usage, build and contribution instructions",
          "tags": "",
          "url": "https://github.com/EclipseDash/quevee/releases/download/v2.0.0/README.md"
        }
      ]
    }
  ]
}
```

## Development & Testing

In the `./tools` folder you'll find test runner scripts that you can use to try and test the quevee actions locally, for development and experimentation. You will have to set the appropriate environment variables with the content you need for testing, e.g. `INPUT_ARTIFACTS_README` needs to contain URL(s) that should be passed to the quevee script as the content of `artifacts_readme`.
