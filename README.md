# Renovate helm helper

This is a very opinionated helper for renovate to enhance PRs generated by renovate for helm charts under a very specific set of conditions:

* You are using Github
* Your `Chart.yaml` just refers to a single upstream chart:
```YAML
apiVersion: v2
version: 1.0.0
name: hello-world
dependencies:
- name: hello-world
  version: 0.1.0
  repository: https://helm.github.io/examples
```
* Your `values.yaml` lives in the same directory and is a copy of the upstream `values.yaml` with edits.
* You have a copy of upstream `values.yaml` stored in the same directory as `orig-values.yaml` with an indent of 2 spaces applied (so that it nearly matches the unedited `values.yaml`

## Howto

Run this python/container using your CI after any renovate.
Set the following environment variables:

| Name         | Content                                                               |
|--------------|-----------------------------------------------------------------------|
| GITHUB_TOKEN | Token which can read and write to the repostory and PRs               |
| GH_OWNER     | The github organisation                                               |
| GIT_EMAIL    | Email address to make commits as                                      |
| GIT_NAME     | Name to make commits as                                               |
| app_repo     | Repository name in github                                             |
| gitSha       | The SHA1 to check                                                     |
| targetBranch | The branch that the PR is being merged into (todo: read this from GH) |
| checkoutPath | Local path for where to checkout the code to from github to examine   |
| prNum        | Pull request number in github                                         |

`https://github.com/<GH_OWNER>/<app_repo>` is the path to your repository.

## How does it help

The `Chart.yaml`s affected by the PR will be compared to the target branch.

If the `values.yaml` upstream has changed, a patch between old upstream and new will be generated and applied to your `values.yaml` attempting to keep it up to date with upstream. If this fails, which is quite likely if it's patching near where you have modified `values.yaml`, it will add the failed diffs as a comment to the PR to help you to manually patch.

The PR will have `values-orig.yaml` updated from upstream, and a commit will be made for this. If this happens your `values.yaml` and upstream will be compared and a unified diff added as a comment to the PR.

It is safe to run this multiple times over the same PR, but `values.yaml` will not be updated more than once in a single PR.