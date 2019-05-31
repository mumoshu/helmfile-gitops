## State of the Art of GitOps with Helm

Highly customizable GitOps pipeline built with:

- [`helmfile`](https://github.com/roboll/helmfile)
- [`brigade`](https://github.com/brigadecore/brigade)

Appendix: The end of Kustomize vs Helm argument

## Prior Arts

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

## Problems

1. Limited Customizability
2. High Number of Total Moving-Parts

### Limited Customizability

`flux` and `argocd` has limited extension points. For example, `argocd` has various "hooks" but you stale once get to think "Oh, I wanna argocd to use `helm upgrade` to manage my app as a helm release!"

### High Nummber of Total Moving-Parts

Let think about building an end-to-end CI/CD pipeline that leverages GitOps.

## Existing Solution

Combo of:

**CI (lint, diff, test on PUSH)**:
Travis, CircleCI, Concourse, Jenkins, Argo CI, ...

**CD (deploy/sync/reconcile on PULL)**:
Flux, Argo CD, Spinnaker, ...

## Why Not?

Go head if you have a big team.

What if your have only a handful of folks to maintain the CI and CD pipeline?

## Goal

I wanna have a single system that handles both CI and CD

## Issues

### Every aspect of CI/CD pipelines should be customizable

Run any workflow composed of scripts and K8s pods programmed with a turing-complete language.

### Single Platform that is capable of handling both CI and CD use-cases

Divergence between CIOps and GitOps should be small, so that we can easily and/or gradually migrate from CIOps(push-based) to GitOps(pull-based), or vice versa.

Build upon an unviersal workflow engine or alike that is capable of achieving both CIOps and GitOps.

### Be declarative!

Cluster operations are hard and the life is short. You should focus on expressing what you want to do, not how.

Leverage declarative management. Rely on K8s and K8s operators to reconsile cluster states to the desired states. You focus on writing declarative specs of your K8s apps.

## Single Tool?

- Single Tool that is capable of both local development and remote production usages?
- Is that really a thing? You see...

## Design

- [brigade](https://github.com/brigadecore/brigade)(an open, event-driven K8s scripting platform) as an universal workflow engine that runs both CI and GitOps/CD pipelines
- [helmfile](https://github.com/roboll/helmfile) to declaratively manage all the apps on K8s
- [variant](https://github.com/mumoshu/variant) as a task runner. Alternatively use `make` or whatever.

The only dependencies are GitHub and Kubernetes

## Install

### Prereqs.

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

### Bootstrap

Run:

```
$ helmfile apply
```

This will install:

- Brigade Server
- Brigade GitHub App
- Brigade Project
- In-Cluster Ngrok Tunnel (GitHub Webhooks to Brigade GitHub App
- Other apps for demonstration purpose

### Local Development

Run:

```
$ git checkout -b change-blah
$ $EDITOR helmfile.yaml
$ helmfile apply
```

### Remote Deployment

Run:

```
$ git add helmfile.yaml && \
  git commit -m 'Change blah' && \
  git push origin master
$ hub pull-request
```

### Review

Open the PR URL printed by `hub` and see:

### Build Status

TBD

### Merge

Merge the pull request into `master`, so that the `brigade` pulls the commit and applies it with `helmfile`.

Voila! You've implemented GitOps.

### Implementation

The `brigade` script looks like:

```javascript
// TODO
```

## Fin.

https://gitpitch.com/mumoshu/helmfile-gitops

## One More Thing

"The end of the "Kustomize vs Helm argument"

### Everyone Does This

- `helm template mychart | kubectl apply -f`
- `helm template mychart --outputs-dir manifests/ && (kustomize build | kubectl apply -f -)`

### Don't use `kubectl apply -f`

When you want:

- `helm diff`: Preview changes before apply
- `helm test`: Run tests included in the chart
- `helm stauts`: List helm-managed resources and the installation note
- `helm get values`: Which settings I used when installing this?

### We Can Do Better

- (Optionally) Generate K8s manifests from Helm chart
- Patch K8s manifests with Kustomize (JSON Patch and Strategic-Merge Patch available)
- Install the patched manifests with Helm

### Example: helm-x

Yet Another Shamelss Plug: https://github.com/mumoshu/helm-x

```
$ helm x diff myapp PATH/TO/MANIFESTS_OR_CHART --version 1.2.4 \
  -f values.yaml \
  --strategic-merge-patch path/to/strategicmerge.patch.yaml \
  --jsonpatch path/to/json.patch.yaml

$ helm x upgrade --install  myapp PATH/TO/MANIFESTS_OR_CHART --version 1.2.4 \
  -f values.yaml \
  --strategic-merge-patch path/to/strategicmerge.patch.yaml \
  --jsonpatch path/to/json.patch.yaml
```
