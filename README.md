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

The `commit_stage` workflow supports the following inputs and secrets:

|name|default|description|
|----|----------|-----------|
|KEYCHAIN_PASSWORD|_required_|*SECRETS* The password for the created macos keychain|
|APPLE_CERTIFICATES_P12_BASE64_PASSWORD|_required_|*SECRETS* The certificate password required to install it in the keychain|
|APPLE_CERTIFICATES_P12_BASE64|_required_| *SECRETS* The base64 value of the p12 certificate file|
|WINPCAP_ROOT_DESTINATION|"." (current folder)| The folder to which the winpcap expected path will be appended. i.e. `${{inputs.WINPCAP_ROOT_DESTINATION}}/externals/3rdparty/winpcap` |
|ALTERNATIVE_BASHUTILS_PATH|"." (current folder)| The folder to which the bashUtils expected path will be appended. i.e. `${{inputs.ALTERNATIVE_BASHUTILS_PATH}}/scripts/bashUtils` |

In addition the `commit_stage` and `release_stage` workflows use the following repository secrets and variables, the variable marked as _required_ will make the job fail if empty. (How to set repository variables and secrets)

|name|default|description|
|----|-------|-----------|
|BUILD_CONFIG                   |_required_|The json string representing the build configuration cf "Build configuration settings"|
|LIB_NAME                   |_required_|The name of the library to push the csharp bindings 'la_avdecc_controller'|
|GTEST_FILTER               |_required_|The GTEST filter value '-INTEGRATION*'|
|MACOS_SIGNING_IDENTITY     |_required_|The signing identity to use during macos signing "MY_COMPANY (MY_TEAM_ID)"|
|BUILD_DIR                  |'_build'|The build directory used to generate build files|
|NUGET_PUBLISH_FEED_URL     |'https://nuget.pkg.github.com/$REPO_OWNER/index.json'|The feed url to publish the nuget package|
|INCLUDE_NUGET_LA_FEED      |'false'|Whether to register L-Acoustics nuget feed, should be set to 'false' if NUGET_PUBLISH_FEED_URL is set to L-Acoustics one|

### Build configuration
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
  },
  "required": ["runner_configs"],
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
  ]
}
```
As a single line string.
```text
'{"runner_configs":[{"labels":"ubuntu-22.04","arch":"x64"},{"labels":"ubuntu-22.04-arm","arch":"arm64"},{"labels":["self-hosted","arm64","macOS","my-runner-group"],"arch":"arm64"},{"labels":["self-hosted","arm64","macOS","my-runner-group"],"arch":"x64"},{"labels":["self-hosted","arm64","macOS","my-runner-group"],"arch":"universal"},{"labels":"windows-2022","arch":"x64"}]}'
```

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

Templates for the dotenv files can be found in [templates/workflows/dotenv](https://github.com/L-Acoustics/la-mw-gh-action/blob/main/templates/workflows/dotenv)

⚠️ **Do not Version the `secrets.env` files.**
