# Parse CHANGELOG.md and get the body of a specific version

## 🚦 Usage

When there is a commit following a specific format `chore: release vX.X.X`, this action will parse the `CHANGELOG.md` file and create a release with the body of the specified version.

It's good with [kiyoon/changelog-action](https://github.com/kiyoon/changelog-action) that generates the `CHANGELOG.md` file.

```yml
on:
  push:
    branches:
      - main
      - master
    paths:
      - 'CHANGELOG.md'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Parse commit message to get version
        id: parse_commit
        env:
          # Avoid script injection by using an environment variable
          # The `${{ ... }}` syntax is like a preprocessor for the workflow, thus it is unsafe to use directly in the script
          COMMIT_MESSAGE: ${{ github.event.head_commit.message }}
        run: |
          # Commit message must be in the format of
          # "chore: release vX.X"
          VERSION=$(echo "$COMMIT_MESSAGE" | grep -oP '^chore:\ release\ v[a-zA-Z0-9.\-\+]+' | head -n 1 | cut -d' ' -f3)
          if [[ -z "$VERSION" ]]; then
            echo "Commit message does not match the expected format. Please use 'chore: release vX.X.X' format in your commit message."
            echo "Commit message: $COMMIT_MESSAGE"
            exit 1
          fi
          echo "RELEASE_VERSION=$VERSION" >> "$GITHUB_OUTPUT"

      - name: Parse CHANGELOG.md to get release notes
        id: parse_changelog
        uses: kiyoon/parse-changelog-action@v1
        with:
          changelog-file-path: CHANGELOG.md
          version: ${{ steps.parse_commit.outputs.RELEASE_VERSION }}

      - name: Create Release
        id: create_release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.parse_commit.outputs.RELEASE_VERSION }}
          name: ${{ steps.parse_commit.outputs.RELEASE_VERSION }}
          body: ${{ steps.parse_changelog.outputs.body }}
```

## 🚧 Maintenance Note 🚧

```bash
brew install bun  # install package manager
bun install       # install dependencies

# IMPORTANT: make sure to build the project when you change the code
bun run build
```
