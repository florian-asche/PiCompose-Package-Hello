# PiCompose-Package-Hello

Test package `picompose-hello` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It prints a greeting and exists to test the build, signing and publishing pipeline end to end. It is not needed on a real device.

## Install

Add the PiCompose APT repository (see [its documentation](https://github.com/florian-asche/PiCompose-Repository/blob/main/docs/apt_repository.md)), then:

```sh
sudo apt install picompose-hello
picompose-hello
```

## Layout

```
debian/             Debian packaging (control, changelog, rules, copyright, install)
picompose-hello     the script installed to /usr/bin
```

The version is the first entry in `debian/changelog`. The package uses the source format `3.0 (native)`, so the version has no Debian revision.

## Local build

```sh
sudo apt install build-essential debhelper devscripts lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-hello_*.deb
```

## Releases

A release of this repository is published to the APT suite `trixie` by the PiCompose APT repository. Pushes to `main` go to `trixie-nightly` and pushes to the branch `test` to `trixie-test`.

## License

This project is released under the [MIT License](LICENSE).
