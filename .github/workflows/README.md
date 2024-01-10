# Reusable workflows

## How to Use Our Reusable Workflows?

In your repository's workflow file, reference the desired workflow from the `.github` repository, i.e:

```yaml
name: Tag Pull Request

on:
  pull_request_target:

jobs:
  tag-pull-request:
    uses: commercelayer/.github/.github/workflows/tag_pull_request.yaml@main
    secrets: inherit
```

Ensure you've provided necessary permissions and secrets if the workflow requires them.

Enjoy the streamlined and standardized CI/CD experience!

For detailed instructions, please refer to [GitHub's documentation on reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows).
