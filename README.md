# Auto-deploy rails chart

Initially forked from GitLab's Auto-deploy Helm Chart (https://gitlab.com/gitlab-org/charts/auto-deploy-app), but due to need workers, this fork was created.

Initially sidekiq is implemented.

Delayed job should be a matter of adding the proper commands

Note: Worker has been added initially to gitlabs own chart: See this commit (https://gitlab.com/gitlab-org/charts/auto-deploy-app/commit/a76e282d6cb72b8f52315efc3fd933beb5aa2789)

## Requirements

- Helm `2.9.0` and above is required in order support `"helm.sh/hook-delete-policy": before-hook-creation` for migrations

## Note on securityContext

This is set to run as user 1001, and group 1001, so ensure that your container has user 1001 and 1001 created. See https://github.com/leifcr/rails5-kubernetes for example

## Usage with helm

### With gitlab

In your CI env variables, or in your modified ```.gitlab-ci.yml``` set the following:

Set ```AUTO_DEVOPS_CHART_REPOSITORY``` to https://leifcr.gitlab.io/auto-deploy-rails
Set ```AUTO_DEVOPS_CHART``` to gitlab/auto-deploy-rails

*Note:* due to a bug in the .gitlab-ci.yml, you need to use gitlab/auto-deploy-rails instead of auto-deploy-rails/auto-deploy-rails, 
even if the latter makes sense looking at the repo. This will conflict if you are using gitlab charts on the same helm instance. As 
you are unlikely to use gitlab charts during deployment, it should work. If you use gitlab charts as well, look at the solution below.

You can also opt to change download_chart in gitlab-ci.yml to the following:

```
function download_chart() {
  if [[ ! -d chart ]]; then
    auto_chart=${AUTO_DEVOPS_CHART:-gitlab/auto-deploy-app}
    auto_chart_base=$(dirname $auto_chart)
    auto_chart_name=$(basename $auto_chart)
    auto_chart_name=${auto_chart_name%.tgz}
    auto_chart_name=${auto_chart_name%.tar.gz}
  else
    auto_chart="chart"
    auto_chart_name="chart"
  fi

  helm init --client-only
  helm repo add $auto_chart_base ${AUTO_DEVOPS_CHART_REPOSITORY:-https://charts.gitlab.io}
  if [[ ! -d "$auto_chart" ]]; then
    helm fetch ${auto_chart} --untar
  fi
  if [ "$auto_chart_name" != "chart" ]; then
    mv ${auto_chart_name} chart
  fi

  helm dependency update chart/
  helm dependency build chart/
}

```

### Manual Usage

Add the repo

```helm repo add auto-deploy-rails https://leifcr.gitlab.io/auto-deploy-rails/```

Copy [```values.yaml```](https://gitlab.com/leifcr/auto-deploy-rails/blob/master/values.yaml) from the repository and modify to fit your app.

## Configuration

| Parameter                          | Description | Default                            |
| ---                                | ---         | ---                                |
| service.replicaCount               |             | `1`                                |
| service.revisionHistoryLimit       |             | `10`                                |
| service.securityContext.enabled    |          | `true` |
| service.securityContext.fsGroup    |          | `1001` |
| service.securityContext.runAsUser  |          | `1001` |
| image.repository                   |             | `gitlab.example.com/group/project` |
| image.tag                          |             | `stable`                           |
| image.pullPolicy                   |             | `Always`                           |
| image.secrets                      |             | `[name: gitlab-registry]`          |
| podAnnotations                     | Pod annotations | `{}`                           |
| application.track                  |             | `stable`                           |
| application.tier                   |             | `web`                              |
| application.migrateCommand         | If present, this variable will run as a shell command within an application Container as a Helm pre-upgrade Hook. Intended to run migration commands. | `nil` |
| application.initializeCommand      | If present, this variable will run as shall command within an application Container as a Helm post-install Hook. Intended to run database initialization commands. | `nil` |
| application.secretName             | Pass in the name of a Secret which the deployment will [load all key-value pairs from the Secret as environment variables](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables) in the application container. | `nil` |
| application.secretChecksum         | Pass in the checksum of the secrets referenced by `application.secretName`. | `nil` |
| hpa.enabled                        | If true, enables horizontal pod autoscaler. A resource request is also required to be set, such as `resources.requests.cpu: 200m`.| `false` |
| hpa.minReplicas                    |             | `1`                                |
| hpa.maxReplicas                    |             | `5`                                |
| gitlab.env                         | GitLab environment. | `nil` |
| gitlab.app                         | GitLab project slug. | `nil` |
| service.enabled                    |             | `true`                             |
| service.args                       | Change the service arguments on the fly if needed | `` |
| service.annotations                | Service annotations | `{}`                       |
| service.name                       |             | `web`                              |
| service.type                       |             | `ClusterIP`                        |
| service.url                        |             | `http://my.host.com/`              |
| service.additionalHosts            | If present, this list will add additional hostnames to the server configuration. | `nil` |
Alternative Names (SANs) | `nil`     |
| service.externalPort               |             | `3000`                             |
| service.internalPort               |             | `3000`                             |
| ingress.tls.enabled                | If true, enables SSL | `true`                    |
| ingress.tls.secretName             | Name of the secret used to terminate SSL traffic | `""` |
| ingress.annotations                | Ingress annotations | `{}` |
| livenessProbe.path                 | Path to access on the HTTP server on periodic probe of container liveness. | `/`                                |
| livenessProbe.scheme               | Scheme to access the HTTP server (HTTP or HTTPS). | `HTTP`                                |
| livenessProbe.initialDelaySeconds  | # of seconds after the container has started before liveness probes are initiated. | `15`                               |
| livenessProbe.timeoutSeconds       | # of seconds after which the liveness probe times out. | `15`                               |
| readinessProbe.path                | Path to access on the HTTP server on periodic probe of container readiness. | `/`                                |
| readinessProbe.scheme              | Scheme to access the HTTP server (HTTP or HTTPS). | `HTTP`                                |
| readinessProbe.initialDelaySeconds | # of seconds after the container has started before readiness probes are initiated. | `5`                                |
| readinessProbe.timeoutSeconds      | # of seconds after which the readiness probe times out. | `3`                                |
| worker.enabled                     |             | `true` |
| worker.securityContext.enabled     |          | `true` |
| worker.securityContext.fsGroup     |          | `1001` |
| worker.securityContext.runAsUser   |          | `1001` |
| worker.args                        |             | `bundle exec sidekiq` |
| worker.replicaCount                |             | `1`  |
| worker.revisionHistoryLimit        |             | `10`  |
| worker.sidekiq_alive.enabled       |             | `true` |
| worker.sidekiq_alive.livenessProbe.path            | Path to access on the HTTP server on periodic probe of container liveness. | `/`                                |
| worker.sidekiq_alive.livenessProbe.initialDelaySeconds | # of seconds after the container has started before liveness probes are initiated. | `15`                               |
| worker.sidekiq_alive.livenessProbe.timeoutSeconds  | # of seconds after which the liveness probe times out. | `15`                               |
| worker.sidekiq_alive.livenessProbe.port  | Port for sidekiq_alive | `7433`                               |
| worker.sidekiq_alive.readinessProbe.path           | Path to access on the HTTP server on periodic probe of container readiness. | `/`                                |
| worker.sidekiq_alive.readinessProbe.initialDelaySeconds | # of seconds after the container has started before readiness probes are initiated. | `5`   |
| worker.sidekiq_alive.readinessProbe.port  | Port for sidekiq_alive | `7433`                               |
| worker.sidekiq_alive.readinessProbe.timeoutSeconds | # of seconds after which the readiness probe times out. | `3`                                |
| postgresql.enabled                 |             | `false`                            |
| podDisruptionBudget.enabled        |             | `false`                            |
| podDisruptionBudget.maxUnavailable |             | `1`                            |
| podDisruptionBudget.minAvailable   | If present, this variable will configure minAvailable in the PodDisruptionBudget. :warning: if you have `replicaCount: 1` and `podDisruptionBudget.minAvailable: 1` `kubectl drain` will be blocked.              | `nil`                            |

NOTE: when providing ingress annotations, these must be givens as multiple --set commands OR set in a file

Example via set:
```
helm install . --dry-run --debug --set ingress.annotations."traefik\.ingress\.kubernetes\.io/redirect-entry-point"=\"https\" --set ingress.annotations."traefik\.ingress\.kubernetes\.io/redirect-permanent"=\"true\"
```

Optional, set it through a settings file and use -f. (See ingress_example_settings.yaml for example)
helm install . --dry-run --debug -f ingress_example_settings.yaml


Static site generated with Jekyll. See chart_site folder for details
