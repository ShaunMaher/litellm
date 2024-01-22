# Helm Chart for LiteLLM

## Prerequisites

- Kubernetes 1.23+
- Helm 3.8.0+

If `db.deployStandalone` is used:
- PV provisioner support in the underlying infrastructure

If `db.useStackgresOperator` is used (not yet implemented):
- The Stackgres Operator must already be installed in the Kubernetes Cluster.  This chart will **not** install the operator if it is missing.

## Parameters

### LiteLLM Proxy Deployment Settings

| Name                                                       | Description                                                                                                                                                                           | Value |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| `replicaCount`                                             | The number of LiteLLM Proxy pods to be deployed                                                                                                                                       | `1`  |
| `image.repository`                                         | LiteLLM Proxy image repository                                                                                                                                                          | `ghcr.io/berriai/litellm`  |
| `image.pullPolicy`                                         | LiteLLM Proxy image pull policy                                                                                                                                                       | `IfNotPresent`  |
| `image.tag`                                                | Overrides the image tag whose default the latest version of LiteLLM at the time this chart was published.                                                                             | `""`  |
| `image.dbReadyImage`                                       | On Pod startup, an initContainer is used to make sure the Postgres database is available before attempting to start LiteLLM.  This field specifies the image to use as that initContainer.  | `docker.io/bitnami/postgresql`  |
| `image.dbReadyTag`                                         | Tag for the above image.  If not specified, "latest" is used.                                                                                                                         | `""`  |
| `imagePullSecrets`                                         | Registry credentials for the above images.                                                                                                                                            | `[]`  |
| `serviceAccount.create`                                    | Whether or not to create a Kubernetes Service Account for this deployment.  The default is `false` because LiteLLM has no need to access the Kubernetes API.                          | `false`  |
| `service.type`                                             | Kubernetes Service type (e.g. `LoadBalancer`, `ClusterIP`, etc.)                                                                                                                      | `ClusterIP`  |
| `service.port`                                             | TCP port that the Kubernetes Service will listen on.  Also the TCP port within the Pod that the proxy will listen on.                                                                 | `8000`  |
| `ingress.*`                                                | See [values.yaml](./values.yaml) for example settings                                                                                                                                 | N/A |
| `proxy_config.*`                                           | See [values.yaml](./values.yaml) for example settings                                                                                                                                 | `false`  |

### LiteLLM Admin UI Settings

| Name                                                       | Description                                                                                                                                                                           | Value |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| `ui.enabled`                                               | Should the LiteLLM Admin UI be deployed                                                                                                                                               | `true`  |
| `ui.replicaCount`                                          | The number of LiteLLM Admin UI pods to be deployed                                                                                                                                    | `1`   |
| `ui.image.repository`                                      | LiteLLM Admin UI image repository                                                                                                                                                     | `ghcr.io/berriai/litellm`  |
| `ui.image.pullPolicy`                                      | LiteLLM Admin UI image pull policy                                                                                                                                                    | `IfNotPresent`  |
| `ui.image.tag`                                             | Overrides the image tag whose default the latest version of LiteLLM at the time this chart was published.                                                                             | `""`  |
| `ui.imagePullSecrets`                                      | Registry credentials for the above images.                                                                                                                                                         | `[]`  |
| `ui.service.type`                                          | Kubernetes Service type (e.g. `LoadBalancer`, `ClusterIP`, etc.)                                                                                                                      | `ClusterIP`  |
| `ui.service.port`                                          | TCP port that the Kubernetes Service will listen on.  Also the TCP port within the Pod that the proxy will listen on.                                                                 | `8000`  |
| `ui.ingress.*`                                             | See [values.yaml](./values.yaml) for example settings                                                                                                                                 | N/A |

### Database Settings
| Name                                                       | Description                                                                                                                                                                           | Value |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| `db.useExisting`                                           | Use an existing Postgres database.  A Kubernetes Secret object must exist that contains credentials for connecting to the database.  An example secret object definition is provided below.  | `false`  |
| `db.endpoint`                                              | If `db.useExisting` is `true`, this is the IP, Hostname or Service Name of the Postgres server to connect to.                                                                         | `localhost`  |
| `db.database`                                              | If `db.useExisting` is `true`, the name of the existing database to connect to.                                                                                                       | `litellm`  |
| `db.secret.name`                                           | If `db.useExisting` is `true`, the name of the Kubernetes Secret that contains credentials.                                                                                           | `postgres`  |
| `db.secret.usernameKey`                                    | If `db.useExisting` is `true`, the name of the key within the Kubernetes Secret that holds the username for authenticating with the Postgres instance.                                | `username`  |
| `db.secret.passwordKey`                                    | If `db.useExisting` is `true`, the name of the key within the Kubernetes Secret that holds the password associates with the above user.                                               | `password`  |
| `db.useStackgresOperator`                                  | Not yet implemented.                                                                                                                                                                  | `false`  |
| `db.deployStandalone`                                      | Deploy a standalone, single instance deployment of Postgres, using the Bitnami postgresql chart.  This is useful for getting started but doesn't provide HA or (by default) data backups.  | `true`  |
| `postgresql.*`                                             | If `db.deployStandalone` is `true`, configuration passed to the Bitnami postgresql chart.   See the [Bitnami Documentation](https://github.com/bitnami/charts/tree/main/bitnami/postgresql) for full configuration details.  See [values.yaml](./values.yaml) for the default configuration.  | See [values.yaml](./values.yaml) |
| `postgresql.auth.*`                                        | If `db.deployStandalone` is `true`, care should be taken to ensure the default `password` and `postgres-password` values are **NOT** used.                                            | `NoTaGrEaTpAsSwOrD`  |

#### Example Postgres `db.useExisting` Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres
data:
  # Password for the "postgres" user
  postgres-password: <some secure password, base64 encoded>
  username: litellm
  password: <some secure password, base64 encoded>
type: Opaque
```