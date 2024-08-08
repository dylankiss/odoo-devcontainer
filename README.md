# Odoo Development Container

This repository contains all files necessary to setup a [Visual Studio Code Dev Container](https://code.visualstudio.com/docs/devcontainers/containers) for Odoo development or just build the Docker image for it.

The prebuilt Docker image used is available at [Docker Hub](https://hub.docker.com/r/dylankiss/odoo-devcontainer).

It is based on the [Ubuntu 24.04 (Noble Numbat)](https://hub.docker.com/_/ubuntu) Docker image and the configuration is based on Odoo's Runbot Docker image and part of the [`common-utils` feature](https://github.com/devcontainers/features/tree/main/src/common-utils).

In short it does the following:
- Install all necessary Debian packages to develop and run Odoo (including Python and PostgreSQL).
- Install the right version of [`wkhtmltox`](https://github.com/wkhtmltopdf/packaging/releases).
- Configure the system with an `odoo` user and give the user access to PostgreSQL.
- Install `zsh` as default shell and [Oh My Zsh!](https://ohmyz.sh/) with a custom [`devcontainers` theme](https://github.com/devcontainers/features/blob/main/src/common-utils/scripts/devcontainers.zsh-theme) taken from the `common-utils` feature.
- Install all Python dependencies, with most of them from the latest [`odoo`](https://github.com/odoo/odoo) and [`documentation`](https://github.com/odoo/documentation) repositories' `requirements.txt` files.
- Install the necessary `node` modules.
- Install [`flamegraph`](https://github.com/brendangregg/FlameGraph).
- Start the PostgreSQL server when the container starts (if working via VS Code).

## Configuration for VS Code

In order to work in this container, you need to have [Visual Studio Code](https://code.visualstudio.com/) installed together with the [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension and [Docker](https://www.docker.com/).

Copy the `devcontainer.json` file in a `.devcontainer` folder in your workspace, alongside your Odoo repositories.

Your folder structure would look like this:

```
master
├── .devcontainer
│   └── devcontainer.json
├── design-themes
├── documentation
├── enterprise
└── odoo
```

If you have set up a multiverse environment with `git worktree` and want to include the (versionless) `internal`, `upgrade` and/or `upgrade-util` repositories in the container, you can create symlinks to them in your workspace folders. You might need to adapt the `devcontainer.json` file accordingly in the `"mounts"` section for the symlinks to work in the container.

By default the `devcontainer.json` file works with this folder structure, where e.g. `17.0` is opened as the workspace folder in VS Code:

```
odoo-dev
├── 17.0
│   ├── .devcontainer
│   ├── design-themes (17.0)
│   ├── documentation (17.0)
│   ├── enterprise (17.0)
│   ├── internal (symlink to ../internal)
│   ├── odoo (17.0)
│   ├── upgrade (symlink to ../upgrade)
│   └── upgrade-util (symlink to ../upgrade-util)
│
├── internal (master)
├── upgrade (master)
└── upgrade-util (master)
```

> [!TIP]
> If you have cloned your Odoo repositories using SSH instead of HTTPS, you need to follow [these instructions](https://code.visualstudio.com/remote/advancedcontainers/sharing-git-credentials#_using-ssh-keys) to share the SSH key(s) with your container.

## Usage with VS Code

Once set up, you can run your container for the first time by opening the Command Palette of Visual Studio Code using `Ctrl+Shift+P` (Linux/Windows) or `Cmd+Shift+P` (Mac) and searching for the command `Dev Containers: Open Folder in Container...`. Starting this command will pull the Docker image (which can take a while at first) and then launch your development container with Visual Studio Code as if it were a local development environment.

Subsequently opening folders in the container will go much quicker because the image is already pulled.

Once the container is launched, the terminal in Visual Studio Code will open as the `odoo` user in the `/workspaces/<your-workspace-folder>` directory and you can launch Odoo as you normally would, using e.g.

```sh
cd odoo
./odoo-bin --addons-path=../enterprise,./addons -d your-database -i base
```

## Usage without VS Code

First you need to pull the Docker image like this:

```sh
docker pull dylankiss/odoo-devcontainer
```

Assume your directory structure looks like this:

```
odoo-dev
├── 17.0
│   ├── design-themes (17.0)
│   ├── documentation (17.0)
│   ├── enterprise (17.0)
│   ├── internal (symlink to ../internal)
│   ├── odoo (17.0)
│   ├── upgrade (symlink to ../upgrade)
│   └── upgrade-util (symlink to ../upgrade-util)
│
├── internal (master)
├── upgrade (master)
└── upgrade-util (master)
```

You can run a working container will all necessary mounted volumes by invoking this command from the `odoo-dev` directory:

```sh
docker run -v ./17.0:/workspaces/17.0 \
           -v ./internal:/workspaces/internal \
           -v ./upgrade:/workspaces/upgrade \
           -v ./upgrade-util:/workspaces/upgrade-util \
           -w /workspaces/17.0 \
           -p 8069:8069 \
           -i -t dylankiss/odoo-devcontainer
```
> [!NOTE]
> The first four lines map the folders on our host system to a folder in the container so they will live update. The next two lines set the default workspace folder and expose the default Odoo port 8069 to the host system.

Once the container is launched, the terminal will run as the `odoo` user in the `/workspaces/17.0` directory in your container. To start developing, start the `postgres` service and launch Odoo as you normally would, using e.g.

```sh
# Start the PostgreSQL server
sudo service postgresql start

# Run Odoo
cd odoo
./odoo-bin --addons-path=../enterprise,./addons -d your-database -i base
```

After this, you can access your Odoo instance as usual via `http://localhost:8069`.

## Good to Know

The image is built for both `amd64` as well as `arm64` chipsets (like the Apple Silicon chips).
