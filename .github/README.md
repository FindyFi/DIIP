# GitHub Actions Workflows

## Deployment Strategy

This repository uses GitHub Actions to automatically deploy the DIIP documentation to GitHub Pages with different strategies based on the branch:

### Branch-based Deployment

| Branch | Deployment Location | URL Pattern |
|--------|-------------------|-------------|
| `main` | Root directory (`/`) | `https://FIDEScommunity.github.io/DIIP` |
| `develop` | `/draft` directory | `https://FIDEScommunity.github.io/DIIP/draft` |
| Pull Requests | `/pr/pr-{number}` directory | `https://FIDEScommunity.github.io/DIIP/pr/pr-123` |

### Workflow Features

#### Main Branch (`main`)

- Deploys to the root of GitHub Pages
- Also creates a copy at `/latest` for backwards compatibility
- Represents the stable, published version of DIIP

#### Develop Branch (`develop`)

- Deploys to `/draft` subdirectory
- Creates a commit status on the latest commit with the preview URL
- Used for reviewing upcoming changes before they go to main

#### Pull Requests

- Automatically creates preview deployments for all PRs
- Each PR gets its own subdirectory: `/pr/pr-{number}`
- Comments on the PR with the preview URL
- Updates the comment when new commits are pushed
- Automatically cleans up the preview when the PR is closed/merged

### Permissions Required

The workflow requires the following permissions:

- `contents: read` - To checkout the repository
- `pages: write` - To deploy to GitHub Pages
- `id-token: write` - For GitHub Pages deployment authentication
- `pull-requests: write` - To comment on pull requests

### File Structure After Deployment

```
gh-pages branch:
├── index.html (main branch content)
├── latest.html (copy of main branch)
├── assets/
├── custom-assets/
├── draft/
│   └── index.html (develop branch content)
└── pr/
    ├── pr-123/
    │   └── index.html (PR #123 content)  
    └── pr-456/
        └── index.html (PR #456 content)
```

### Maintenance

The workflow automatically handles cleanup:

- PR preview directories are removed when PRs are closed
- No manual intervention needed for routine deployments
- Failed deployments can be re-triggered by pushing a new commit or re-running the workflow