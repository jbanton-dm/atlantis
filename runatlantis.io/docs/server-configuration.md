# Server Configuration
This page explains how to configure the `atlantis server` command.

Configuration to `atlantis server` can be specified via command line flags,
 environment variables, a config file or a mix of the three.

[[toc]]

## Environment Variables
All flags can be specified as environment variables.

1. Take the flag name, ex. `--gh-user`
1. Ignore the first `--` => `gh-user`
1. Convert the `-`'s to `_`'s => `gh_user`
1. Uppercase all the letters => `GH_USER`
1. Prefix with `ATLANTIS_` => `ATLANTIS_GH_USER`

::: warning NOTE
The flag `--atlantis-url` is set by the environment variable `ATLANTIS_ATLANTIS_URL` **NOT** `ATLANTIS_URL`.
:::

## Config File
All flags can also be specified via a YAML config file.

To use a YAML config file, run `atlantis server --config /path/to/config.yaml`.

The keys of your config file should be the same as the flag names, ex.
```yaml
gh-token: ...
log-level: ...
```

::: warning
The config file you pass to `--config` is different from the `--repo-config` file.
The `--config` config file is only used as an alternate way of setting `atlantis server` flags.
:::

## Precedence
Values are chosen in this order:
1. Flags
1. Environment Variables
1. Config File


## Flags
* ### `--allow-fork-prs`
  ```bash
  atlantis server --allow-fork-prs
  ```
  Respond to pull requests from forks. Defaults to `false`.
  
  :::warning SECURITY WARNING
  Potentially dangerous to enable
  because if attackers can create a pull request to your repo then they can cause Atlantis
  to run arbitrary code. This can happen because
  Atlantis will automatically run `terraform plan`
  which can run arbitrary code if given a malicious Terraform configuration.
  :::
  
