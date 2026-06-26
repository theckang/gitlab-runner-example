# GitLab Runner on OpenShift

This is an example of running GitLab Runner on OpenShift. It will trigger builds of a HTML application in OpenShift using the GitLab Runner agent.

## Prerequisites

1. OpenShift 4+ Cluster
2. Running GitLab Instance
3. Quay (or registry of your choice)

## Steps

### Quay

1. Login to Quay
1. Create a Repository
1. Go to Repository Settings, create a Robot Account, and set Permissions to `Write`
1. Select the Robot Account and take note of the username and token

### GitLab

1. Login to GitLab
2. Create a new project
3. Import project using this repository
4. In the project, navigate to Settings -> CI/CD -> Runners -> Expand -> New project runner

In this view, **make sure** to add the openshift tag and create the runner.
Take note of the Runner Token.

5. In the project, navigate to Settings -> CI/CD -> Variables -> Expand -> Add variable

In this view, **make sure** to create `Masked` variables. 

Create all three variables

```bash
QUAY_IMAGE_NAME=quay.io/yourorganization/yourrepository
QUAY_ROBOT_USER=yourServiceAccountUser
QUAY_ROBOT_PASS=yourServiceAccountToken
```

### OpenShift

1. Login to your OpenShift Cluster.

2. Create a project

```bash
oc new-project gitlab-runner-test
```

3. Follow [these instructions](https://docs.gitlab.com/runner/install/operator/) to install the GitLab Runner Operator 

4. Assign the `anyuid` SCC to the Gitlab Runner build agents

> Note: `nonroot` does not work because the SETUID and SETGUID linux capabilities are used by buildah in the build.
> Additionally, the `SETFCAP` linux capability must be allowed in the security context constraint.

```bash
oc adm policy add-scc-to-user anyuid -z gitlab-runner-app-sa -n gitlab-runner-test
```

5. Edit the `secret.yaml` file with your Runner Token. Create the secret.

```bash
oc create -f configs/secret.yaml -n gitlab-runner-test
```

6. Edit the `config.toml` file with your gitlab instance url. Create the configmap.

> Note: `SETFCAP` linxux capability is used in this configuration for buildah

```bash
oc create configmap my-runner-config --from-file=configs/config.toml
```

7. Edit the `runner.yaml` file with your gitlab instance url. Create the runner.

```bash
oc create -f configs/runner.yaml -n gitlab-runner-test
```

### Test

1. Navigate to GitLab

1. In the project, navigate to Build -> Jobs

1. You should see a build here that is complete. 

> Note: To trigger a new build, hit the circle of two arrows to "Run again". Or, you can push a change to the Gitlab repository.
