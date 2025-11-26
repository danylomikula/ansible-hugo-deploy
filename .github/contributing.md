# Contributing

When contributing to this repository, please first discuss the change you wish to make via issue or any other method with the repository owners before making a change.

## Pull Request Process

1. Update the README.md with details of changes to variables, features, or configuration options if applicable.
2. Run pre-commit hooks `pre-commit run -a`.
3. Ensure ansible-lint passes: `ansible-lint`.
4. Test your changes on Rocky Linux 10.
5. Once all outstanding comments and checklist items have been addressed, your contribution will be merged! Merged PRs will be included in the next release.

## Checklists for contributions

- [ ] Add [semantic prefix](#semantic-pull-requests) to your PR or Commits (at least one of your commit groups)
- [ ] CI tests are passing
- [ ] README.md has been updated after any changes to variables and features
- [ ] Run pre-commit hooks `pre-commit run -a`
- [ ] ansible-lint passes
- [ ] Tested on Rocky Linux 10

## Semantic Pull Requests

To generate changelog, Pull Requests or Commits must have semantic prefix and follow conventional specs below:

- `feat:` for new features (minor version bump)
- `fix:` for bug fixes (patch version bump)
- `docs:` for documentation and examples
- `refactor:` for code refactoring
- `test:` for tests
- `ci:` for CI purpose
- `chore:` for chores stuff

The `chore` prefix is skipped during changelog generation. It can be used for `chore: update changelog` commit message by example.

## Development Setup

```bash
# Install Ansible dependencies
ansible-galaxy collection install -r requirements.yml

# Install pre-commit hooks
pip install pre-commit
pre-commit install

# Test your changes
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --check  # Dry run
ansible-playbook site.yml --tags caddy,firewall  # Test specific roles
```