* ### `--allow-repo-config`
  <Badge text="Deprecated" type="warn"/>
  ```bash
  atlantis server --allow-repo-config
  ```
  This flag is deprecated. It allows all repos to use all restricted
  `atlantis.yaml` keys. See [Repo Level Atlantis.yaml](repo-level-atlantis-yaml.html) for more details.
  
  Instead of using this flag, create a server-side `--repo-config` file:
  ```yaml
  # repos.yaml
  repos:
  - id: /.*/
    allowed_overrides: [apply_requirements, workflow]
    allow_custom_workflows: true
  ```
  Or use
  ```bash
  --repo-config-json='{"repos":[{"id":"/.*/", "allowed_overrides":["apply_requirements","workflow"], "allow_custom_workflows":true}]}'
  ````
  
  ::: warning SECURITY WARNING
  This setting enables pull requests to run arbitrary code on the Atlantis server.
  Only enable in trusted settings.
  :::

* ### `--atlantis-url`
  ```bash
  atlantis server --atlantis-url="https://my-domain.com:9090/basepath"
  ```
  Specify the URL that Atlantis is accessible from. Used in the Atlantis UI
  and in links from pull request comments. Defaults to `http://$(hostname):$port`
  where `$port` is from the [`--port`](#port) flag. Supports a basepath if you're hosting Atlantis under a path.

* ### `--automerge`
  ```bash
  atlantis server --automerge
  ```
  Automatically merge pull requests after all plans have been successfully applied.
  Defaults to `false`. See [Automerging](automerging.html) for more details.

* ### `--bitbucket-base-url`
  ```bash
  atlantis server --bitbucket-base-url="http://bitbucket.corp:7990/basepath"
  ```
  Base URL of Bitbucket Server (aka Stash) installation. Must include
  `http://` or `https://`. If using Bitbucket Cloud (bitbucket.org), do not set. Defaults to
  `https://api.bitbucket.org`.
  
* ### `--bitbucket-token`
  ```bash
  atlantis server --bitbucket-token="token"
  # or (recommended)
  ATLANTIS_BITBUCKET_TOKEN='token' atlantis server
  ```
  Bitbucket app password of API user.

* ### `--bitbucket-user`
  ```bash
  atlantis server --bitbucket-user="myuser"
  ```
  Bitbucket username of API user.
  
* ### `--bitbucket-webhook-secret`
  ```bash
  atlantis server --bitbucket-webhook-secret="secret"
  # or (recommended)
  ATLANTIS_BITBUCKET_WEBHOOK_SECRET='secret' atlantis server
  ```
  Secret used to validate Bitbucket webhooks. Only Bitbucket Server supports webhook secrets.
  For Bitbucket.org, see [Security](security.html#bitbucket-cloud-bitbucket-org) for mitigations.
  
  ::: warning SECURITY WARNING
  If not specified, Atlantis won't be able to validate that the incoming webhook call came from Bitbucket.
  This means that an attacker could spoof calls to Atlantis and cause it to perform malicious actions.
  :::
  
* ### `--checkout-strategy`
  ```bash
  atlantis server --checkout-strategy="<branch|merge>"
  ```
  How to check out pull requests.
  Defaults to `branch`. See [Checkout Strategy](checkout-strategy.html) for more details.
  
* ### `--config`
  ```bash
  atlantis server --config="my/config/file.yaml"
  ```
  YAML config file where flags can also be set. See [Config File](#config-file) for more details.

* ### `--data-dir`
  ```bash
  atlantis server --data-dir="path/to/data/dir"
  ```
  Directory where Atlantis will store its data. Will be created if it doesn't exist.
  Defaults to `~/.atlantis`. Atlantis will store its database, checked out repos, Terraform plans and downloaded
  Terraform binaries here. If Atlantis loses this directory, [locks](locking.html)
  will be lost and unapplied plans will be lost.
  
* ### `--default-tf-version`
  ```bash
  atlantis server --default-tf-version="v0.12.0"
  ```
  Terraform version to default to. Will download to `<data-dir>/bin/terraform<version>`
  if not in `PATH`. See [Terraform Versions](terraform-versions.html) for more details.

* ### `--gh-hostname`
  ```bash
  atlantis server --gh-hostname="my.github.enterprise.com"
  ```
  Hostname of your GitHub Enterprise installation. If using [Github.com](https://github.com),
  don't set. Defaults to `github.com`.
  
* ### `--gh-token`
  ```bash
  atlantis server --gh-token="token"
  # or (recommended)
  ATLANTIS_GH_TOKEN='token' atlantis server
  ```
  GitHub token of API user.

* ### `--gh-user`
  ```bash
  atlantis server --gh-user="myuser"
  ```
   GitHub username of API user.
  
* ### `--gh-webhook-secret`
  ```bash
  atlantis server --gh-webhook-secret="secret"
  # or (recommended)
  ATLANTIS_GH_WEBHOOK_SECRET='secret' atlantis server
  ```
  Secret used to validate GitHub webhooks (see [https://developer.github.com/webhooks/securing/](https://developer.github.com/webhooks/securing/)).
  
  ::: warning SECURITY WARNING
  If not specified, Atlantis won't be able to validate that the incoming webhook call came from GitHub.
  This means that an attacker could spoof calls to Atlantis and cause it to perform malicious actions.
  :::

* ### `--gitlab-hostname`
  ```bash
  atlantis server --gitlab-hostname="my.gitlab.enterprise.com"
  ```
  Hostname of your GitLab Enterprise installation. If using [Gitlab.com](https://gitlab.com),
  don't set. Defaults to `gitlab.com`.
  
* ### `--gitlab-token`
  ```bash
  atlantis server --gitlab-token="token"
  # or (recommended)
  ATLANTIS_GITLAB_TOKEN='token' atlantis server
  ```
  GitLab token of API user.

* ### `--gitlab-user`
  ```bash
  atlantis server --gitlab-user="myuser"
  ```
   GitLab username of API user.
  
* ### `--gitlab-webhook-secret`
  ```bash
  atlantis server --gh-webhook-secret="secret"
  # or (recommended)
  ATLANTIS_GITLAB_WEBHOOK_SECRET='secret' atlantis server
  ```
  Secret used to validate GitLab webhooks.
  
  ::: warning SECURITY WARNING
  If not specified, Atlantis won't be able to validate that the incoming webhook call came from GitLab.
  This means that an attacker could spoof calls to Atlantis and cause it to perform malicious actions.
  :::
  
* ### `--help`
  ```bash
  atlantis server --help
  ```
  View help.

* ### `--log-level`
  ```bash
  atlantis server --log-level="<debug|info|warn|error>"
  ```
  Log level. Defaults to `info`.

* ### `--port`
  ```bash
  atlantis server --port=8080
  ```
  Port to bind to. Defaults to `4141`.
  
* ### `--repo-config`
  ```bash
  atlantis server --repo-config="path/to/repos.yaml"
  ```
  Path to a YAML server-side repo config file. See [Server Side Repo Config](server-side-repo-config.html).

* ### `--repo-config-json`
  ```bash
  atlantis server --repo-config-json='{"repos":[{"id":"/.*/", "apply_requirements":["mergeable"]}]}'
  ```
  Specify server-side repo config as a JSON string. Useful if you don't want to write a config file to disk.
  See [Server Side Repo Config](server-side-repo-config.html) for more details.
  
  ::: tip
  If specifying a [Workflow](custom-workflows.html#reference), [step](custom-workflows.html#step)'s
  can be specified as follows:
  ```json
  {
    "repos": [],
    "workflows": {
      "custom": {
        "plan": {
          "steps": [
            "init",
            {
              "plan": {
                "extra_args": ["extra", "args"]
              }
            },
            {
              "run": "my custom command"
            }
          ]
        }
      }
    }
  }
  ```
  :::
  
* ### `--repo-whitelist`
  ```bash
  # NOTE: Use single quotes to avoid shell expansion of *.
  atlantis server --repo-whitelist='github.com/myorg/*'
  ```
  Atlantis requires you to specify a whitelist of repositories it will accept webhooks from.

  Notes:
  * Accepts a comma separated list, ex. `definition1,definition2`
  * Format is `{hostname}/{owner}/{repo}`, ex. `github.com/runatlantis/atlantis`
  * `*` matches any characters, ex. `github.com/runatlantis/*` will match all repos in the runatlantis organization
  * For Bitbucket Server: `{hostname}` is the domain without scheme and port, `{owner}` is the name of the project (not the key), and `{repo}` is the repo name

  Examples:
  * Whitelist `myorg/repo1` and `myorg/repo2` on `github.com`
    * `--repo-whitelist=github.com/myorg/repo1,github.com/myorg/repo2`
  * Whitelist all repos under `myorg` on `github.com`
    * `--repo-whitelist='github.com/myorg/*'`
  * Whitelist all repos in my GitHub Enterprise installation
    * `--repo-whitelist='github.yourcompany.com/*'`
  * Whitelist all repositories
    * `--repo-whitelist='*'`
  
* ### `--require-approval`
  <Badge text="Deprecated" type="warn"/>
  ```bash
  atlantis server --require-approval
  ```
  This flag is deprecated. It requires all pull requests to be approved
  before `atlantis apply` is allowed. See [Apply Requirements](apply-requirements.html) for more details.
  
  Instead of using this flag, create a server-side `--repo-config` file:
  ```yaml
  # repos.yaml
  repos:
  - id: /.*/
    apply_requirements: [approved]
  ```
  Or use `--repo-config-json='{"repos":[{"id":"/.*/", "apply_requirements":["approved"]}]}'` instead.
  
* ### `--require-mergeable`
  <Badge text="Deprecated" type="warn"/>
  ```bash
  atlantis server --require-mergeable
  ```
  This flag is deprecated. It causes all pull requests to be mergeable
  before `atlantis apply` is allowed. See [Apply Requirements](apply-requirements.html) for more details.
  
  Instead of using this flag, create a server-side `--repo-config` file:
  ```yaml
  # repos.yaml
  repos:
  - id: /.*/
    apply_requirements: [mergeable]
  ```
  Or use `--repo-config-json='{"repos":[{"id":"/.*/", "apply_requirements":["mergeable"]}]}'` instead.
  
* ### `--silence-whitelist-errors`
  ```bash
  atlantis server --silence-whitelist-errors
  ```
  Some users use the `--repo-whitelist` flag to control which repos Atlantis
  responds to. Normally, if Atlantis receives a pull request webhook from a repo not listed
  in the whitelist, it will comment back with an error. This flag disables that commenting.

  Some users find this useful because they prefer to add the Atlantis webhook
  at an organization level rather than on each repo.

* ### `--slack-token`
  ```bash
  atlantis server --slack-token=token
  # or (recommended)
  ATLANTIS_SLACK_TOKEN='token' atlantis server
  ```
  API token for Slack notifications. Slack is not fully supported. TODO: Slack docs.

* ### `--ssl-cert-file`
  ```bash
  atlantis server --ssl-cert-file="/etc/ssl/certs/my-cert.crt"
  ```
  File containing x509 Certificate used for serving HTTPS.
  If the cert is signed by a CA, the file should be the concatenation
  of the server's certificate, any intermediates, and the CA's certificate.
  
* ### `--ssl-key-file`
  ```bash
  atlantis server --ssl-cert-file="/etc/ssl/private/my-cert.key"
  ```
  File containing x509 private key matching `--ssl-cert-file`.
  
* ### `--tfe-token`
  ```bash
  atlantis server --tfe-token="xxx.atlasv1.yyy"
  # or (recommended)
  ATLANTIS_TFE_TOKEN='xxx.atlasv1.yyy' atlantis server
  ```
  A token for Terraform Enterprise integration. See [Terraform Enterprise](terraform-enterprise.html) for more details.
