---
@title[State of the Art of GitOps with Helm]

## State of the Art of GitOps with Helm

Highly customizable GitOps pipeline built with:

- [`helmfile`](https://github.com/roboll/helmfile)
- [`brigade`](https://github.com/brigadecore/brigade)

---
@title[Prior Arts]

## Prior Arts

1. **Weaveworks [Flux](https://github.com/weaveworks/flux) Operator + Raw K8s manifests OR Helm Chart**:
2. **[Argo CD](https://github.com/argoproj/argo-cd) + K8s Manifests OR Helm Chart OR Helmfile**:

---
@title[Flux]

### Flux

> ![Flux Deployment Pipeline](https://github.com/weaveworks/flux/raw/master/site/images/deployment-pipeline.png)
> https://github.com/weaveworks/flux

- `flux` fetches git commits and reconsile K8s resources.
- `helm-operator` reconciles `HelmRelease` resources to reconsile K8s resources.

---
@title[Argo CD]

### Argo CD

> ![Argo CD Architecture](https://argoproj.github.io/argo-cd/assets/argocd_architecture.png)
> https://argoproj.github.io/argo-cd/#architecture

- `argocd` fetches git commits and reconcile K8s resources
- "argocd config management plugin" to plug-in any K8s manifests builder(ex. ksonnet, kubecfg, etc.)

PLUG: Wanna declaratively manage Argo CD projects? Use [the community Helm chart](https://github.com/chatwork/charts/tree/master/argoproj-crd)

---
@title[Problems]

## Problems

1. Limited Customizability
2. High Number of Total Moving-Parts

---
@title[Limited Customizability]

### Limited Customizability

`flux` and `argocd` has limited extension points. For example, `argocd` has various "hooks" but you stale once get to think "Oh, I wanna argocd to use `helm upgrade` to manage my app as a helm release!"

---
@title[High Nummber of Total Moving-Parts]

### High Nummber of Total Moving-Parts

Let think about building an end-to-end CI/CD pipeline that leverages GitOps.

---
@title[Existing Solution]

## Existing Solution

- Use any CI system for CI(lint, diff, test): Travis, CircleCI, Concourse, Jenkins, Argo CI, ...
- Use any CD system for CD(deploy/sync/reconcile): Flux, Argo CD, Spinnaker, ...

Go head if you have a big team.

But what if your have only a handful of folks to maintain the CI and CD pipeline?

---
@title[Goal]

## Goal

Can't we have a single versatile system that handles both CI and CD?

---
@title[Issues]

## Issues

---
@title[Every aspect of CI/CD pipelines should be customizable]

### Every aspect of CI/CD pipelines should be customizable

Run any workflow composed of scripts and K8s pods programmed with a turing-complete language.

---
@title[Single Platform that is capable of handling both CI and CD use-cases]

### Single Platform that is capable of handling both CI and CD use-cases

Divergence between CIOps and GitOps should be small, so that we can easily and/or gradually migrate from CIOps(push-based) to GitOps(pull-based), or vice versa.

Build upon an unviersal workflow engine or alike that is capable of achieving both CIOps and GitOps.

---
@title[Be declarative!]

### Be declarative!

Cluster operations are hard and the life is short. You should focus on expressing what you want to do, not how.

Leverage declarative management. Rely on K8s and K8s operators to reconsile cluster states to the desired states. You focus on writing declarative specs of your K8s apps.

---
@title[Single Tool?]

## Single Tool?

- Single Tool that is capable of both local development and remote production usages?
- Is that really a thing? You see...

---
@title[Design]

## Design

- [brigade](https://github.com/brigadecore/brigade)(an open, event-driven K8s scripting platform) as an universal workflow engine that runs both CI and GitOps/CD pipelines
- [helmfile](https://github.com/roboll/helmfile) to declaratively manage all the apps on K8s
- [variant](https://github.com/mumoshu/variant) as a task runner. Alternatively use `make` or whatever.

The only dependencies are GitHub and Kubernetes

---
@title[Install]

## Install

---
@title[Prereqs.]

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

---
@title[Bootstrap]

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

---
@title[Local Development]

### Local Development

Run:

```
$ git checkout -b change-blah
$ $EDITOR helmfile.yaml
$ helmfile apply
```

---
@title[Remote Deployment]

### Remote Deployment

Run:

```
$ git add helmfile.yaml && \
  git commit -m 'Change blah' && \
  git push origin master
$ hub pull-request
```

---
@title[Review]

### Review

Open the PR URL printed by `hub` and see:

---
@title[Build Status]

### Build Status

TBD

---
@title[Merge]

### Merge

Merge the pull request into `master`, so that the `brigade` pulls the commit and applies it with `helmfile`.

Voila! You've implemented GitOps.

---
@title[Implementation]

### Implementation

The `brigade` script looks like:

```javascript
// TODO
```

---
@title[Fin.]

## Fin.

https://gitpitch.com/mumoshu/helmfile-gitops
