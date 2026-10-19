# Uplink

Uplink is the [agent.golf](https://agent.golf) service that you run in your own network. It receives the OpenTelemetry data of your coding agents (Claude Code, Codex, and OMP), keeps session content on your hosts, and sends session data to agent.golf.

Prompts, responses, and sensitive tool details stay encrypted on the Uplink host. Uplink opens every connection, and agent.golf never connects to Uplink.

Read [Uplink on Your Network](https://docs.agent.golf/concepts/uplink) to learn what Uplink does and what it sends.

This repository holds the Uplink [releases](https://github.com/agentgolf/uplink/releases). It does not hold the source code.

## Before you install

- An agent.golf organization.
- The `uplink` role in that organization for the member who signs in Uplink.
- A supported host, a Docker host, or a Kubernetes cluster, with a persistent disk for the state directory.
- Outbound access from Uplink to `api.agent.golf` on port 443 and to the agent.golf data connection on port 7422.
- Inbound access to port 4318 from the machines where your coding agents run.

## Supported systems

| System | Architecture | Tested on |
| --- | --- | --- |
| macOS 15 and later | Apple silicon (`arm64`) | macOS 15 and 26 |
| Linux with glibc 2.34 or later | `amd64`, `arm64` | Ubuntu 22.04, 24.04, and 26.04; Debian 13; RHEL 9 and 10; Amazon Linux 2023 |

Windows, Intel Macs, and Linux distributions with musl libc, such as Alpine Linux, are not supported.

## Install

### Host

Install the latest release:

```bash
curl -fsSL https://get.agent.golf/uplink | sudo sh
```

The script downloads the `uplink` binary for your system, checks it against `SHA256SUMS`, and installs it to `/usr/local/bin`. Set `UPLINK_VERSION` to install a specific version, or `UPLINK_INSTALL_DIR` to install into another directory.

Sign in, then start Uplink:

```bash
uplink auth login
uplink
```

To run Uplink as a systemd service, see [Install Uplink](https://docs.agent.golf/uplink/install#run-uplink-as-a-systemd-service).

### Docker

The image is `ghcr.io/agentgolf/uplink`, for `linux/amd64` and `linux/arm64`. Each release has the tag `<version>`.

```bash
docker run -d --name uplink --restart on-failure \
  -p 4318:4318 \
  -p 127.0.0.1:4319:4319 \
  -v uplink-state:/data/.ag-uplink \
  ghcr.io/agentgolf/uplink:<version>

docker exec -it uplink uplink auth login
```

### Kubernetes

The Helm chart is `oci://ghcr.io/agentgolf/charts/uplink`. The chart version is the Uplink version.

```bash
helm install uplink oci://ghcr.io/agentgolf/charts/uplink \
  --version <version> \
  --namespace uplink --create-namespace

kubectl --namespace uplink exec -it uplink-0 -c uplink -- uplink auth login
```

Read the chart values with `helm show values oci://ghcr.io/agentgolf/charts/uplink --version <version>`.

## Sign in

`uplink auth login` prints a sign-in URL and a code. Open the URL, and approve the sign-in as a member with the `uplink` role.

Uplink keeps its credentials in its state directory. Without credentials, Uplink does not exit. It stays not ready and logs the `uplink auth login` instruction every 30 seconds. After you sign in, Uplink continues its startup without a restart.

- Sign in one time for each Uplink instance. Instances must not share a credentials file.
- Keep the state directory on writable persistent storage.
- If Uplink stays stopped for more than 30 days, run `uplink auth login` again.

## Verify a download

Each release has `SHA256SUMS` and its keyless [cosign](https://docs.sigstore.dev/) signature bundle, `SHA256SUMS.sigstore.json`. The notes of each release show the exact `cosign verify-blob` command for that release.

```bash
cosign verify-blob SHA256SUMS \
  --bundle SHA256SUMS.sigstore.json \
  --certificate-identity-regexp '^https://github\.com/agentgolf/[^/]+/\.github/workflows/uplink-chart-publish\.yml@refs/(tags/uplink-v.+|heads/main)$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com

sha256sum --check --ignore-missing SHA256SUMS
```

## Upgrade

- **Host:** run the install script again, then restart Uplink.
- **Docker:** remove the container and run it again with the new tag and the same volume.
- **Kubernetes:** run `helm upgrade` with the new `--version` and the same values.

The state directory keeps the credentials, so you do not sign in again.

## Documentation

- [Uplink on Your Network](https://docs.agent.golf/concepts/uplink)
- [Install Uplink](https://docs.agent.golf/uplink/install)
- [Verify the installation](https://docs.agent.golf/uplink/verify)
- [Configuration](https://docs.agent.golf/uplink/configuration)
- [Troubleshooting](https://docs.agent.golf/uplink/troubleshooting)
