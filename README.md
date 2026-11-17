# PiCompose-Package-Hello

Example package `picompose-hello` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It prints a greeting and exists to test the build, signing and publishing pipeline end to end. It is not needed on a real device.

The repository is also meant as a template: copy it as a starting point for a package of your own.

## Install

Add the PiCompose APT repository (see [its documentation](https://github.com/florian-asche/PiCompose-Repository/blob/main/docs/apt_repository.md)), then:

```sh
sudo apt install picompose-hello
picompose-hello
```

## Layout

```
debian/
  control           package metadata, build dependencies, architecture
  changelog         the version history, written by the workflows
  rules             build instructions (plain dh)
  install           which files go where
  copyright         license
  source/format     3.0 (native): the version has no Debian revision
picompose-hello     the script installed to /usr/bin
```

## Using it as a template

1. Copy the repository and name it `PiCompose-Package-<Name>`.
2. Rename the source and binary package in `debian/control` and the name in `debian/changelog`. Package names start with `picompose-`.
3. Set `Architecture: all` for scripts, configuration and device tree overlays, and `any` for compiled content. This example lists `amd64 arm64` to keep the builds fast. `any` builds for `amd64`, `arm64` and `armhf`.
4. Replace the script and `debian/install` with your own files, and adjust `debian/copyright` and `LICENSE`.
5. Add the repository to `sources.yml` of the APT repository and add the secret `HUB_DISPATCH_TOKEN` to it.

## Versioning and changelog

The version of the package is the first entry of `debian/changelog`, which is part of the package (`apt changelog picompose-hello` shows the whole history). You do not edit it by hand, the workflows write it.

1. **Pull request:** Every pull request needs exactly one of the labels `major`, `minor` and `patch`. It decides how the version is raised when it is merged. Tick one box under "Version bump" in the description or set the label by hand. The check "Version label" fails until there is exactly one.
2. **Merge to `main`:** The workflow raises the version in `debian/changelog` by the highest label of the merged pull requests, adds the titles of the pull requests as changelog entry, commits `debian/changelog` to `main` (the commit "Version 1.3.0"), builds exactly that version and publishes it to the suite `trixie-nightly`. No tag is created.
3. **Release:** Start the workflow **Release** (Actions, Release, Run workflow). It tags the newest commit of `main` with `v<version>` of `debian/changelog`, creates the GitHub release and publishes that version to the suite `trixie`. Tags only exist for releases. A release created by hand in the GitHub UI is not published.
4. **Any other branch:** The package gets the version `<version>+dev<timestamp>.<sha>`. Nothing is published, the packages are kept as workflow artifacts for 14 days. Use a branch per change to try it on a device.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-hello_*.deb
```

The `.deb` files are written to the parent directory.

## Publishing

The workflow `.github/workflows/build.yml` calls the reusable workflow of the [PiCompose-Repository](https://github.com/florian-asche/PiCompose-Repository), which builds the package and tells the APT repository to publish it.

| Started by | Result | Version |
| --- | --- | --- |
| Push to `main` (merged pull request) | suite `trixie-nightly` | the next version, written to `debian/changelog` |
| Workflow "Release" | suite `trixie`, GitHub release `v<version>` | the version of `debian/changelog` |
| Push to another branch, pull request from a fork | artifact for 14 days | `<version>+dev<timestamp>.<sha>` |

The repository needs the secret `HUB_DISPATCH_TOKEN` and a branch protection of `main` that requires the check "Version label" and lets `github-actions[bot]` push, see the [DEVELOPMENT.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/DEVELOPMENT.md) of the APT repository.

## License

This project is released under the [MIT License](LICENSE).

The greeting is a test of the build and publishing pipeline.
