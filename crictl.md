# Container Runtime Interface (CRI) CLI

`crictl` provides a CLI for CRI-compatible container runtimes. This allows the CRI runtime developers to debug their runtime without needing to set up Kubernetes components.

`crictl` is currently in Beta and still under quick iterations. It is hosted at the [cri-tools](https://github.com/kubernetes-sigs/cri-tools) repository. We encourage the CRI developers to report bugs or help extend the coverage by adding more functionalities.

## Install crictl

> **NOTE:** The below steps are based on linux-amd64, however you can get downloads for all other platforms (Windows, ARM, etc) in the [releases page](https://github.com/kubernetes-sigs/cri-tools/releases).

`crictl` can be downloaded from cri-tools [release page](https://github.com/kubernetes-sigs/cri-tools/releases):

- using `wget`:

```sh
VERSION="v1.26.0" # check latest version in /releases page
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

- using `curl`:

```sh
VERSION="v1.26.0" # check latest version in /releases page
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-${VERSION}-linux-amd64.tar.gz --output crictl-${VERSION}-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

## Usage

```sh
crictl [global options] command [command options] [arguments...]
```

COMMANDS:

- `attach`:             Attach to a running container
- `create`:             Create a new container
- `exec`:               Run a command in a running container
- `version`:            Display runtime version information
- `images, image, img`: List images
- `inspect`:            Display the status of one or more containers
- `inspecti`:           Return the status of one or more images
- `imagefsinfo`:        Return image filesystem info
- `inspectp`:           Display the status of one or more pods
- `logs`:               Fetch the logs of a container
- `port-forward`:       Forward local port to a pod
- `ps`:                 List containers
- `pull`:               Pull an image from a registry
- `run`:                Run a new container inside a sandbox
- `runp`:               Run a new pod
- `rm`:                 Remove one or more containers
- `rmi`:                Remove one or more images
- `rmp`:                Remove one or more pods
- `pods`:               List pods
- `start`:              Start one or more created containers
- `info`:               Display information of the container runtime
- `stop`:               Stop one or more running containers
- `stopp`:              Stop one or more running pods
- `update`:             Update one or more running containers
- `config`:             Get and set `crictl` client configuration options
- `stats`:              List container(s) resource usage statistics
- `statsp`:             List pod(s) resource usage statistics
- `completion`:         Output bash shell completion code
- `checkpoint`:         Checkpoint one or more running containers
- `help, h`:            Shows a list of commands or help for one command

`crictl` by default connects on Unix to:

- `unix:///var/run/dockershim.sock` or
- `unix:///run/containerd/containerd.sock` or
- `unix:///run/crio/crio.sock` or
- `unix:///var/run/cri-dockerd.sock`

or on Windows to:

- `npipe:////./pipe/dockershim` or
- `npipe:////./pipe/containerd-containerd` or
- `npipe:////./pipe/cri-dockerd`

For other runtimes, use:

- [frakti](https://github.com/kubernetes/frakti): `unix:///var/run/frakti.sock`

The endpoint can be set in three ways:

- By setting global option flags `--runtime-endpoint` (`-r`) and `--image-endpoint` (`-i`)
- By setting environment variables `CONTAINER_RUNTIME_ENDPOINT` and `IMAGE_SERVICE_ENDPOINT`
- By setting the endpoint in the config file `--config=/etc/crictl.yaml`

If the endpoint is not set then it works as follows:

- If the runtime endpoint is not set, `crictl` will by default try to connect using:
  - dockershim
  - containerd
  - cri-o
  - cri-dockerd
- If the image endpoint is not set, `crictl` will by default use the runtime endpoint setting

> Note: The default endpoints are now deprecated and the runtime endpoint should always be set instead.
The performance maybe affected as each default connection attempt takes n-seconds to complete before timing out and going to the next in sequence.

Unix:

```sh
$ cat /etc/crictl.yaml
runtime-endpoint: unix:///var/run/dockershim.sock
image-endpoint: unix:///var/run/dockershim.sock
timeout: 2
debug: true
pull-image-on-create: false
```

Windows:

```cmd
C:\> type %USERPROFILE%\.crictl\crictl.yaml
runtime-endpoint: tcp://localhost:3735
image-endpoint: tcp://localhost:3735
timeout: 2
debug: true
pull-image-on-create: false
```
## Additional options

- `--timeout`, `-t`: Timeout of connecting to server in seconds (default: `2s`).
0 or less is interpreted as unset and converted to the default. There is no
option for no timeout value set and the smallest supported timeout is `1s`
- `--debug`, `-D`: Enable debug output
- `--help`, `-h`: show help
- `--version`, `-v`: print the version information of `crictl`
- `--config`, `-c`: Location of the client config file (default: `/etc/crictl.yaml`). Can be changed by setting `CRI_CONFIG_FILE` environment variable. If not specified and the default does not exist, the program's directory is searched as well

## Client Configuration Options

Use the `crictl` config command to get and set the `crictl` client configuration
options.

USAGE:

```sh
crictl config [command options] [<crictl options>]
```

For example `crictl config --set debug=true` will enable debug mode when giving subsequent `crictl` commands.

COMMAND OPTIONS:

- `--get value`: Show the option value
- `--set value`: Set option (can specify multiple or separate values with commas: opt1=val1,opt2=val2)
- `--help`, `-h`: Show help (default: `false`)

`crictl` OPTIONS:

- `runtime-endpoint`: Container runtime endpoint (no default value)
- `image-endpoint`: Image endpoint (no default value)
- `timeout`: Timeout of connecting to server (default: `2s`)
- `debug`: Enable debug output (default: `false`)
- `pull-image-on-create`: Enable pulling image on create requests (default: `false`)
- `disable-pull-on-run`: Disable pulling image on run requests (default: `false`)

> When enabled `pull-image-on-create` modifies the create container command to first pull the container's image.
This feature is used as a helper to make creating containers easier and faster.
Some users of `crictl` may desire to not pull the image necessary to create the container.
For example, the image may have already been pulled or otherwise loaded into the container runtime, or the user may be running without a network. For this reason the default for `pull-image-on-create` is `false`.

> By default the run command first pulls the container image, and `disable-pull-on-run` is `false`.
Some users of `crictl` may desire to set `disable-pull-on-run` to `true` to not pull the image by default when using the run command.

> To override these default pull configuration settings, `--no-pull` and `--with-pull` options are provided for the create and run commands.
