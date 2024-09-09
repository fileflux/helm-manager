# FileFlux Manager Helm Chart

This is a Helm chart for deploying **FileFlux Manager** on a Kubernetes cluster

## Prerequisites
- AWS Infrastructure setup using Terraform located [here](https://github.com/fileflux/infra)
- Helm 3.0+ (for manual deployment) [guide](https://helm.sh/docs/intro/install/)
- CockroachDB Helm chart (for manual deployment) located [here](https://github.com/fileflux/helm-cockroachdb)
- A namespace where the chart will be deployed (default is 's3' and is created by the Terraform IaC code located [here](https://github.com/fileflux/infra))


## Working
While using in the FileFlux project, the FileFlux Manager Helm chart is automatically deployed using ArgoCD, which is deployed via the Terraform IaC code in the infra repository located [here](https://github.com/fileflux/infra). The chart is configured to deploy the FileFlux Manager application with the necessary configurations and dependencies automatically. Any update to the chart should be pushed to the helm-manager repository, which is synced with the ArgoCD application in the Kubernetes cluster.

## Files
The FileFlux Manager Helm chart consists of the following components:
- **deployment.yaml**: Creates and manages a Deployment resource for the FileFlux Manager application with the necessary configurations.
- **clusterip.yaml**: Exposes the FileFlux Manager application within the Kubernetes cluster using a ClusterIP service.
- **serviceaccount.yaml**: Creates a Service Account that provides the necessary permissions for the FileFlux Manager application to access resources within the cluster and AWS services
- **secret.yaml**: Deploys SecretProviderClass object for accessing secrets in AWS Secrets Manager and deploying them as Kubernetes secrets.
- **hpa.yaml**: Deploys a Horizontal Pod Autoscaler to scale the FileFlux Manager application based on CPU utilization.
- **lifecycle.yaml**: Creates a Deployment, Service Account, Cluster Role, and Cluster Role Binding for the pod labeler application that adds labels to pods and nodes based on the worker node's instance ID.
- **service.yaml**: Creates a LoadBalancer service for the FileFlux Manager application to expose it to the internet via a custom domain name (current default s3.lokesh.cloud).

## Configuration

The following table lists the configurable parameters of the s3manager chart and their default values.

| Key                               | Default Value                                                    | Description                                                        |
|------------------------------------|------------------------------------------------------------------|--------------------------------------------------------------------|
| `replicaCount`                     | `1`                                                              | Number of replicas for the deployment                              |
| `namespace`                        | `s3`                                                             | Namespace for the deployment                                       |
| `image.repository`                 | `chlokesh1306/s3manager`                                          | Docker image repository for the manager                            |
| `serviceAccountName`               | `db-access`                                                      | Name of the service account                                        |
| `serviceAccount.create`            | `True`                                                           | Specifies whether a service account should be created              |
| `serviceAccount.automount`         | `True`                                                           | Specifies whether to automount service account                     |
| `service.type`                     | `ClusterIP`                                                      | Kubernetes service type                                            |
| `service.port`                     | `8080`                                                           | Port for the service                                               |
| `resources.limits.cpu`             | `500m`                                                           | CPU resource limit                                                 |
| `resources.limits.memory`          | `512Mi`                                                          | Memory resource limit                                              |
| `resources.requests.cpu`           | `250m`                                                           | CPU resource request                                               |
| `resources.requests.memory`        | `256Mi`                                                          | Memory resource request                                            |
| `livenessProbe.initialDelaySeconds`| `30`                                                             | Initial delay before liveness probe                                |
| `livenessProbe.periodSeconds`      | `10`                                                             | Period between liveness probes                                     |
| `readinessProbe.initialDelaySeconds`| `30`                                                             | Initial delay before readiness probe                               |
| `hpa.enabled`                      | `True`                                                           | Enable Horizontal Pod Autoscaler                                   |
| `hpa.minReplicas`                  | `3`                                                              | Minimum number of replicas                                         |
| `hpa.maxReplicas`                  | `5`                                                              | Maximum number of replicas                                         |
| `hpa.targetCPUUtilizationPercentage`| `80`                                                             | Target CPU utilization for HPA                                     |
| `volumes[0].name`                  | `secrets-store-inline`                                            | Name of the secrets store inline volume                            |
| `volumeMounts[0].name`             | `secrets-store-inline`                                            | Name of the secrets store inline volume mount                      |
| `volumeMounts[0].mountPath`        | `/mnt/secrets-store`                                             | Mount path for the secrets store inline volume                     |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
helm install s3manager ./path/to/release -n s3 --set replicaCount=2
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart:

```bash
helm install s3manager ./path/to/release -n s3 -f values.yaml
```

## Troubleshooting
If you encounter issues:

Check the status of the pods:
```bash
kubectl get pods -n s3 -l app=s3manager
```

View the logs of the manager application:
```bash
kubectl logs -n s3 -l app=s3manager -f
```

Access the shell of the manager pod for debugging:
```bash
kubectl exec -it <s3manager-pod-name> -n s3 -- sh
```

## Installing the Chart (manual)
Download the latest release from the following GitHub repository: 

```bash
https://github.com/fileflux/helm-manager
```

Install the chart with the release name `s3manager`:

```bash
helm install -n s3 s3manager ./path/to/release
```

## Upgrading the Chart (manual)

To upgrade the `s3manager` release:

```bash
helm upgrade s3manager ./path/to/release -n s3
```

Ensure the chart version is updated in the `Chart.yaml` before upgrading.

## Uninstalling the Chart (manual)

To uninstall/delete the `s3manager` helm release:

```bash
helm uninstall s3manager -n s3
```

The command removes all the Kubernetes components associated with the chart.

## Additional Information

This helm chart is primarly designed to be used in the FileFlux project and is automatically deployed using ArgoCD. The chart can be used independently by providing the necessary configurations and dependencies however, it is recommended deploy and manage this chart via ArgoCD in the FileFlux project for seamless integration and deployment. Note that it is necessary to have the AWS infrastructure setup using Terraform and the CockroachDB Helm chart deployed before deploying this chart.