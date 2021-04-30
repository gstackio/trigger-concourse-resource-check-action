# Trigger Concourse resource-check action

This action will trigger a resource check in a specific
[Concourse](https://concourse-ci.org) pipeline.

This is best-practice when integrating Github with Concourse, in order for
Concourse to immediately detect changes that have just been pushed to branches
and PRs. With this, you Concourse pipeline will have the best reactivity
possible. PRs will trigger CI tests immediately, for the best contributor
experience in your community.


## Usage

### Trigger a check on a `git` resource

#### Github workflow configuration

Here is below an example workflow configuration that you can add in the
`.github/workflows` folder of your Git repo.

```yaml
name: Trigger Concourse push
on: push
jobs:
  trigger-resource-check:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger resource check
        uses: gstackio/trigger-concourse-resource-check-action@v1
        with:
          concourse-url:           https://concourse.example.com
          concourse-team:          developers
          concourse-pipeline:      deploy-app
          concourse-resource:      github-repo
          concourse-webhook-token: ${{ secrets.CONCOURSE_WEBHOOK_TOKEN }}
```

#### Concourse pipeline secret generatioin

A `concourse-webhook-token` secret should be created in your Concourses secret
vault, which can be either Credhub or Hashicorp Vault. Ask your Concourse ops
team about which vault implementation is actually used by your Concourse
installation.

Typically you generate it directly in Credhub with a `credhub` CLI invocation
like the following.

```
credhub generate --name "/concourse/main/deploy-app/concourse-webhook-token" --type "password"
```

#### Github serets configuration

A `CONCOURSE_WEBHOOK_TOKEN` Github secret should be created, either at the
organization or at the repository level, with the same value as the
`concourse-webhook-token` Credhub secret above.

For example, to fetch the above secret in Credhub, a command like the
following can be used.

```
credhub get --name "/concourse/main/deploy-app/concourse-webhook-token"
```

#### Concourse pipeline configuration

In your Concourse pipeline, add a `webhook_token` property to your resource of
type `git`, named `github-source-code` here in the example.

The `((concourse-webhook-token))` placeholder refers to the secret generated
in Credhub above.

You can then lower the rate at which the resource will automatically check for
any new commits in Github. We suggest a rate of once every 24h here for the
[`check_every`][check_every_resource_property] resource property.

```yaml
resources:
  - name: github-source-code
    type: git
    icon: github
    source:
      uri: ...
      branch: ...
      private_key: ...
    webhook_token: ((concourse-webhook-token))
    check_every: 24h
```

Resource of type [`git`][git_resource] are standard in Concourse. You don't
need to define it as a custom resource type.

[check_every_resource_property]: https://concourse-ci.org/resources.html#schema.resource.check_every
[git_resource]: https://github.com/concourse/git-resource

### Trigger a check on a `github-pr` resource

Resource of type [`github-pr`][github_pr_rsc] come as a plugin in Concourse,
referred to as a “custom resource type”. You need to declare it in the
[`resource_types`][rsc_types_config] section of your pipeline declaratioin.
Refer to the `github-pr` documentation for
[detailed examples][github_pr_rsc_examples].

The `concourse-webhook-token` secret generation in Credhub and
`CONCOURSE_WEBHOOK_TOKEN` secret setup in Github are the same as above with a
`git` resource. The `webhook_token` resource property in your pipeline
definition is also to be set in the same way as above.

[github_pr_rsc]: https://github.com/telia-oss/github-pr-resource
[rsc_types_config]: https://concourse-ci.org/resource-types.html
[github_pr_rsc_examples]: https://github.com/telia-oss/github-pr-resource#example

#### Github workflow configuration

Here is below an example workflow configuration that you can add in the
`.github/workflows` folder of your Git repo. Both `push` and `pull_request`
event types should be listened at in this case.

```yaml
name: Trigger Concourse pull request
on: [ push, pull_request ]
jobs:
  trigger-resource-check:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger resource check
        uses: gstackio/trigger-concourse-resource-check-action@v1
        with:
          concourse-url:           https://concourse.example.com
          concourse-team:          developers
          concourse-pipeline:      test-app
          concourse-resource:      github-pull-requests
          concourse-webhook-token: ${{ secrets.CONCOURSE_WEBHOOK_TOKEN }}
```


## Reference

### Inputs

All inputs are required.

* **concourse-url:** The base URL for your Concourse CI
* **concourse-team:** The Concourse team where the pipeline lives. If not set, defaults to `main`
* **concourse-pipeline:** The Concourse pipeline where the resource lives
* **concourse-resource:** The resource for which a check is to be triggered
* **concourse-webhook-token:** The secret value used for the `webhook_token` property of the Concourse resource

### Required secrets

A secret named `CONCOURSE_WEBHOOKK_TOKEN` is required. This can be defined at
organization level or repository level.

### Outputs

None

### Example usage

```yaml
uses: gstackio/trigger-concourse-resource-check-action@v1
with:
  concourse-url:           https://concourse.example.com
  concourse-team:          test
  concourse-pipeline:      deploy-app
  concourse-resource:      build-and-deploy
  concourse-webhook-token: secret-token-value
```
