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
  changelog         the releases, written by the release workflow
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

`debian/changelog` holds the releases and is part of the package, so `apt changelog picompose-hello` shows the whole history. You do not edit it by hand, the release workflow writes it.

- **Release:** Start the workflow **Release** (Actions, Release, Run workflow) and enter a version like `1.2.3` that is higher than the last release. It writes the changelog entry from the notes GitHub generates from the merged pull requests, commits `debian/changelog` to `main`, tags that commit `v<version>`, creates the GitHub release and builds and publishes the package to the suite `trixie`. A release created by hand in the GitHub UI is not published.
- **Push to `main`:** The package gets the version `<last release>+git<timestamp>.<sha>` and is published to the suite `trixie-nightly`. The changelog entry of this build, made from the title and description of the pull request, is only part of the package. Every build overwrites the previous one.
- **Any other branch:** `<last release>+dev<timestamp>.<sha>`. Nothing is published, the packages are kept as workflow artifacts for 14 days. Use a branch per change to try it on a device.

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
| Workflow "Release" | suite `trixie` | the version you enter |
| Push to `main` | suite `trixie-nightly` | `<last release>+git<timestamp>.<sha>` |
| Push to another branch, pull request from a fork | artifact for 14 days | `<last release>+dev<timestamp>.<sha>` |

The repository needs the secret `HUB_DISPATCH_TOKEN`, see the [DEVELOPMENT.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/DEVELOPMENT.md) of the APT repository.

## License

This project is released under the [MIT License](LICENSE).

The greeting is a test of the build and publishing pipeline.
