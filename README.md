# helmfile-gitops

This project is a demonstration of a highly customizable GitOps pipeline build with [`helmfile`](https://github.com/roboll/helmfile) and [`brigade`](https://github.com/brigadecore/brigade).

## Design

### Every aspect of CI/CD pipelines should be customizable

Run any workflow composed of scripts and K8s pods programmed with a turing-complete language.

### Divergence between CIOps and GitOps should be small

So that we can easily and/or gradually migrate from CIOps(push-based) to GitOps(pull-based), or vice versa.

Build upon an unviersal workflow engine or alike that is capable of achieving both CIOps and GitOps.

### Be declarative!

Cluster operations are hard and the life is short. You should focus on expressing what you want to do, not how.

Leverage declarative management. Rely on K8s and K8s operators to reconsile cluster states to the desired states. You focus on writing declarative specs of your K8s apps.

## Implementations

- [brigade](https://github.com/brigadecore/brigade)(an open, event-driven K8s scripting platform) as an universal workflow engine that runs both CI and GitOps/CD pipelines
- [helmfile](https://github.com/roboll/helmfile) to declaratively manage all the apps on K8s
- [variant](https://github.com/mumoshu/variant) as a task runner. Alternatively use `make` or whatever.

The only dependencies are GitHub and Kubernetes

## Install

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

## Usage

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
