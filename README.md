# L-Acoustics Github Actions and Workflows

This repository holds the composite actions and reusable workflows to be used to build test and release L-Acoustics libraries and Application.
It currently supports projects built using the L-Acoustics middleware scripts.

## Usage of reusable workflows

The reusable workflows should be called from a workflow defined within the target repository. (see [Reusable workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows) on github).
There are 2 workflows representing the stages of the CI:
- The `commit_stage` workflow executing the code validate, build, test activities.
- The `release_stage` workflow executing the code validate, build, (test) and artifact push activities.

Each workflow can be integrated into a repository using the following template
```yaml

name: Commit stage

on :
  push:
    branches: [ $MAIN_PUSH_BRANCHES ]
  pull_request:
    branches: [ $PR_BASE_BRANCHES ]
    paths-ignore:
      - 'Docker'
      - '**/README.md'
      - '**/LICENCE'

jobs:
  commit-stage:
    uses: L-Acoustics/la-mw-gh-action/.github/workflows/commit_stage.yml@$VERSION_REF
    secrets:
      GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
      KEYCHAIN_PASSWORD: ${{secrets.KEYCHAIN_PASSWORD}}
      APPLE_CERTIFICATES_P12_BASE64_PASSWORD: ${{secrets.APPLE_CERTIFICATES_P12_BASE64_PASSWORD}}
      APPLE_CERTIFICATES_P12_BASE64: ${{secrets.APPLE_CERTIFICATES_P12_BASE64}}
```
The `commit_stage` workflow supports the following inputs and secrets:

|name|default|description|
|----|----------|-----------|
|KEYCHAIN_PASSWORD|_required_|*SECRETS* The password for the created macos keychain|
|APPLE_CERTIFICATES_P12_BASE64_PASSWORD|_required_|*SECRETS* The certificate password required to install it in the keychain|
|APPLE_CERTIFICATES_P12_BASE64|_required_| *SECRETS* The base64 value of the p12 certificate file|
|WINPCAP_ROOT_DESTINATION|"." (current folder)| The folder to which the winpcap expected path will be appended. i.e. `${{inputs.WINPCAP_ROOT_DESTINATION}}/externals/3rdparty/winpcap` |
|ALTERNATIVE_BASHUTILS_PATH|"." (current folder)| The folder to which the bashUtils expected path will be appended. i.e. `${{inputs.ALTERNATIVE_BASHUTILS_PATH}}/scripts/bashUtils` |

In addition the `commit_stage` workflow **expects** the following repository secrets and variables. (How to set repository variables and secrets)

|name|default|description|
|----|-------|-----------|
|BUILD_DIR              |_required_|The build directory used to generate build files|
|RUNNER_LABELS_MACOS    |_required_|The label(s) to use for the macos runner. This should be a JSON string or a JSON array of string e.g. '"macos-14"' or '["self-hosted","my-label","my-label-2"]')|
|RUNNER_LABELS_WINDOWS  |_required_|The label(s) to use for the windows runner. This should be a JSON string or a JSON array of string e.g. '"windows-2022"' or '["self-hosted","my-label","my-label-2"]')|
|RUNNER_LABELS_LINUX    |_required_|The label(s) to use for the linux runner. This should be a JSON string or a JSON array of string e.g. '"ubuntu-22.04"' or '["self-hosted","my-label","my-label-2"]')|
|BUILD_MACOS            |_required_|Flag for executing the build for Macos e.g. 'true'|
|BUILD_LINUX            |_required_|Flag for executing the build on Linux e.g. 'true'|
|BUILD_WINDOWS          |_required_|Flag for executing the build on Windows e.g. 'false'|
|LIB_NAME               |_required_|The name of the library to push the csharp bindings 'la_avdecc_controller'|
|GTEST_FILTER           |_required_|The GTEST filter value '-INTEGRATION*'|
|MACOS_SIGNING_IDENTITY |_required_|The signing identity to use during macos signing "MY_COMPANY (MY_TEAM_ID)"|
|NUGET_PUBLISH_FEED_URL |_required_|The feed url to publish the nuget package|
|INCLUDE_NUGET_LA_FEED  |_required_|Whether to register L-Acoustics nuget feed, should be set to 'false' if NUGET_PUBLISH_FEED_URL is set to L-Acoustics one|

### How to set repository secrets and variables
#### (Recommended) Github cli
The repository variables and secrets can be set through the github command line interface (see [setup github cli](https://docs.github.com/en/github-cli/github-cli/quickstart)). **Sufficient repository permissions** are required to set secrets and variables.
1. Create a file `variables.env` in `.github/workflows`.
2. Add required variables to the file. NB: All values should be surrounded by single quotes.
3. Set the repository variables using `gh variable` command.
```bash
$> gh variable set -f .github/workflows/variables.env
```
4. Repeat the steps with secrets using the `gh secret` command.

⚠️ **Do not Version the `secrets.env` files.**

**Examples**:
```env
# .github/workflows/variables.env
RUNNER_LABELS_MACOS='["self-hosted","mw-label"]'
RUNNER_LABELS_WINDOWS='"windows-2022"'
RUNNER_LABELS_LINUX='"ubuntu-22.04"'

RELEASE_MACOS='true'
RELEASE_LINUX='false'
RELEASE_WINDOWS='true'

BUILD_MACOS='true'
BUILD_LINUX='false'
BUILD_WINDOWS='true'

LIB_NAME='my_awsome_lib'
BUILD_DIR='_build'
GTEST_FILTER='-INTEGRATION*'
MACOS_SIGNING_IDENTITY="COM.COMPANY (9999999)"

NUGET_PUBLISH_FEED_URL=
INCLUDE_NUGET_LA_FEED
```

```env
# ./secrets.env
KEYCHAIN_PASSWORD="mypass"
APPLE_CERTIFICATES_P12_BASE64_PASSWORD="SecurityP@ass"
APPLE_CERTIFICATES_P12_BASE64=12398103298140948120af123ac

```
#### Web user interface
The repository variables and secrets can be set through the github user interface given that you have **sufficient permissions**.







