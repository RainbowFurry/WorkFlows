name: Publish Latest Release

on:
  push:
    branches:
      - main  # Replace 'main' with the branch you want to monitor

jobs:
  create-release:
    runs-on: ubuntu-latest

  steps:
    - name: Checkout code
      uses: actions/checkout@v3

   - name: Set up Git
      run: |
        git config user.name "github-actions[bot]"
        git config user.email "41898282+github-actions[bot]@users.noreply.github.com"

  - name: Get version from package.json (for Node.js projects)
      id: get_version
      run: echo "version=$(jq -r .version package.json)" >> $GITHUB_ENV
      # Alternatively, use your versioning system or tag if applicable

    - name: Create Release Notes
      id: release_notes
      run: |
        echo "Generating release notes..."
        echo "Changes in version ${{ env.version }}" > release_notes.md
        git log --oneline --decorate $(git describe --tags --abbrev=0)..HEAD >> release_notes.md
      shell: bash

    - name: Publish GitHub Release
      uses: actions/create-release@v1
      with:
        tag_name: v${{ env.version }}  # Or use another versioning/tagging scheme
        release_name: v${{ env.version }}
        body_path: release_notes.md
        draft: false
        prerelease: false
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
