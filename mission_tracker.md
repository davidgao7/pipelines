# Issue want to solve:
- [#7339 Component Visualizations are not shown in Run Output](https://github.com/kubeflow/pipelines/issues/7339)

#  Progress solving this issue:
> This is the place where I track the progressing solving this issue, along with valuable sources

## Building the virtual environment for running kubeflow pipelines
- <font color="red">remember to start the 3 container to continue working</font>

```bash
docker start 843cca50c5aa 2fb4eae3b2f7 6178545dbb5b
```
### Local Quickstart ([reference](https://www.deploykf.org/guides/local-quickstart/))

####  create kubernetes cluster
    
  ```zsh
  # NOTE: this will change your kubectl context to the new cluster
  k3d cluster create "deploykf" \
  --image "rancher/k3s:v1.26.9-k3s1"
  ```
    
### Install k9s to monitor the pods
    
    
  ```bash
  brew install k9s
  ```
    
  - show k9s dashboard
    
  ```zsh
  k9s
  ```

  ### install ArgoCD and  deployKF ArgoCD Plugin to the cluster
  ```zsh
  # clone the deploykf repo
  # NOTE: we use 'main', as the latest plugin version always lives there
    git clone -b main https://github.com/deployKF/deployKF.git ./deploykf

  # ensure the script is executable
    chmod +x ./deploykf/argocd-plugin/install_argocd.sh

  # run the install script
  # WARNING: this will install into your current kubectl context
    bash ./deploykf/argocd-plugin/install_argocd.sh 
  ```

### create ArgoCD Applications
 > YAML-based config, xxx-values.yaml is for configuring large number of values(more than 1500) , deployKF supports `in-place upgrade`, we can start with a few important config first , then grow values file over time.

- deployKF provide sample yaml file to twick.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: deploykf-app-of-apps
  namespace: argocd
  labels:
    app.kubernetes.io/name: deploykf-app-of-apps
    app.kubernetes.io/managed-by: deploykf
  spec:
    project: "default"
    source:
      repoURL: "https://github.com/deployKF/deployKF.git"
      targetRevision: "v0.1.4"
      path: "~/argocd-plugin"
  
  # plugin configuration
  plugin:
    name: "deploykf"
    parameters:
      # declear the deployKF generator version
      # available versions: https://github.com/deployKF/deployKF/releases
      - name: "source_version"
        string: "0.1.4"

      ## paths to values files within the `repoURL` repository
      ##  - the values in these files are merged, with later files taking precedence
      ##  - we strongly recommend using 'sample-values.yaml' as the base of your values
      ##    so you can easily upgrade to newer versions of deployKF
      ##
      - name: "values_files"
        array:
          - "./sample-values.yaml"
  ```

### Create k8s locally
  ```zsh
  # clone the deploykf repo
# NOTE: we use 'main', as the latest script always lives there
git clone -b main https://github.com/deployKF/deployKF.git ./deploykf

# ensure the script is executable
chmod +x ./deploykf/scripts/sync_argocd_apps.sh

# run the script
bash ./deploykf/scripts/sync_argocd_apps.sh
  ```

![argoCD](k8s_running.png)

---
> now k8s is running, we can deploy different pods.

### deploy kubeFlow pipline to locak k8s (v1 and v2 are same)

  [k3s setup](https://www.kubeflow.org/docs/components/pipelines/v1/installation/localcluster-deployment/)

  1. To deploy the Kubeflow Pipelines, run the following commands

  ```zsh
  # env/platform-agnostic-pns hasn't been publically released, so you will install it from master
export PIPELINE_VERSION=2.0.5
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"
kubectl wait --for condition=established --timeout=60s crd/applications.app.k8s.io
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/platform-agnostic-pns?ref=$PIPELINE_VERSION"
  ```

 - version need: piplinev2, so use the official guide
