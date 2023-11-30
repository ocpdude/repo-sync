# Repo Sync

> Keep a pair of GitHub repos in sync.

## How it Works

This project uses [GitHub Actions](https://github.com/features/actions) workflows to keep pairs of git repos in sync. It runs on a [schedule](#cron) (every 15 minutes by default). Shortly after changes are made to the default branch of **repo A**, the Actions workflow runs on **repo B** and generates a pull request including the recent changes from **repo A**. If more changes are made to **repo A** before the pull request is merged, those changes will be added to the existing pull request. The same is true in the opposite direction: changes made to **repo B** will eventually get picked up by the workflow in **repo A**. 

## Features

- One-way or two-way sync
- Sync between a private and public repo
- Sync between two private repos
- Sync between two public repos
- Sync from a third-party repo to a Github repo
- Uses Github Actions and a flexible scheduled job. No external service required!

## Requirements

- Your two repos must share a commit history.
- Your repos must be using GitHub Actions v2.0 or higher. If you're not sure, check for a `.github/workflows` directory in your repo. If it doesn't exist, you'll need to [enable GitHub Actions](https://help.github.com/en/articles/configuring-a-workflow#enabling-github-actions).

## Manual Installation

### Step 1. Set up Secrets

[GitHub Secrets] are variables stored on your GitHub repository that are made available in the GitHub Actions environment. There are two (2) required secrets on each repo. Go to Settings > Secrets on your repo page and add the following secrets:

#### `SOURCE_REPO`

The shorthand name or URL of the repo to sync.

- If the source repo is a **public** GitHub repo, use a shorthand name like `owner/repo`.
- If the source repo is a **private** GitHub repo, specify an HTTPS clone URL in the format `https://<access_token>@github.com/owner/repo.git` that includes an access token with `repo` and `workflow` scopes. [Generate a token](https://github.com/settings/tokens/new?description=repo-sync&scopes=repo,workflow).
- If the source repo is not hosted on GitHub, specify an HTTPS URL that includes pull access credentials.


#### `INTERMEDIATE_BRANCH`

The name of the temporary branch to use when creating a pull request, e.g. `repo-sync`. You can use whatever name you like, but it should NOT be the name of a branch that already exists, as it will be overwritten.

### Step 2. Create Actions workflow files

Create a file `.github/workflows/repo-sync.yml` in **both repositories** and add the following content:

```yaml
name: Repo Sync

on:
  schedule: 
  - cron: "*/15 * * * *" # every 15 minutes. set to whatever interval you like

jobs:
  repo-sync:
    name: Repo Sync
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: ocpdude/repo-sync@main
      name: Sync repo to branch
      with:
        source_repo: ${{ secrets.SOURCE_REPO }}
        source_branch: main
        destination_branch: ${{ secrets.INTERMEDIATE_BRANCH }}
        github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Step 3. Watch the pull requests roll in!

There is no step 3! Once commited to your repo, your workflows will start running on the schedule you've specified in the workflow file. Whenever changes are found, a pull request will be created (or updated if a sync pull request already exists).

## Advanced configuration

The workflow file is fully customizable allowing for advanced configurations.

#### cron

The default cron is every 15 minutes. This can be easily adjusted by changing the cron string.

#### Manual events

Instead of triggering workflows using the cron scheduler, you can setup [manual events](https://help.github.com/en/articles/events-that-trigger-workflows#manual-events) to trigger the workflow when the source repo changes.

#### Workflow steps

You can add/remove workflow steps to meet your needs. For example, the "Create pull request" step can be removed, or perhaps a "Merge pull request" step can be added.

#### Customize pull request

You can customize PR title, body, label, reviewer, assingee, milestone by setting environment variables as explained at [ocpdude/pull-request](https://github.com/ocpdude/pull-request#advanced-options).

#### Use SSH clone url and deploy keys

You can use SSH clone url and specify `SSH_PRIVATE_KEY` environment variable instead of using the https clone url. This is useful when you want to sync a private repo to a public repo.

[GitHub Secrets]: https://help.github.com/en/actions/configuring-and-managing-workflows/using-variables-and-secrets-in-a-workflow
[Actions workflow file]: https://help.github.com/en/articles/configuring-a-workflow
