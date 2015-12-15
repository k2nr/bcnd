# Barcelona Deployer

Degica's opinionated deploy pipeline tool

## Installation

```
gem 'bcnd'
```

## Prerequisites

bcnd is very opinionated about your development configurations.
So there are lots of requirements you need to prepare beforehand.

### Quay.io

You need quay.io as your docker image registry

### Barcelona

where your applications will be deployed

### GitHub

bcnd only supports GitHub as a source code repository.
Your GitHub repository must have 2 branches: mainline and stable
bcnd doesn't offer a customizability for the branch names.

#### mainline branch

a mainline branch includes all reviewed commits. By default mainline branch is `master`.
bcnd automatically deploys mainline branch to your mainline environment.

#### stable branch

a stable branch includes stable commits. By default stable branch is `production`.
bcnd automatically deploys stable branch to your stable environment.

### Webhooks

- webhook for quay.io automated builds
- webhook for your CI service

## Usage

### `bcnd:deploy` rake task

Execute `bcnd:deploy` in your CI's "after success" script

Here's the example for travis CI

```yml
after_success:
  - bundle exec rake bcnd:deploy
```

### Environment variables

Set the following environment variables to your CI build environment:

- `QUAY_TOKEN`
  - a quay.io oauth token. you can create a new oauth token at quay.io organization page -> OAuth Applications -> Create New Application -> Generate Token
- `GITHUB_TOKEN`
  - a github oauth token which has a permission to read your application's repository
- `HERITAGE_TOKEN`
  - Barcelona's heritage token
- `QUAY_REPOSITORY`
  - A name of your quay repository. It should be `[organization]/[repo name]`. If you don't set this variable bcnd uses github repository name as a quay repository name.

## How It Works

### Deploying to a staging environment

When a commit is pushed into a mainline branch, bcnd deploys your application to barcelona's mainline environment.
Here's what bcnd actually does:

- Wait for quay.io automated build to be finished
- Attach a docker image tag to the `latest` docker image.
  - The tag is the latest mainline branch's commit hash
- Call Barcelona deploy API with the tag name
  - `bcn deploy -e [mainline env] --tag [git_commit_hash]`

### Deploying to a production environment

When a commit is pushed into a stable branch, bcnd deploys your application to barcelona's stable environment
Here's what bcnd actually does:

- Compare `master` and `production` branch
  - If there is a difference between the branches, bcnd raises an error
- Get master branch's latest commit hash
- Find a docker image from quay.io with the tag of the master commit hash
- Call Barcelona deploy API with the tag name
  - `bcn deploy -e [stable env] --tag [git_commit_hash]`
 