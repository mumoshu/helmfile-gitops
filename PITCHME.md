# helmfile-gitops

This project is a demonstration of a highly customizable GitOps pipeline built with [`helmfile`](https://github.com/roboll/helmfile) and [`brigade`](https://github.com/brigadecore/brigade).

---
@title[Prior Arts]

1. **Weaveworks [Flux](https://github.com/weaveworks/flux) Operator + Raw K8s manifests OR Helm Chart**:
2. **[Argo CD](https://github.com/argoproj/argo-cd) + K8s Manifests OR Helm Chart OR Helmfile**:

### Flux

> ![Flux Deployment Pipeline](https://github.com/weaveworks/flux/raw/master/site/images/deployment-pipeline.png)
> https://github.com/weaveworks/flux

- `flux` fetches git commits and reconsile K8s resources.
- `helm-operator` reconciles `HelmRelease` resources to reconsile K8s resources.

### Argo CD

> ![Argo CD Architecture](https://argoproj.github.io/argo-cd/assets/argocd_architecture.png)
> https://argoproj.github.io/argo-cd/#architecture

- `argocd` fetches git commits and reconcile K8s resources
- "argocd config management plugin" to plug-in any K8s manifests builder(ex. ksonnet, kubecfg, etc.)

PLUG: Wanna declaratively manage Argo CD projects? Use [the community Helm chart](https://github.com/chatwork/charts/tree/master/argoproj-crd)

---
@title[Problem]

### Sub-optimal Customizability

`flux` and `argocd` has limited extension points. For example, `argocd` has various "hooks" but you stale once get to think "Oh, I wanna argocd to use `helm upgrade` to manage my app as a helm release!"

### High Nummber of Total Moving-Parts

Let think about building an end-to-end CI/CD pipeline that leverages GitOps.

A common setup would look like:

- Use any CI system for CI(lint, diff, test): Travis, CircleCI, Concourse, Jenkins, Argo CI, ...
- Use any CD system for CD(deploy/sync/reconcile): Flux, Argo CD, Spinnaker, ...

Go head if you have a big team. But what if your have only a handful of folks to maintain the CI and CD pipeline?

Can't we have a single versatile system that handles both CI and CD?

---
@title[Goals]

### Every aspect of CI/CD pipelines should be customizable

Run any workflow composed of scripts and K8s pods programmed with a turing-complete language.

### Single Platform that is capable of handling both CI and CD use-cases

Divergence between CIOps and GitOps should be small, so that we can easily and/or gradually migrate from CIOps(push-based) to GitOps(pull-based), or vice versa.

Build upon an unviersal workflow engine or alike that is capable of achieving both CIOps and GitOps.

### Be declarative!

Cluster operations are hard and the life is short. You should focus on expressing what you want to do, not how.

Leverage declarative management. Rely on K8s and K8s operators to reconsile cluster states to the desired states. You focus on writing declarative specs of your K8s apps.

### Single Tool that is capable of both local development and remote production usages.

---
@title[Design]

- [brigade](https://github.com/brigadecore/brigade)(an open, event-driven K8s scripting platform) as an universal workflow engine that runs both CI and GitOps/CD pipelines
- [helmfile](https://github.com/roboll/helmfile) to declaratively manage all the apps on K8s
- [variant](https://github.com/mumoshu/variant) as a task runner. Alternatively use `make` or whatever.

The only dependencies are GitHub and Kubernetes

---
@title[Install]

1. Grab and install the latest release of [variant](https://github.com/mumoshu/variant)
2. Create a GitHub App for Brigade following [this guide](https://github.com/brigadecore/brigade-github-app/blob/c04ea3fa28f2e0a3a64d74131bfef1fe7698355a/README.md#1-create-a-github-app)
3. Set envvars:

   ```
   export NGROK_TOKEN=<Your ngrok token shown in https://dashboard.ngrok.com/get-started>
   export BRIGADE_GITHUB_APP_KEY=<Local path to the private key file for TLS client auth from the step 2>
   export BRIGADE_GITHUB_APP_ID=<App ID from the step 2>
   export BRIGADE_PROJECT_SECRET=<Webhook (shared) secret from the step 2>
   export SSH_KEY=$HOME/.ssh/id_rsa (Or whatever ssh private key you want Brigade to use while git-cloning private Git repos)
   export GITHUB_TOKEN=<Used for updating commit/pull request statuses>
   ```

---
@title[Usage]

```
$ variant
Usage:
  variant [command]

Available Commands:
  build       Create a single executable from the Variantfile
  diff
  env         Print currently selected environment
  help        Help about any command
  init        Create a Variant command
  polaris     Open polaris dashboard in your browser

Polaris: https://github.com/reactiveops/polaris

  sync
  tools
  version     Print the version number of this command

Flags:
  -C, --color                Colorize output (default true)
  -c, --config-file string   Path to config file
  -h, --help                 help for variant
      --logtostderr          write log messages to stderr (default true)
  -o, --output string        Output format. One of: json|text|bunyan (default "text")
  -v, --verbose              verbose output

Use "variant [command] --help" for more information about a command.
```
