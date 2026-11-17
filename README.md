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
  changelog         version history, the first entry is the package version
  rules             build instructions (plain dh)
  install           which files go where
  copyright         license
  source/format     3.0 (native): the version has no Debian revision
picompose-hello     the script installed to /usr/bin
```

## Using it as a template

1. Copy the repository and name it `PiCompose-Package-<Name>`.
2. Rename the source and binary package in `debian/control` and the entries in `debian/changelog`. Package names start with `picompose-`.
3. Set `Architecture: all` for scripts, configuration and device tree overlays, and `any` for compiled content. This example uses `any` so that it is built for `amd64`, `arm64` and `armhf`, which exercises the multi-architecture pipeline.
4. Replace the script and `debian/install` with your own files, and adjust `debian/copyright` and `LICENSE`.

## Versioning

The version is the first entry in `debian/changelog`. Raise it for every change. Add a new entry with `dch -i` and use a later date than the previous entry, otherwise lintian reports `latest-changelog-entry-without-new-date`.

The PiCompose APT repository adds a suffix to builds that are not releases, for example `0.1.1~git20261117.194bae2` for builds from `main`. The `~` makes them sort below the release.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-hello_*.deb
```

The `.deb` files are written to the parent directory.

## Publishing

Packages are built and published by the PiCompose APT repository, see its [DEVELOPMENT.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/DEVELOPMENT.md). This repository has no build workflow of its own yet.

## License

This project is released under the [MIT License](LICENSE).
