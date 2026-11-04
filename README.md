# Monsieur Cuisine cookbook template

Use this repository as a starter for a private, version-controlled Monsieur Cuisine recipe collection. Recipes are stored as Markdown; the GitHub Action validates them and manages the corresponding account recipes.

## Start your cookbook

1. Select **Use this template** on GitHub and create a repository in your account.
2. In the new repository, add the Actions secrets described below.
3. Add recipes under `recipes/` and open a pull request.

The [CLI Action guide](https://github.com/fopina/monsieur-cuisine-cli/blob/main/action/README.md) is the canonical reference for the recipe format, accepted fields, and validation rules. Use it rather than copying the full schema here.

## Configure Actions secrets

### Monsieur Cuisine API token

Create a dedicated login session for this cookbook, then pipe its token directly into GitHub with the CLI and GitHub CLI:

```sh
uvx --from 'git+https://github.com/fopina/monsieur-cuisine-cli.git@main' monsieur-cuisine-cli --session-file github-cookbook-session.json auth login
uvx --from 'git+https://github.com/fopina/monsieur-cuisine-cli.git@main' monsieur-cuisine-cli --session-file github-cookbook-session.json auth get-token | gh secret set MC_API_ACCESS_TOKEN --repo OWNER/REPOSITORY
```

The `--from ...@main` source makes the new token command available before its next PyPI release; after release, the shorter `uvx monsieur-cuisine-cli ...` form will work. Replace `OWNER/REPOSITORY` with the new repository. `gh` must be authenticated and have access to it. Keep this session file for the cookbook workflow; refreshing the API token rotates it and invalidates the old value. The action writes a rotated token back to `MC_API_ACCESS_TOKEN` after a token-authenticated operation.

### GitHub secret-writing token

Add `GH_SECRETS_WRITE_TOKEN` as a second repository secret. It must be a fine-grained personal access token for the cookbook repository with **Secrets: Read and write** permission. See [GitHub's fine-grained token documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).

The **Get recipe** workflow opens a pull request when run on `main`. In the repository, enable **Settings → Actions → General → Workflow permissions → Allow GitHub Actions to create and approve pull requests**. The draft workflow only updates its existing recipe PR branch; it needs `contents: write` for that commit.

For the exact action inputs and permissions, see the CLI Action guide's [cookbook workflow instructions](https://github.com/fopina/monsieur-cuisine-cli/blob/main/action/README.md#use-in-a-cookbook-repository). For CLI token import/export and other login details, see [CLI sign-in and token transfer](https://github.com/fopina/monsieur-cuisine-cli/blob/main/README-CLI.md#transfer-an-api-token).

## Add recipes

Create a `.md` file under `recipes/`. The action validates recipe fields, localized dropdown values, and images on pushes and pull requests. The [recipe format and validation reference](https://github.com/fopina/monsieur-cuisine-cli/blob/main/action/README.md#recipe-format) explains the required table fields and supported annotations.

The demos show two distinct locale-specific recipes: [Tomato soup (English, `en-US`)](recipes/tomato-soup.md) and [Sopa cremosa de tomate (Português, `pt-PT`)](recipes/sopa-de-tomate.md). There is no “master” locale containing every recipe in every language: create a separate recipe file for each language, with its own title, ingredients, steps, locale-specific dropdown values, and account recipe ID.

When a new recipe PR is opened from a branch in this repository, the workflow creates a draft and records its ID in the PR. Merging publishes that recipe as private. Recipes created by this workflow are not public; make one public separately through Monsieur Cuisine if desired. See the CLI Action guide for the [draft and publish flow](https://github.com/fopina/monsieur-cuisine-cli/blob/main/action/README.md#recipe-pull-request-workflow).

## Fetch a recipe

Run **Get recipe** from the Actions tab and enter a Monsieur Cuisine URL or numeric recipe ID. The workflow creates Markdown, converts it back to JSON, compares the conversion with the fetched source, and attaches both JSON artifacts. Its PR links to the run and reports the conversion/comparison status; a conversion failure is recorded for review rather than blocking creation of the Markdown PR.
