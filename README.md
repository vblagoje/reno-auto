# Reno Auto
Automatically generate comprehensive Reno release notes for your PR requests

![License](https://img.shields.io/github/license/vblagoje/auto-reno)
![Docker Pulls](https://img.shields.io/docker/pulls/vblagoje/openapi-rag-service)

## Description
Reno Auto is a GitHub Action designed to generate [reno](https://docs.openstack.org/reno/latest/) release notes using Large Language Models (LLMs) automatically. By default, it utilizes OpenAI's models, but it also supports integration with a variety of other LLM providers such as fireworks.ai, together.xyz, anyscale, octoai, etc., allowing users to select their preferred provider and LLMs to best suit their needs.

![Auto Reno  Demo](https://raw.githubusercontent.com/vblagoje/various/main/new_reno.gif)


## Usage
The minimum requirements to use this action with its default settings are:
- You have an `OPENAI_API_KEY` set in your repository secrets.
- You have given "Read and write permissions" to workflows in your repository:
  - Settings -> Actions -> General -> Workflow Permissions: Select 'Read and write permissions' and Save
- Add a workflow to your repository to trigger this action when a new PR is opened. See the example below.
- (Optional) Use the [example pull request template in your repository to create an initial PR description](https://github.com/vblagoje/pr-auto/blob/main/.github/pull_request_template.md)

## Recommended Usage Example and Safety Explanation

Here's a minimal example of how to use the Reno Auto in a pull request workflow:

```yaml
name: Pull Request Reno Note Generator Workflow

on:
  pull_request_target:
    types: [opened]

jobs:
  create-pr-release-note:
    runs-on: ubuntu-latest
    steps:
      - name: Generate Release Note for this PR
        uses: vblagoje/reno-auto@v1
        id: reno-auto-step
        with:
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}

      - name: Create PR comment
        uses: peter-evans/create-or-update-comment@v4
        with:
          issue-number: ${{github.event.pull_request.number}}
          body: |
            We use reno release [notes](https://docs.openstack.org/reno/latest/) to describe the code changes in this PR. Follow these steps:

            1. Install reno via `pip install reno` (only once per virtual environment)
            2. Run this command in your terminal from the project root:
            ```
            reno new ${{steps.reno-auto-step.outputs.file-name}}
            ```
            3. This command will generate a new release note file in the `releasenotes/notes` directory.
               Paste the following release note text into that file:
            ```
            ${{steps.reno-auto-step.outputs.note}}
            ```
            4. Review the release note text, adjust if needed, and save the file.
            5. Add this file to your commit and push it to the branch.
```
This usage is recommended because it addresses the security concerns in the following ways:

1. No code checkout: The workflow doesn't use actions/checkout, so it never checks out any code from forks. This eliminates the risk of running untrusted code.
2. Limited scope: The vblagoje/reno-auto@v1 action only generates release notes based on the pull request metadata. It doesn't interact with the actual code in the PR.
3. Secure comment creation: The peter-evans/create-or-update-comment@v4 action is a well-known and trusted action that only creates or updates a comment on the PR. It doesn't modify any repository files or execute any untrusted code.
4. No cache usage: This workflow doesn't interact with or modify any caches, avoiding the risk of cache poisoning.

By following this example, you can safely generate and provide release note suggestions for pull requests, including those from forks, without exposing your repository to the security risks associated with executing untrusted code or granting unnecessary permissions.


## Inputs

- `github_token` **Required** GITHUB_TOKEN or a repository-scoped Personal Access Token (PAT), defaulting to the GitHub token provided by the GitHub Actions runner. It is essential for invoking the GitHub API REST service to retrieve Pull Request details. Using GITHUB_TOKEN permits actions to access both public and private repositories, helping to bypass rate limits imposed by the GitHub API.

- `openai_api_key`
**Required** The OpenAI API key for authentication. Note that this key could be from other LLM providers as well.

- `openai_base_url` **Optional** The base URL for the OpenAI API. Using this input one can use different LLM providers (e.g. fireworks.ai, together.xyz, anyscale, octoai etc.) Defaults to https://api.openai.com/v1

- `github_repository`**Optional** The GitHub repository where the pull request is made. Defaults to the current repository.

- `base_branch` **Optional** The base (target) branch in the pull request. Defaults to the base branch of the current PR.

- `head_branch` **Optional** The head (source) branch in the pull request. Defaults to the head branch of the current PR.

- `generation_model` **Optional** LLM to use for reno release note text generation. Defaults to gpt-4o from OpenAI.

- `function_calling_model` **Optional** LLM to use for function calling (service parameter resolution, output formatting etc). Defaults to gpt-3.5-turbo from OpenAI.

- `system_prompt` **Optional** System message/prompt to help the model generate reno release note (prompt text or URL where prompt text can be found). Defaults to https://bit.ly/reno_release_note_system_prompt_v2

## Important Security Consideration

Using reno-auto in conjunction with fetching code from untrusted PR forks (via actions/checkout) poses a significant security risk, especially when used with the `pull_request_target` event. 
For example, another Github Action [create-or-update-release-note](https://github.com/vblagoje/create-or-update-release-note) can create a release note commit directly in the PR branch. However, it needs to check out the code from the fork, which can have significant security risks. Use this approach for trusted PRs only (e.g., PRs from your own repository).
For more detailed information on these security considerations, refer to:
- [GitHub Actions documentation on pull_request_target](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target)
- [GitHub Security Lab article: "Keeping your GitHub Actions and workflows secure: Preventing pwn requests"](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/)


## Contributing

If you have ideas for enhancing Reno Auto, or if you encounter a bug, we encourage you to contribute by opening an issue or a pull request. 
The core of this GitHub Action is built on top of Docker image of the [vblagoje/openapi-rag-service](https://github.com/vblagoje/openapi-rag-service/) project. 
Therefore, for contributions beyond minor edits to the `action.yml` or `README.md`, please direct your pull requests to 
the [vblagoje/openapi-rag-service](https://github.com/vblagoje/openapi-rag-service/) GitHub repository.


## License
This project is licensed under [Apache 2.0 License](LICENSE).
