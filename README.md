# Gitlab k8s runner

GitLab Kubernetes Runner is an essential component for CI/CD pipelines in the GitLab ecosystem, especially when dealing with complex applications that require orchestration and scaling capabilities provided by Kubernetes. This runner allows you to execute your GitLab CI jobs on a Kubernetes cluster.

## Setup

> The official guide you can find in <https://docs.gitlab.com/runner/install/kubernetes/>.

The setup that is presented below is meant for cluster with at least 2 nodes with at least 16GB of memory each.
The configuration creates 2 nodes 4 jobs per node and each job can reach max 4GB of memory. This is important as building docker images inside docker (docker-in-docker) may require a lot of resources to build images for multiple architectures, and even 4GB might not be enough. For this one please pay attention how much RAM your nodes have available per job.

1. Make sure your cluster is up and running:

```sh
kubectl cluster-info

helm version  # helm 3.x or more
```

2. Get your Gitlab runner token

- Go to your gitlab in stance (e.g.: https://gitlab.com)
- As admin select in side menu -> **CI-CD** -> **Runners**
- Click "New instance runner"
- Add tags: `k8s, kubernetes, docker`
- Select the checkbox with "Run untagged jobs"
- Click Create runner
- In this step copy the registration token and store it securely it will not show up later after closing the page and we will need it later to our configuration

  For example (format: glrt-xxxxxxxxxxxxxxxxxxxxxx)

3. Install the helm chart in cluster

Login to your master node and execute:

```sh
# Add GitLab Helm repository
helm repo add gitlab https://charts.gitlab.io
helm repo update

# Create namespace
kubectl create namespace gitlab

# Verify
kubectl get namespace gitlab
```

4. Configure the runner with values from `gitlab-runner-values.yaml`

- in `runnerToken` change the token to your token from step 2

> [!TIP]
> Before running you can also check the details for more helm chart values with command:
>
> ```sh
> # show available values
> helm inspect values gitlab/gitlab-runner
>
> # or save values to a sample file
> helm inspect values gitlab/gitlab-runner > custom-values.yaml
> ```

Check the available versions of `gitlab-runner` to install by:

```sh
helm search repo -l gitlab/gitlab-runner
```

5. Create system user

Creating user is optional, but may help with creating default directory used by gitlab-runner.

```sh
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```

5. Install helm chart

You can also add specific version with `--version 0.84.2`, otherwise it will use latest one.

```sh
helm install gitlab-runner \
  --namespace gitlab \
  -f gitlab-runner-values.yaml \
  gitlab/gitlab-runner
```

6. Verify if all works

```sh
helm list --namespace gitlab
```

Execute the sample job in gitlab pipeline in order to see job is picked successfully by runner.

## Need Upgrade

```sh
helm upgrade gitlab-runner \
  --namespace gitlab \
  -f gitlab-runner-values.yaml \
  gitlab/gitlab-runner
```

## Uninstall

```sh
helm delete --namespace gitlab gitlab-runner
```
