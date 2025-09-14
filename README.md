# Breitbandmessung.de Docker Container
<!-- <> -->
- [Breitbandmessung.de Docker Container](#breitbandmessungde-docker-container)
  - [Tags](#tags)
  - [Deploy via docker run](#deploy-via-docker-run)
  - [Deploy via docker-compose](#deploy-via-docker-compose)
  - [Deploy as Portainer Stack](#deploy-as-portainer-stack)
  - [Automated Speedtesting](#automated-speedtesting)
    - [Setup breitbandmessung.de](#setup-breitbandmessungde)
    - [Start automation via GUI (easy method)](#start-automation-via-gui-easy-method)
    - [Start automation via terminal (safe, fallback method)](#start-automation-via-terminal-safe-fallback-method)
    - [During the process](#during-the-process)
  - [Support for ARM-Architecture (Raspberry Pi)](#support-for-arm-architecture-raspberry-pi)
  - [Manually Building the Container (for development purposes)](#manually-building-the-container-for-development-purposes)
  - [Local testing with act](#local-testing-with-act)
    - [Prerequisites](#prerequisites)
    - [Option A: Secrets file (recommended)](#option-a-secrets-file-recommended)
    - [Option B: PowerShell one‑liner (Windows)](#option-b-powershell-oneliner-windows)
    - [Option C: Bash/WSL one‑liners (Linux/macOS/WSL)](#option-c-bashwsl-oneliners-linuxmacoswsl)
    - [Notes](#notes)
    - [Troubleshooting](#troubleshooting)
  - [Additional Notes](#additional-notes)


Setup the container in these five steps:

1. Deploy the container through one of the deployment options.
(The initial start can take a multiple seconds, depending on your internet connection.)

2. Configure your "Tarifangaben" inside the breibandmessung App.

3. (optional) Start the automation for automatic measurements. (see ➡️ [Automated Speedtesting](#automated-speedtesting))

## Tags

| Registry | Image | Tag | Build |
|:------------------:|:------------------:|:--------------:|:-----------------:|
| [Docker-Hub](https://hub.docker.com/r/goldjunge491/breitbandmessung/tags) | goldjunge91/breitbandmessung | latest | [![Build and Publish Docker Image](https://github.com/goldjunge91/breitbandmessung-docker/actions/workflows/build_docker_image.yml/badge.svg?branch=master)](https://github.com/goldjunge91/breitbandmessung-docker/actions/workflows/build_docker_image.yml) |
| [Github (ghcr.io)](https://github.com/goldjunge91/breitbandmessung-docker/pkgs/container/breitbandmessung-docker/versions?filters%5Bversion_type%5D=tagged) | ghcr.io/goldjunge91/breitbandmessung-docker | latest | [![Build and Publish Docker Image](https://github.com/goldjunge91/breitbandmessung-docker/actions/workflows/build_docker_image.yml/badge.svg?branch=master)](https://github.com/goldjunge91/breitbandmessung-docker/actions/workflows/build_docker_image.yml) |
| [Github (ghcr.io)](https://github.com/goldjunge91/breitbandmessung-docker/pkgs/container/breitbandmessung-docker/versions?filters%5Bversion_type%5D=tagged) | ghcr.io/goldjunge91/breitbandmessung-docker | staging | [![Build and Publish Docker Image](https://github.com/goldjunge91/breitbandmessung-docker/actions/workflows/build_docker_image.yml/badge.svg?branch=staging)](https://github.com/goldjunge91/breitbandmessung-docker/actions/workflows/build_docker_image.yml) |

## Deploy via docker run

The container can be run via plain docker:

```bash
docker run -d \
    --name breitband-desktop \
    -e TZ=Europe/Berlin  `#optional (default)` \
    -v $PWD/breitbandmessung/data:/config/xdg/config/Breitbandmessung \
    -p 5800:5800 \
    fabianbees/breitbandmessung:latest
```

Appdata for the Breitbandmessung Desktop App lives in the following directory (inside the container): ```/config/xdg/config/Breitbandmessung```. Therefore this directory should be mounted to a host directory.

## Deploy via docker-compose

Deploy container via docker-compose v3 schema:

```bash
git clone https://github.com/goldjunge91/breitbandmessung-docker.git

cd breitbandmessung-docker

docker compose up
```

```yaml
version: "3.8"
services:
  breitband-desktop:
  image: goldjunge491/breitbandmessung:latest
    container_name: breitband-desktop
    environment:
      - TZ=Europe/Berlin
    volumes:
      - $PWD/breitbandmessung/data:/config/xdg/config/Breitbandmessung
    ports:
      - 5800:5800
    restart: unless-stopped
```

## Deploy as Portainer Stack

<details>
<summary>see screenshot (click to expand)</summary>
<br>
<img src="screenshots/portainer-stack.png">
</details>

## Automated Speedtesting

### Setup breitbandmessung.de

1. Open your browser with the following url: <http://ip-of-docker-host:5800>

2. Go throgh setup process, until you reach the following page:
![Screenshot1](screenshots/screenshot1.png)
**DO NOT KLICK THE BUTTON "Messung durchführen" if you want to use the Speedtest automation script!**
--> The automation script requires this exact screen to be shown for the automatic execution of a "Messkampagne".

### Start automation via GUI (easy method)

3. To start the script, use the website on the exposed port and put the string 'RUN' in the clipboard. To stop the script, remove the string. You may need to do this twice (error unknown). After a maximum of 15 seconds you should see the screen in action.
![Screenshot1](screenshots/clipboard.png)

```⚠️ If the clipboard method doesn't work for you, please try the following alternate method first, before opening an issue!```

### Start automation via terminal (safe, fallback method)

3. open a console (bash) to your docker container (```docker exec -it breitband-desktop bash```) and execute the following command inside this docker container:

```bash
touch /RUN
```

This creates a empty file called ```RUN``` in the root directory of the container, the automation script is looking for this file for knowing when the setup process has finished and speedtesting can start.

### During the process

4. Speedtesting get's started, the script tries to click through the buttons for running a speedtest every 5 minutes. If the countdown timer (waiting period) has not finished yet, the clicks will do nothing.

5. When all mesurements are done, the automation-script can be stopped by removing the ```/RUN``` file with the following command ```rm /RUN``` inside the docker container or change the content of the clipboard to something else than `RUN` via the GUI.

## Support for ARM-Architecture (Raspberry Pi)

```⚠️ The ARM-Architecture (➡️ also all Raspberry Pi's) is not supported! ⚠️```

Support for this architecture currently cannot be provided, as the precompiled binary of the "breitbandmessung.de" program is not available for this architecture.

You can try your luck and contact the developers of the official app (<https://breitbandmessung.de/impressum> ➡️ <info@breitbandmessung.de>) and ask them to publish a linux .deb package compiled for the aarch64 architecture.

## Manually Building the Container (for development purposes)

You can also build the docker container localy for development.

```⚠️ Previously this was a mandatory step for normal deployment, now the app is installed on start of the container (not packaged in the image itself).```

You can do this with the following commands:

```bash
git clone https://github.com/goldjunge91/breitbandmessung-docker.git

cd breitbandmessung-docker

docker build -t breitband:latest .
```

## Local testing with act

Run GitHub Actions locally with act to iterate quickly on workflow changes without pushing commits.
act supports passing secrets via environment variables or a secrets file, which is the safest way to provide a token locally.

### Prerequisites

- Docker and act installed locally.
- GitHub CLI authenticated; gh auth status --show-token prints a token.

### Option A: Secrets file (recommended)

Create a .secrets file in the repo root and load it with --secret-file to avoid exposing tokens in process args.
The secrets file uses .env format (one KEY=VALUE per line).

```text
# .secrets (keep out of version control)
GITHUB_TOKEN=gho_YOUR_TOKEN_HERE
```

Run the workflow job locally, loading the secrets file:

```bash
act -W .github/workflows/build_docker_image.yml push -j build --verbose --secret-file .secrets
```

### Option B: PowerShell one‑liner (Windows)

This extracts the token from gh auth status and sets GITHUB_TOKEN for the current session, then passes it to act as a secret.

```powershell
$g = (gh auth status --show-token 2>&1 | Out-String); if($g -match '(gh[a-z]*_[A-Za-z0-9_]+)'){ $t=$Matches[48]; $env:GITHUB_TOKEN=$t; $last4 = if($t.Length -ge 4){ $t.Substring($t.Length-4) } else { $t }; Write-Host "GITHUB_TOKEN set (masked): ***$last4" } else { Write-Host "No token found" }
act -W .github/workflows/build_docker_image.yml push -j build --verbose -s GITHUB_TOKEN
```

### Option C: Bash/WSL one‑liners (Linux/macOS/WSL)

These extract the token from gh auth status and set GITHUB_TOKEN for the current shell, then pass it to act as a secret.

Simple extraction:

```bash
token=$(gh auth status --show-token 2>&1 | sed -n 's/.*Token: //p' | tr -d '\r'); if [ -n "$token" ]; then export GITHUB_TOKEN="$token"; last4=${token: -4}; printf 'GITHUB_TOKEN set (masked): ***%s\n' "$last4"; else echo "No token found"; fi
act -W .github/workflows/build_docker_image.yml push -j build --verbose -s GITHUB_TOKEN
```

Robust regex extraction:

```bash
token=$(gh auth status --show-token 2>&1 | grep -oE 'gh[a-z]*_[A-Za-z0-9_]+' | head -n1); if [ -n "$token" ]; then export GITHUB_TOKEN="$token"; last4=${token: -4}; printf 'GITHUB_TOKEN set (masked): ***%s\n' "$last4"; else echo "No token found"; fi
act -W .github/workflows/build_docker_image.yml push -j build --verbose -s GITHUB_TOKEN
```

### Notes

- Never commit tokens; store tokens in secrets and local .secrets files excluded from version control.
- Passing -s GITHUB_TOKEN tells act to use the environment variable or prompt securely, mapping it to secrets.GITHUB_TOKEN in the workflow.

### Troubleshooting

- If cloning public actions fails with authentication or network errors, verify container networking and retry.
- For workflows that upload/download artifacts, pass a local path via --artifact-server-path to emulate the Actions runtime.

## Additional Notes

- The automation-script is configured, to run speedtests only between 13:00 and 22:30 o'clock, because it is assumed, that the network load between 0 o'clock and 13:00 AM is lower than it is under normal use during the day.
(This behaviour could be alterd, by changing the values in the **if statement in line 18** of the file ```run-speedtest.sh``` before building the docker image).

- If you want to have more granular control for when speedtest should be run, the docker container could be stopped when no speedtest should be run, and restarted if testing should continue.

- A "Messkampagne" which is in progrss, can only be stopped, by purging the appdata of the container, or by altering your "Tarifangaben".
