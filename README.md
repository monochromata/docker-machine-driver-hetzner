# Hetzner Cloud Docker machine driver

[![Go Report Card](https://goreportcard.com/badge/github.com/JonasProgrammer/docker-machine-driver-hetzner)](https://goreportcard.com/report/github.com/JonasProgrammer/docker-machine-driver-hetzner)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://secure.travis-ci.org/JonasProgrammer/docker-machine-driver-hetzner.png)](http://travis-ci.org/JonasProgrammer/docker-machine-driver-hetzner)

> This library adds the support for creating [Docker machines](https://github.com/docker/machine) hosted on the [Hetzner Cloud](hetzner.de/cloud). 

You need to create a project-specific access token under `Access` > `API Tokens` in the project control panel
and pass that to `docker-machine create` with the `--hetzner-api-token` option.

## Installation

You can find sources and pre-compiled binaries [here](https://github.com/JonasProgrammer/docker-machine-driver-hetzner/releases).

```shell
# Download the binary (this example downloads the binary for linux amd64)
$ wget https://github.com/JonasProgrammer/docker-machine-driver-hetzner/releases/download/0.2.4/docker-machine-driver-hetzner_0.2.4_linux_amd64.tar.gz
$ tar -xvf docker-machine-driver-hetzner_0.2.4_linux_amd64.tar.gz

# Make it executable and copy the binary in a directory accessible with your $PATH
$ chmod +x docker-machine-driver-hetzner
$ cp docker-machine-driver-hetzner /usr/local/bin/
```

## Usage

    $ docker-machine create \
      --driver hetzner \
      --hetzner-api-token=QJhoRT38JfAUO037PWJ5Zt9iAABIxdxdh4gPqNkUGKIrUMd6I3cPIsfKozI513sy \
      some-machine
    
### Using environment variables

    $ HETZNER_API_TOKEN=QJhoRT38JfAUO037PWJ5Zt9iAABIxdxdh4gPqNkUGKIrUMd6I3cPIsfKozI513sy \
      && HETZNER_IMAGE=centos-7 \
      && docker-machine create \
         --driver hetzner \
         some-machine
    

## Options

-   `--hetzner-api-token`: **required**. Your project-specific access token for the Hetzner Cloud API.
-   `--hetzner-image`: The name of the Hetzner Cloud image to use, see [Images API](https://docs.hetzner.cloud/#resources-images-get) for how to get a list.
-   `--hetzner-server-type`: The type of the Hetzner Cloud server, see [Server Types API](https://docs.hetzner.cloud/#resources-server-types-get) for how to get a list.
-   `--hetzner-server-location`: The location to create the server in, see [Locations API](https://docs.hetzner.cloud/#resources-locations-get) for how to get a list.
**NOTICE: Beware that Hetzner does not reject invalid location names at the time of writing this; instead, a seemingly random location is chosen. Double check both the option value's
spelling and the newly created server to make sure the desired location was chosen indeed.**
-   `--hetzner-existing-key-path`: Use an existing (local) SSH key instead of generating a new keypair.
-   `--hetzner-existing-key-id`: **requires `--hetzner-existing-key-path`**. Use an existing (remote) SSH key instead of uploading the imported key pair,
    see [SSH Keys API](https://docs.hetzner.cloud/#resources-ssh-keys-get) for how to get a list

The Hetzner Cloud driver will use `debian-9` as the default image and `cx11` as the default type.

#### Existing SSH keys

When you specify the `--hetzner-existing-key-path` option, the driver will attempt to copy `(specified file name)`
and `(specified file name).pub` to the machine's store path. They public key file's permissions will be set according
to your current `umask` and the private key file will have `600` permissions.

When you additionally specify the `--hetzner-existing-key-id` option, the driver will not create an SSH key using the API
but rather try to use the existing public key corresponding to the given id. Please note that during machine creation,
the driver will attempt to [get the key](https://docs.hetzner.cloud/#resources-ssh-keys-get-1) and **compare it's
fingerprint to the local public key's fingerprtint**. Keep in mind that the both the local and the remote key must be
accessible and have matching fingerprints, otherwise the machine will fail it's pre-creation checks.

Also note that the driver will attempt to delete the linked key during machine removal, unless `--hetzner-existing-key-id`
was used during creation.

#### Environment variables and default values

| CLI option                          | Environment variable              | Default                    |
| ----------------------------------- | --------------------------------- | -------------------------- |
| **`--hetzner-api-token`**           | `HETZNER_API_TOKEN`               | -                          |
| `--hetzner-image `                  | `HETZNER_IMAGE_IMAGE`             | `debian-9`                 |
| `--hetzner-server-type`             | `HETZNER_TYPE`                    | `cx11`                     |
| `--hetzner-server-location`         | `HETZNER_LOCATION`                | - *(let Hetzner choose)*   |
| `--hetzner-existing-key-path`       | `HETZNER_EXISTING_KEY_PATH`       | - *(generate new keypair)* |
| `--hetzner-existing-key-id`         | `HETZNER_EXISTING_KEY_ID`         | 0 *(upload new key)*       |


## Building from source

Use an up-to-date version of [Go](https://golang.org/dl)

To use the driver, you can download the sources and build it locally:

```shell
# Get sources and build the binary at ~/go/bin/docker-machine-driver-hetzner
$ go get github.com/jonasprogrammer/docker-machine-driver-hetzner

# Make the binary accessible to docker-machine
$ export GOPATH=$(go env GOPATH)
$ export GOBIN=$GOPATH/bin
$ export PATH="$PATH:$GOBIN"
```

## Development

Fork this repository, yielding `github.com/<yourAccount>/docker-machine-driver-hetzner`.

```shell
# Get the sources of your fork and build it locally
$ go get github.com/<yourAccount>/docker-machine-driver-hetzner

# * This integrates your fork into the $GOPATH (typically pointing at ~/go)
# * Your sources are at $GOPATH/src/github.com/<yourAccount>/docker-machine-driver-hetzner
# * That folder is a local Git repository. You can pull, commit and push from there.
# * The binary will typically be at $GOPATH/bin/docker-machine-driver-hetzner
# * In the source directory $GOPATH/src/github.com/<yourAccount>/docker-machine-driver-hetzner
#   you may use go get to re-build the binary.
# * Note: when you build the driver from different repositories, e.g. from your fork
#   as well as github.com/jonasprogrammer/docker-machine-driver-hetzner,
#   the binary files generated by these builds are all called the same
#   and will hence override each other.

# Make the binary accessible to docker-machine
$ export GOPATH=$(go env GOPATH)
$ export GOBIN=$GOPATH/bin
$ export PATH="$PATH:$GOBIN"

# Make docker-machine output help including hetzner-specific options
$ docker-machine create --driver hetzner
```
