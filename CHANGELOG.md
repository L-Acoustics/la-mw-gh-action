# Changelog

All notable changes to the project will be documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project follows incremental versioning.

## [v3]
### Added
- Mechanism to use custom token to during checkout, the token can pe specified or generated using Github App authentication.
- Workflow variable `release` to declare whether the current build is a release build or not (default to false).
- Allow execution of the validate job if the PR comes from a forked repository.
- Upagre `action/checkout` to v5.
- Use `action/create-github-app-token` v2
- Documentation to use github app to authenticate private submodules.
- By default `pre-workflow-checks`, `build-test` only run when called within a L-Acoustics repository, can be override with given variable.
### Removed
- Workflow variable `WINPCAP_ROOT_DESTINATION` and ÀLTERNATIVE_BASHUTILS_PATH`, replaced by build configuration parameters.
- Workflow variable `release_linux`, `release_macos`, `release_windows`, replaced by single `release` variable.
- Old prepare matrix job in release stage.
