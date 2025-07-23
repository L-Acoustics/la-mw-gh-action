# L-Acoustics Runner configuration action

This action is used to configure the runner for the following platforms:
- Macos
- Linux
- Windows


## Description
### Inputs
The following inputs are expected when calling the action:
|name|default|description|
|----|----------|-----------|
|keychain_password|_required_|(Macos runner) The password for the created macos keychain|
|apple_certificate_installer_p12_base64_password|_required_| The installer certificate password required to install it in the keychain|
|apple_certificate_app_p12_base64_password|_required_|The application certificate password required to install it in the keychain|
|apple_certificate_installer_p12_base64|_required_| The base64 value of the installer p12 certificate file|
|apple_certificate_app_p12_base64|_required_| The base64 value of the app p12 certificate file|
|setup_keychain|'false'| Whether to setup keychain for signing|
### Steps
The following describes what each steps do depending on the OS.
#### Linux configure
The steps installs the following package on a linux runner:
- wget
- ninja-build
- build-essential
- libcap-dev
- cmake (version > 3.31.5)
#### Windows configure
The steps isntall the following packages using chocolatey:
- grep

The steps also installs a legacy verison of winpcap.
The msvc dev commandline is initialized.

#### Macos configure
The steps install the following packages using brew:
- ninja
- wget
- bash
- grep
- cmake

The steps setup keychain if required.
