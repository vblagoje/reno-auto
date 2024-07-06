# Reno Auto
Automatically generate comprehensive Reno release notes for your PR requests

![License](https://img.shields.io/github/license/vblagoje/auto-reno)
![Docker Pulls](https://img.shields.io/docker/pulls/vblagoje/openapi-rag-service)

## Description
Reno Auto is a GitHub Action designed to generate [reno](https://docs.openstack.org/reno/latest/) release notes using Large Language Models (LLMs) automatically. By default, it utilizes OpenAI's models, but it also supports integration with a variety of other LLM providers such as fireworks.ai, together.xyz, anyscale, octoai, etc., allowing users to select their preferred provider and LLMs to best suit their needs. This action can be customized with system and user-provided prompts to tailor the release note generation.

Reno Auto, in conjunction with the accompanying GitHub Actions, [create-or-update-release-note](https://github.com/vblagoje/create-or-update-release-note) and [find-pr-release-note](https://github.com/vblagoje/find-pr-release-note), enables the creation of a fully automated workflow for generating and updating release notes.

![Auto Reno  Demo](https://raw.githubusercontent.com/vblagoje/various/main/auto-reno-medium.gif)


## Usage
The minimum requirements to use this action with its default settings are:
- You have an `OPENAI_API_KEY` set in your repository secrets.
- You have given "Read and write permissions" to workflows in your repository:
  - Settings -> Actions -> General -> Workflow Permissions: Select 'Read and write permissions' and Save
- Add a workflow to your repository to trigger this action when a new PR is opened. See the example below.
- (Optional) Use the [example pull request template in your repository to create an initial PR description](https://github.com/vblagoje/pr-auto/blob/main/.github/pull_request_template.md)

## Minimal Example Workflow

Here's a minimal example of how to use the Reno Auto in a pull request workflow:

```yaml
name: Pull Request Reno note Generator Workflow

on:
  pull_request:
    types: [opened]
  
jobs:
  create-pr-release-note:
    runs-on: ubuntu-latest
    steps:

      - name: Checkout Repository
        uses: actions/checkout@v2
        with:
          ref: ${{ github.head_ref }}

      - name: Generate Release Note for this PR
        uses: vblagoje/reno-auto@v1
        id: reno-auto-id
        with:
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}

      - name: Create PR release note
        uses: vblagoje/create-or-update-release-note@v1
        with:
          note-name: ${{steps.reno-auto-id.outputs.file-name}}
          note-content: ${{steps.reno-auto-id.outputs.note}}
```


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

- `user_prompt` **Optional** Additional user prompt to help the model generate reno release note

- `bot_name` **Optional** The name of the bot so users can guide reno release note generation with @bot_name from PR comments. Defaults to reno-auto-bot


## Contributing

If you have ideas for enhancing Reno Auto, or if you encounter a bug, we encourage you to contribute by opening an issue or a pull request. 
The core of this GitHub Action is built on top of Docker image of the [vblagoje/openapi-rag-service](https://github.com/vblagoje/openapi-rag-service/) project. 
Therefore, for contributions beyond minor edits to the `action.yml` or `README.md`, please direct your pull requests to 
the [vblagoje/openapi-rag-service](https://github.com/vblagoje/openapi-rag-service/) GitHub repository.

## Smoke Test for Docker Image

To confirm the correct operation of the Docker image, perform a smoke test locally using the following steps:

1. **Prepare Your OpenAI API Key**: Ensure your OpenAI API key is ready for use.

2. **Execute the Image**:
   Run the following command in your terminal, replacing `<YOUR_OPENAI_API_KEY>` with your actual API key and `<YOUR_GITHUB_TOKEN>` with your actual GitHub token:

   ```bash
   docker run -e <YOUR_OPENAI_API_KEY> -e OPENAPI_SERVICE_TOKEN=<YOUR_GITHUB_TOKEN> -e SYSTEM_PROMPT=https://bit.ly/reno_release_note_system_prompt_v3 -e OPENAPI_SERVICE_SPEC=https://bit.ly/github_compare -e FUNCTION_CALLING_PROMPT="Compare branches main (BASE) and test/benchmarks2.0 (HEAD), in Github repository deepset-ai/haystack (owner/repo)" -e FUNCTION_CALLING_VALIDATION_SCHEMA=https://bit.ly/compare_branches_function_params_schema -e OUTPUT_SCHEMA=https://bit.ly/reno_release_note_fc_schema_v2 -e NOTE_TEMPLATE=https://bit.ly/reno_release_note_template -e SERVICE_RESPONSE_SUBTREE=files vblagoje/openapi-rag-service
   ```

   Modify the parameters `deepset-ai/haystack main test/benchmarks2.0` according to the specific repository main and pr branches, relevant to your use case.

3. **Check the Output**: After execution, verify the output to ensure release note looks as expected.

This test will help you verify the basic functionality of the Docker image. Remember to adjust the command with the appropriate 
project, repository, and branches you wish to compare etc.


## License
This project is licensed under [Apache 2.0 License](LICENSE).
