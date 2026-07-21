# Contributing

## Before You Start

Read the root README and [docs/API.md](API.md). The project currently contains a minimal Lambda handler and deployment scaffolding; do not assume that AWS, Docker, or Kubernetes resources exist.

## Development Setup

Use Python 3.8 to match the current GitHub Actions workflow, or explicitly document and test a runtime upgrade. Create a virtual environment and install:

```bash
python -m pip install -r function/requirements.txt
python -m pip install flake8
```

## Change Process

1. Create a focused branch from `main`.
2. Explain the behavior and deployment impact in the pull request.
3. Add or update tests for behavior changes.
4. Run lint and the local handler check before pushing.
5. Keep generated ZIPs, credentials, `.env` files, and build output out of Git.
6. Request review for changes to workflows, permissions, dependencies, or deployment behavior.

## Quality Gates

The minimum checks are:

```bash
flake8 function --count --select=E9,F63,F7,F82 --show-source --statistics
python -c "from function.lambda_function import lambda_handler; assert lambda_handler({'input': 'Hello'}, None) == 'World'"
```

New behavior should include automated tests for valid input, missing input, and invalid input. Do not weaken the CI lint failure gate to hide defects.

## Commit and Pull Requests

Use clear, imperative commit messages. Pull requests should include:

- summary of the change;
- test commands and results;
- configuration or environment-variable changes;
- security and rollback considerations for deployment changes;
- documentation updates where behavior or operations changed.

Keep changes small and avoid unrelated formatting. Workflow and IAM changes require particular care because they can expand deployment privileges.

## Dependency and Security Rules

Pin or constrain dependencies, review Dependabot proposals, and remove unused imports and packages. Never commit access keys, tokens, passwords, private certificates, or production event data. Report suspected vulnerabilities privately to the repository maintainers rather than opening a public issue with exploit details.

## License

No license file is currently present. Until a license is added, contributions are accepted only with maintainer approval and should not be redistributed as open-source software.
