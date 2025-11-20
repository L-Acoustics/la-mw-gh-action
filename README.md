# L-Acoustics Github Actions and Workflows

This repository holds the composite actions and reusable workflows to be used to build test and release L-Acoustics libraries and Application.
It currently supports projects built using the L-Acoustics middleware scripts.

## Usage of reusable workflows

The reusable workflows should be called from a workflow defined within the target repository. (see [Reusable workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows) on github).
There are 2 workflows representing the stages of the CI:
- The `commit_stage` workflow executing the code validate, build, test activities.
- The `release_stage` workflow executing the code validate, build, (test) and artifact push activities.

Each workflow can be integrated into a repository using the following templates:
- [commit_stage.yml](https://github.com/L-Acoustics/la-mw-gh-action/blob/main/templates/workflows/commit_stage.yml)
- [release_stage.yml](https://github.com/L-Acoustics/la-mw-gh-action/blob/main/templates/workflows/release_stage.yml)

The workflow configuration is done with JSON object defined in [repository action variables](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables#defining-configuration-variables-for-multiple-workflows) and [repository secrets](https://docs.github.com/en/actions/concepts/security/secrets?versionId=free-pro-team%40latest&productId=actions) .
### Repository variables and secrets
The **required secrets** are :

|name|default|description|
|----|----------|-----------|
|KEYCHAIN_PASSWORD|_required_|*SECRETS* The password for the created macos keychain|
|APPLE_CERTIFICATES_P12_BASE64_PASSWORD|_required_|*SECRETS* The certificate password required to install it in the keychain|
|APPLE_CERTIFICATES_P12_BASE64|_required_| *SECRETS* The base64 value of the p12 certificate file|
|APP_PRIVATE_KEY|_optional_| *SERCRETS* The app private key for custom token generation (cf. [Use github app](#using-a-github-app-to-authenticate-private-submodules))|

The **required variables** are:
|name|default|description|
|----|-------|-----------|
|BUILD_CONFIG                   |_required_|The json string representing the build configuration cf "Build configuration settings"|

### Workflow configuration (json):
The **required parameters** are:
(cf build configuration schema for details)

|key|default|description|
|----|----------|-----------|
|runner_configs             |_required |The array containing the runner configuration to use.                           |
|macos_signing_identity     |_required_|The signing identity to use during macos signing e.g. "MY_COMPANY (MY_TEAM_ID)" |
|package_name               |_required_|The name of the package that will be pushed e.g. 'my_avdecc_controller'         |

The **optional parameters** are:
|name|default|description|
|----|-------|-----------|
|gtest_filter               |'*'                                                    |The GTEST filter value '-INTEGRATION*'|
|build_dir                  |'_build'                                               |The build directory used to generate build files|
|nuget_publish_feed_url     |'https://nuget.pkg.github.com/$REPO_OWNER/index.json'  |The feed url to publish the nuget package|
|include_nuget_la_feed      |'false'                                                |Whether to register L-Acoustics nuget feed, should be set to 'false' if NUGET_PUBLISH_FEED_URL is set to L-Acoustics one|
|winpcap_destination_root|"." (current folder)                                      | The folder to which the winpcap expected path will be appended. i.e. `$winpcap_destination_root/externals/3rdparty/winpcap` |
|alternative_bashutils_path|"." (current folder)                                    | The folder to which the bashUtils expected path will be appended. i.e. `$alternative_bashutils_path/scripts/bashUtils` |

### Build config schema
The build configuration is done through a json string read within the workflow. This Json string should be set as a repository variable.
The schema of the json object is shown bellow:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "runner_configs": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {

          "labels": {
            "oneOf": [
              { "type": "string" },
              {
                "type": "array",
                "items": { "type": "string" },
                "minItems": 1
              }
            ]
          },
          "arch": {
            "type": "string",
            "enum": ["x64", "arm64", "universal"]
          }
        },
        "required": ["labels", "arch"],
        "additionalProperties": false
      }
    },
    "gtest_filter": {
      "type": "string"
    },
    "macos_signing_identity":{
      "type": "string"
    },
    "package_name":{
      "type": "string"
    },
    "include_nuget_la_feed":{
      "type": "string",
      "enum": ["true","false"]
    },
    "nuget_publish_feed_url":{
      "type": "string"
    },
    "alternative_bashutils_path": {
      "type": "string"
    },
    "winpcap_destination_root": {
      "type": "string"
    },
    "build_directory": {
      "type": "string"
    },
  },
  "required": ["runner_configs", "macos_signing_identity", "package_name"],
  "additionalProperties": false
}
```
An example of a configuration:
```json
{
  "runner_configs": [
    {
      "labels": "ubuntu-22.04",
      "arch": "x64"
    },
    {
      "labels": "ubuntu-22.04-arm",
      "arch": "arm64"
    },
    {
      "labels": ["self-hosted","arm64","macOS","my-runner-group"],
      "arch": "arm64"
    },
    {
      "labels": ["self-hosted","arm64","macOS","my-runner-group"],
      "arch": "x64"
    },
    {
      "labels": ["self-hosted","arm64","macOS","my-runner-group"],
      "arch": "universal"
    },
    {
      "labels": "windows-2022",
      "arch": "x64"
   }
  ],
  "gtest_filter":"-MYTESTS*",
  "macos_signing_identity":"COM_COMPANY (QABCD123)",
  "package_name": "my_package_to_push"
}
```
As a single line string.
```text
"{\"runner_configs\":[{\"labels\":\"ubuntu-22.04\",\"arch\":\"x64\"},{\"labels\":\"ubuntu-22.04-arm\",\"arch\":\"arm64\"},{\"labels\":\"windows-2022\",\"arch\":\"x64\"},{\"labels\":\"macos-14\",\"arch\":\"x64\"},{\"labels\":\"macos-14\",\"arch\":\"arm64\"},{\"labels\":\"macos-14\",\"arch\":\"universal\"}],\"gtest_filter\":\"-MANUAL*\",\"macos_signing_identity\":\"L-ACOUSTICS (4WPJ48N2K4)\",\"package_name\":\"la_networkInterfaceHelper\"}"
```
#### Set build configuration programatically

For better readability and ease editing, it is recommended that you create a file holding the json config and parse the content as escaped json string to the github variable.
**Example using jq**
```json
# content of : $TARGET_REPOSITORY/.github/workflow/build_config.json

{
  "runner_configs": [
    {
      "labels": "ubuntu-22.04",
      "arch": "x64"
    },
    {
      "labels": "ubuntu-22.04-arm",
      "arch": "arm64"
    }
  ],
  "gtest_filter":"-MYTESTS*",
  "macos_signing_identity":"COM_COMPANY (QABCD123)",
  "package_name": "my_package_to_push"
}
```

```bash
cd $TARGET_REPOSITORY
jq '.| tostring' .github/workflows/build_config.json | gh variable set BUILD_CONFIG
```

### How to set repository secrets and variables
#### (Recommended) Github cli
The repository variables and secrets can be set through the github command line interface (see [setup github cli](https://docs.github.com/en/github-cli/github-cli/quickstart)). **Sufficient repository permissions** are required to set secrets and variables.
1. Create a file `variables.env` in `.github/workflows`.
2. Add required variables to the file. NB: All values should be surrounded by quotes.
3. Set the repository variables using `gh variable` command.
```bash
$> gh variable set -f .github/workflows/variables.env
```
4. Repeat the steps with secrets using the `gh secret` command.

Templates for the dotenv files can be found in [templates/workflows/dotenv](https://github.com/L-Acoustics/la-mw-gh-action/blob/main/templates/workflows/dotenv)

⚠️ **Do not Version the `secrets.env` files.**

### Using a GitHub App to authenticate private submodules 

If your repository includes private submodules, you can optionally use a GitHub App to authenticate the checkout operation instead of the default `GITHUB_TOKEN`.

- `USE_GH_APP`: repository **variable** (`"True"` or `"False"`). When set to `"True"`, the workflows will create an app installation token and use it for `actions/checkout` so private submodules can be accessed.
- `APP_ID`: repository **variable** (the numeric GitHub App ID) required when `USE_GH_APP` is `"True"`.
- `APP_PRIVATE_KEY`: repository **secret** (the GitHub App private key PEM content) required when `USE_GH_APP` is `"True"`.

Example entries for the dotenv templates (do not commit `secrets.env`):

`templates/workflows/dotenv/variables.env` (example additions)
```dotenv
APP_ID="12345"
USE_GH_APP="True"
```

`templates/workflows/dotenv/secrets.env` (example additions — keep this file secret)
```dotenv
APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----"
```

Behavior notes:
- When `USE_GH_APP` is `"True"`, the workflows call `actions/create-github-app-token` to exchange the app credentials for an installation token, and `actions/checkout` uses that token with `persist-credentials: false`.
- If `USE_GH_APP` is enabled but `APP_PRIVATE_KEY` or `APP_ID` are not provided the workflow will fail early with an explanatory message.
