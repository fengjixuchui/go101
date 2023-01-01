
# GoTV

**GoTV** is an abbreviation of **Go** **T**oolchain **V**ersion.
It is a tool used to manage and use multiple coexisting installations of official Go toolchain versions harmoniously and conveniently.

Project page: https://github.com/go101/gotv

Please follow [@Go100and1](https://twitter.com/go100and1) to get the latest news of GoTV
(and all kinds of Go details/facts/tips/...).

## Installation

Run

```
go install go101.org/gotv@latest
```

to install GoTV.

A 1.17+ toolchain version is needed to finish the installation.
The toolchain version may be uninstalled after pinning a suitable toolchain version (see below).

## Usage

Run `gotv` without any arguments to show help messages.

Most `gotv` commands are in the following format:

```
gotv ToolchainVersion [go-arguments...]
```

During running the first such a command, the Go git repository will be cloned (which needs several minutes to finish).

`ToolchainVersion` might be

* a Go release version, such as `1.17.13`, `1.18`, `1.19rc1`,
  which mean the release tags `go1.17.13`, `go1.18`, `go1.19rc1`, respectively,
  in the Go git repository.
  Note:
  * `1.N.` means the latest release of `1.N`.
  * `1.` means the latest Go 1 release version.
  * `.` means the latest Go release version.
* `:1.N`, which means the local latest `release-branch.go1.N` branch in the Go git repository.
* `:tip`, which means the local latest `master` branch in the Go git repository.
* a version suffixed with `!` means to fetch remote versions (by running `gotv fetch-versions`) firstly.

Examples:

```
$ gotv 1.17. version
[Run]: $HOME/.cache/gotv/tag_go1.17.13/bin/go version
go version go1.17.13 linux/amd64

$ gotv 1.18.3 version
[Run]: $HOME/.cache/gotv/tag_go1.18.3/bin/go version
go version go1.18.3 linux/amd64

$ cat main.go
package main

const A = 3

func main() {
	const (
		A = A + A
		B
	)
	println(A, B)
}

$ gotv 1.17. run main.go
[Run]: $HOME/.cache/gotv/tag_go1.17.13/bin/go run main.go
6 6

$ gotv 1.18.3 run main.go
[Run]: $HOME/.cache/gotv/tag_go1.18.3/bin/go run main.go
6 12
```

Other `gotv` commands:

```
# sync the local Go git repository with remote
gotv fetch-versions

# list all versions seen locally
gotv list-versions

# build and cache some toolchain versions
gotv cache-version ToolchainVersion [ToolchainVersion ...]

# uncache some toolchain versions to save disk space
gotv uncache-version ToolchainVersion [ToolchainVersion ...]

# pin a specified toolchain version at a stable path
gotv pin-version ToolchainVersion

# unpin the current pinned toolchain version
gotv unpin-version
```

## Pin a toolchain version

We can use the `gotv pin-version` command to pin a specific toolchain version to a stable path.
After adding the stable path to the `PATH` environment veriable,
we can use the official `go` command directly.
And after doing these, the toolchain versions installed through ways other than GoTV
may be safely uninstalled.

It is recommanded to pin a 1.17+ version for [bootstrap purpose](https://github.com/golang/go/issues/44505) now.
The following example shows how to pin Go toolchain version 1.17.13:

```
$ gotv pin-version 1.17.
[Run]: cp -r $HOME/.cache/gotv/tag_go1.17.13 $HOME/.cache/gotv/pinned-toolchain

Please put the following shown pinned toolchain path in
your PATH environment variable to use go commands directly:

	/home/username/.cache/gotv/pinned-toolchain/bin
```

After the prompted path is added to the PATH environment variable,
open a new terminal window:

```
$ go version
go version go1.17.13 linux/amd64
```

The command `gotv pin-version .!` will upgrade the pinned toolchain to the latest release version
(which may be a beta or rc version).

## Set a bootstrap toolchain version

To build a toolchain verision, another already built toolchain version is needed to be used in the building process.
The other toolchain version is called the bootstrap version.

Some facts:

* Toolchain versions <= 1.12.17 are unable to be built with toochain versions >= 1.16;
* Toolchain versions <= 1.5.4 are uanable to be built with toolchain versions >= 1.6;
* It is planned to [require a 1.17+ toolchain version to build 1.20+ toolchain versions](https://github.com/golang/go/issues/44505);
* It is proposed to [require a 1.20+ toolchain verison to build 1.22+ toolchain verisons](https://github.com/golang/go/issues/54265).

Currently, GoTV uses the toolchain set in the `PATH` environment variable as the bootstrap version by default.
If `GOROOT_BOOTSTRAP` environment variable is set, then its value will be used.







