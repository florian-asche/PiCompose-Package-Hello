# PiCompose-Package-Software-Hello

Example package `picompose-software-hello` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It prints a greeting and exists to test the build, signing and publishing pipeline end to end. It is not needed on a real device.

The repository is also meant as a template: copy it as a starting point for a package of your own.

## Install

Add the PiCompose APT repository (see [its documentation](https://github.com/florian-asche/PiCompose-Repository/blob/main/docs/apt_repository.md)), then:

```sh
sudo apt install picompose-software-hello
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

The version of the package is the first entry of `debian/changelog`, which is part of the package (`apt changelog picompose-software-hello` shows the whole history). You do not edit it by hand, the workflows write it.

1. **Pull request:** Every pull request needs exactly one of the labels `major`, `minor` and `patch`. It decides how the version is raised when it is merged. Tick one box under "Version bump" in the description or set the label by hand. The check "Version label" fails until there is exactly one. Make it required in the branch protection of `main`.
2. **Merge to `main`:** The workflow "Version" raises the version by the highest label of the merged pull requests, adds the pull request titles to `debian/changelog` and commits it to `main` (the commit "Version 1.3.0"). It fails if a pull request has no or more than one label, or if a commit has no pull request. Start it by hand and choose a level to get past that.
3. **Pre-release:** The workflow creates the tag `v1.3.0` as a **pre-release**, with the release notes that GitHub generates from the template in `.github/release.yml` for the pull requests since the last tag.
4. **Build:** The workflow "Build and publish a version" is started on the tag. It builds the package from the tag and publishes it to the suite `trixie-nightly`.
5. **Release:** In the GitHub UI, open the pre-release and turn it into a release (untick "Set as a pre-release"). That starts the workflow "Build and publish a version" for the release, which publishes the same version to the suite `trixie` and generates the release notes again for all pull requests since the last release.
6. **Any other branch:** The package gets the version `<version>+dev<timestamp>.<sha>`. Nothing is published, the packages are kept as workflow artifacts for 14 days. Use a branch per change to try it on a device.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-software-hello_*.deb
```

The `.deb` files are written to the parent directory.

## Publishing

The workflows call the reusable workflows of the [PiCompose-Repository](https://github.com/florian-asche/PiCompose-Repository), which raise the version, build the package and tell the APT repository to publish it.

| Workflow | Started by | Result |
| --- | --- | --- |
| Version | Push to `main`, or by hand | New version in `debian/changelog`, tag `v<version>` as pre-release, starts the build |
| Build and publish a version | The Version workflow (pre-release), or turning the pre-release into a release | Pre-release: suite `trixie-nightly`. Release: suite `trixie` |
| Build and publish | Push to another branch, pull request from a fork | Artifact for 14 days, nothing published |

The repository needs the secret `HUB_DISPATCH_TOKEN` and a branch protection of `main` that requires the check "Version label" and lets `github-actions[bot]` push, see the [DEVELOPMENT.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/DEVELOPMENT.md) of the APT repository.

## License

This project is released under the [MIT License](LICENSE).

The greeting is a test of the build and publishing pipeline.

The version of this package is raised by the label of the pull request that is merged.
