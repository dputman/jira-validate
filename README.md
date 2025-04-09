# jira-validate 🧹

> A light-weight lint workflow when using GitHub along with [JIRA][jira] for project management.
> Ported from [pivotal-lint](https://github.com/ClearTax/pivotal-lint/) for similar usage with Atlassian's Jira Software.

![GitHub package.json version](https://img.shields.io/github/package-json/v/dputman/jira-validate?style=flat-square)
[![GitHub](https://img.shields.io/github/license/dputman/jira-validate?style=flat-square)](https://github.com/dputman/jira-validate/blob/main/LICENSE.md)
[![All Contributors](https://img.shields.io/badge/all_contributors-2-orange.svg?style=flat-square)](#contributors)
![build & test](https://github.com/dputman/jira-validate/workflows/lint,%20build%20&%20test/badge.svg)

---

<!-- toc -->

- [Installation](#installation)
  - [Semantic Versions](#semantic-versions)
- [Features](#features)
  - [PR Status Checks](#pr-status-checks)
  - [PR Description & Labels](#pr-description--labels)
    - [Description](#description)
    - [Labels](#labels)
    - [Soft-validations via comments](#soft-validations-via-comments)
  - [Options](#options)
  - [`jira-token`](#jira-token)
  - [Skipping branches](#skipping-branches)
- [Contributing](#contributing)
- [FAQ](#faq)
- [Contributors](#contributors)

<!-- tocstop -->

## Installation

To make `jira-validate` a part of your workflow, just add a `jira-validate.yml` file in your `.github/workflows/` directory in your GitHub repository.

```yml
name: jira-validate
on: [pull_request]

jobs:
  jira-validate:
    runs-on: ubuntu-latest
    steps:
      - uses: dputman/jira-validate@main
        name: jira-validate
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          jira-token: ${{ secrets.JIRA_TOKEN }}
          jira-base-url: https://your-domain.atlassian.net
          skip-branches: '^(production-release|main|release\/v\d+)$'
          skip-comments: true
          pr-threshold: 1000
```

It can also be used as part of an existing workflow by adding it as a step. More information about the [options here](#options).

### Semantic Versions

If you want more stability in versions of `jira-validate` than `@main` you can also use the [semantic releases for jira-validate](https://github.com/dputman/jira-validate/releases).

Example:

```yaml
# ...
steps:
  - uses: dputman/jira-validate@v0.0.1
    name: jira-validate
    # ...
```

## Features

### PR Status Checks

`jira-validate` adds a status check which helps you avoid merging PRs which are missing a valid Jira Issue Key in the branch name. It will use the [Jira API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/) to validate a given key.

### PR Description & Labels

#### Description

When a PR passes the above check, `jira-validate` will also add the issue details to the top of the PR description. It will pick details such as the Issue summary, type, estimation points, status and labels and add them to the PR description.

#### Labels

`jira-validate` will automatically label PRs with:

- A label based on the Jira Project name (the project the issue belongs to). For example, if your project name is `Escher` then it will add `escher` as a label.
- `HOTFIX-PROD` - if the PR is raised against `production-release`.
- `HOTFIX-PRE-PROD` - if the PR is raised against `release/v*`.
- Jira issue type ([based on your project](https://confluence.atlassian.com/adminjiracloud/issue-types-844500742.html)).

<figure>
 <img src="https://assets1.cleartax-cdn.com/cleargst-frontend/misc/1580891341_jira_lint.png" alt="Issue details and labels added to a PR" />
 <figcaption>
 Issue details and labels added to a PR.
 </figcaption>
</figure>

#### Issue Status Validation
Issue status is shown in the [Description](#description).
**Why validate issue status?** 
In some cases, one may be pushing changes for a story that is set to `Done`/`Completed` or it may not have been pulled into working backlog or current sprint.

 This option allows discouraging pushing to branches for stories that are set to statuses other than the ones allowed in the project; for example - you may want to only allow PRs for stories that are in `To Do`/`Planning`/`In Progress` states.

The following flags can be used to validate issue status:
- `validate_issue_status`
  - If set to `true`, `jira-validate` will validate the issue status based on `allowed_issue_statuses`
- `allowed_issue_statuses`
  - This will only be used when `validate_issue_status` is `true`. This should be a comma separated list of statuses. If the detected issue's status is not in one of the `allowed_issue_statuses` then `jira-validate` will fail the status check.

**Example of invalid status**
  <p>:broken_heart: The detected issue is not in one of the allowed statuses :broken_heart: </p>    
      <table>
        <tr>
            <th>Detected Status</th>
            <td>${issueStatus}</td>
            <td>:x:</td>
        </tr>
        <tr>
            <th>Allowed Statuses</th>
            <td>${allowedStatuses}</td>
            <td>:heavy_check_mark:</td>
          </tr>
      </table>
  <p>Please ensure your jira story is in one of the allowed statuses</p>
    
#### Soft-validations via comments

`jira-validate` will add comments to a PR to encourage better PR practices:

**A good PR title**

<figure>
  <img src="https://user-images.githubusercontent.com/6426069/69525276-c6e62b80-0f8d-11ea-9db4-23d524b5276c.png" />
  <figcaption>When the title of the PR matches the summary/title of the issue well.</figcaption>
</figure>

---

<figure>
  <img src="https://user-images.githubusercontent.com/6426069/69480647-6a6cfa00-0e2f-11ea-8750-4294f686dac7.png" />
  <figcaption>When the title of the PR is <strong>slightly different</strong> compared to the summary/title of the issue</figcaption>
</figure>

---

<figure>
  <img src="https://user-images.githubusercontent.com/6426069/69526103-7243b000-0f8f-11ea-9deb-acb8cbb6610b.png" />
  <figcaption>When the title of the PR is <strong>very different</strong>  compared to the summary/title of the issue</figcaption>
</figure>

---

**A comment discouraging PRs which are too large (based on number of lines of code changed).**

<figure>
  <img src="https://user-images.githubusercontent.com/6426069/69480043-e06e6280-0e29-11ea-8e24-173355c304dd.png" />
  <figcaption>Batman says no large PRs 🦇</figcaption>
</figure>

### Options

| key             | description                                                                                                                                                                                                                                                                                                        | required | default |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ------- |
| `github-token`  | Token used to update PR description. `GITHUB_TOKEN` is already available [when you use GitHub actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token#about-the-github_token-secret), so all that is required is to pass it as a param here. | true     | null    |
| `jira-token`    | Token used to fetch Jira Issue information.  Check [below](#jira-token) for more details on how to generate the token.                                                                                                          | true     | null    |
| `jira-base-url` | The subdomain of JIRA cloud that you use to access it. Ex: "https://your-domain.atlassian.net".                                                                                                                                                                                                                    | true     | null    |
| `skip-branches` | A regex to ignore running `jira-validate` on certain branches, like production etc.                                                                                                                                                                                                                                    | false    | ' '     |
| `skip-comments` | A `Boolean` if set to `true` then `jira-validate` will skip adding lint comments for PR title.                                                                                                                                                                                                                         | false    | false   |
| `pr-threshold`  | An `Integer` based on which `jira-validate` will add a comment discouraging huge PRs.                                                                                                                                                                                                                                  | false    | 800     |
| `validate_issue_status`  | A `Boolean` based on which `jira-validate` will validate the status of the detected jira issue                                                                                                                                                                                                              | false    | false   |
| `allowed_issue_statuses`  | A comma separated list of allowed statuses. The detected jira issue's status will be compared against this list and if a match is not found then the status check will fail. *Note*: Requires `validate_issue_status` to be set to `true`.                                                                                        | false    | `"In Progress"` |


### `jira-token`

Since tokens are private, we suggest adding them as [GitHub secrets](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets).

The Jira token is used to fetch issue information via the Jira REST API. To get the token:-
1. Generate an [API token via JIRA](https://confluence.atlassian.com/cloud/api-tokens-938839638.html)
2. Create the encoded token in the format of `base64Encode(<username>:<api_token>)`.
   For example, if the username is `ci@example.com` and the token is `954c38744be9407ab6fb`, then `ci@example.com:954c38744be9407ab6fb` needs to be base64 encoded to form `Y2lAZXhhbXBsZS5jb206OTU0YzM4NzQ0YmU5NDA3YWI2ZmI=`
3. The above value (in this example `Y2lAZXhhbXBsZS5jb206OTU0YzM4NzQ0YmU5NDA3YWI2ZmI=`) needs to be added as the `JIRA_TOKEN` secret in your GitHub project.

Note: The user should have the [required permissions (mentioned under GET Issue)](https://developer.atlassian.com/cloud/jira/platform/rest/v3/?utm_source=%2Fcloud%2Fjira%2Fplatform%2Frest%2F&utm_medium=302#api-rest-api-3-issue-issueIdOrKey-get).

### Skipping branches

Since GitHub actions take string inputs, `skip-branches` must be a regex which will work for all sets of branches you want to ignore. This is useful for merging protected/default branches into other branches. Check out some [examples in the tests](https://github.com/dputman/jira-validate/blob/08a47ab7a6e2bc235c9e34da1d14eacf9d810bd1/__tests__/utils.test.ts#L33-L44).

`jira-validate` already skips PRs which are filed by bots (for eg. [dependabot](https://github.com/marketplace/dependabot-preview)). You can add more bots to [this list](https://github.com/dputman/jira-validate/blob/08a47ab7a6e2bc235c9e34da1d14eacf9d810bd1/src/constants.ts#L4), or add the branch-format followed by the bot PRs to the `skip-branches` option.

## Contributing

Follow the instructions [here](https://help.github.com/en/articles/creating-a-javascript-action#commit-and-push-your-action-to-github) to know more about GitHub actions.

## FAQ

<details>
  <summary>Why is a Jira key required in the branch names?</summary>

The key is required in order to:

- Automate change-logs and release notes ⚙️.
- Automate alerts to QA/Product teams and other external stake-holders 🔊.
- Help us retrospect the sprint progress 📈.

</details>

<details>
  <summary>Is there a way to get around this?</summary>
  Nope 🙅

</details>

[jira]: https://www.atlassian.com/software/jira

