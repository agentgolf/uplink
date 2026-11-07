# Uplink

Uplink is the self-hosted [Agent Golf](https://agent.golf) service. It receives the OpenTelemetry data of your coding agents, such as Claude Code, Codex, and omp. It keeps session content in your network, and sends session data to Agent Golf.

This repository holds the Uplink [releases](https://github.com/agentgolf/uplink/releases). Full documentation: [docs.agent.golf/uplink/overview](https://docs.agent.golf/uplink/overview).

Uplink runs on Linux (`amd64` or `arm64`, glibc 2.34 or later) and on macOS 26 or later on Apple silicon.

## Install

Each release has three forms: the `uplink` binary, the container image, and the Helm chart.

| Form          | Get it                                                                     |
| ------------- | -------------------------------------------------------------------------- |
| Binary        | `curl -fsSL https://get.agent.golf/uplink \| sh`                           |
| Container     | `docker pull ghcr.io/agentgolf/uplink:<version>`                           |
| Helm chart    | `oci://ghcr.io/agentgolf/charts/uplink`, chart version `<version>`         |

Replace `<version>` with the version of the [latest release](https://github.com/agentgolf/uplink/releases/latest). The install script checks the binary against `SHA256SUMS`. The notes of each release show the command that checks the signature of `SHA256SUMS`. See [Install Uplink](https://docs.agent.golf/uplink/install).

## Create an Uplink key

An Uplink key (`agu_...`) starts an uplink instance. It belongs to your workspace. You need the **Manage Uplink Keys** permission.

1. In the dashboard, open **Settings > Uplink > Self-Hosted**.
2. In **Keys That Start Uplink Instances**, enter a **Key Name**, for example the host or the cluster.
3. Select **Create Uplink Key**.
4. Copy the secret. Agent Golf cannot show it again.

## Start Uplink

Use the commands of the **Uplink Instance Setup** card in the dashboard. Put your key in place of `<uplink-key>`.

Docker:

```bash
docker run -d --name uplink --restart on-failure \
  -e AGENTGOLF_UPLINK_KEY=<uplink-key> \
  -p 4318:4318 -p 127.0.0.1:4319:4319 \
  -v uplink-state:/data/.ag-uplink \
  ghcr.io/agentgolf/uplink:<version>
```

Kubernetes:

```bash
kubectl create namespace uplink
kubectl --namespace uplink create secret generic uplink-key \
  --from-literal=uplink-key=<uplink-key>
helm install uplink oci://ghcr.io/agentgolf/charts/uplink --version <version> \
  --namespace uplink \
  --set uplinkKey.existingSecret=uplink-key
```

Linux / macOS:

```bash
curl -fsSL https://get.agent.golf/uplink | sh
export AGENTGOLF_UPLINK_KEY=<uplink-key>
uplink
```

The uplink instance shows in the **Uplink** section of the dashboard after its first report.

## Connect your harness

Each user connects their own harness with a personal publishing key (`agk_...`). The harness sends the key in the `x-agentgolf-publishing-key` header, so Agent Golf links the sessions to the user.

With the [`agentgolf` CLI](https://github.com/agentgolf/cli):

```bash
AGENTGOLF_API_BASE_URL=https://api.agent.golf agentgolf harness setup claude-code --endpoint 'http://127.0.0.1:4318'
```

Use `claude-code`, `codex`, or `omp`. Put the OTLP endpoint of your uplink instance in place of `http://127.0.0.1:4318`.

Or in the dashboard:

1. Open **Settings > Uplink > Publishing Keys**.
2. Enter a **Key Name**, and select **Create Publishing Key**. Copy the secret.
3. In **Harness Setup**, enter the **OTLP Endpoint of Your Uplink Instance**, and select your harness.
4. Copy the settings into your harness.

Restart the harness after you change its settings. See [Agent Harnesses](https://docs.agent.golf/harnesses/overview).

## Commands

| Command                                    | Result                                                                                                       |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `uplink`                                   | Start the uplink instance.                                                                                   |
| `uplink -version`                          | Show the version.                                                                                            |
| `uplink -help`                             | Show every flag.                                                                                             |
| `uplink auth create-key -name <name>`      | Sign in one time, create an Uplink key, and print its secret, for example for a Kubernetes Secret.           |
| `uplink auth login -key [-name <name>]`    | Sign in one time, create an Uplink key, and save it in the `uplink-key` file that the uplink instance reads. |
| `uplink auth status`                       | Show which Uplink key the uplink instance uses. It never shows the key and changes nothing.                  |
| `uplink auth logout`                       | Revoke the sign-in of the credentials file on Agent Golf, and remove the file.                               |

Main settings. Each setting is an environment variable. Some also have a flag, which overrides the variable.

| Variable                                  | Flag               | Default                 | Purpose                                                       |
| ----------------------------------------- | ------------------ | ----------------------- | ------------------------------------------------------------- |
| `AGENTGOLF_UPLINK_KEY`                    |                    | Empty                   | The Uplink key.                                               |
| `AGENTGOLF_UPLINK_KEY_FILE`               | `-uplink-key-file` | Empty                   | A file that holds the Uplink key, for example a mounted Secret. |
| `AGENTGOLF_WORKSPACE_ID`                  | `-workspace-id`    | Empty                   | Stop when the key belongs to another workspace.               |
| `OTLP_HTTP_HOST`, `OTLP_HTTP_PORT`        |                    | `127.0.0.1`, `4318`     | The OTLP listener for your harnesses.                         |
| `OTLP_TLS_CERT_FILE`, `OTLP_TLS_KEY_FILE` |                    | Empty                   | Serve the OTLP listener over HTTPS.                           |
| `OTLP_ADMIN_HOST`, `OTLP_ADMIN_PORT`      |                    | `127.0.0.1`, `4319`     | `/healthz`, `/readyz`, and `/metrics`.                        |
| `UPLINK_STORE_DIR`                        | `-store-dir`       | In `~/.ag-uplink/prod/` | Where Uplink keeps session data.                              |
| `UPLINK_LOG_LEVEL`                        | `-log-level`       | `info`                  | `error`, `warn`, `info`, `debug`, or `trace`.                 |

The container image binds both listeners on all interfaces. See [Configure Uplink](https://docs.agent.golf/uplink/configuration).

## Documentation

[docs.agent.golf/uplink/overview](https://docs.agent.golf/uplink/overview)
