# Contributing to GitHub Actions Library

Thank you for your interest in contributing! This document provides guidelines for contributing to this repository.

## Code of Conduct

Please be respectful and constructive in all interactions.

## How to Contribute

### Reporting Issues

- Check existing issues before creating a new one
- Use issue templates when available
- Provide clear reproduction steps

### Submitting Changes

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/my-feature`
3. **Make your changes**
4. **Test locally** (see Testing section)
5. **Commit with conventional commits**: `git commit -m "feat: add new workflow"`
6. **Push and create a Pull Request**

## Development Guidelines

### Workflow Standards

1. **SHA-pin all actions** - Use full commit SHAs, not version tags:
   ```yaml
   # ✅ Good
   uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
   
   # ❌ Bad
   uses: actions/checkout@v4
   ```

2. **Minimal permissions** - Request only necessary permissions:
   ```yaml
   permissions:
     contents: read
     packages: write
   ```

3. **Document inputs/outputs** - All inputs and outputs must have descriptions

4. **Use workflow_call** for reusable workflows:
   ```yaml
   on:
     workflow_call:
       inputs:
         my-input:
           description: 'Clear description'
           required: true
           type: string
   ```

### Composite Action Standards

1. **Include branding** - Add icon and color
2. **Provide outputs** - Expose useful outputs
3. **Handle errors** - Use `continue-on-error` where appropriate
4. **Generate summaries** - Use `$GITHUB_STEP_SUMMARY` for visibility

### Testing

Before submitting:

1. Validate YAML syntax:
   ```bash
   python3 -c "import yaml; yaml.safe_load(open('path/to/file.yml'))"
   ```

2. Run actionlint:
   ```bash
   actionlint
   ```

3. Test in a fork with actual workflow runs

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` New features
- `fix:` Bug fixes
- `docs:` Documentation changes
- `refactor:` Code refactoring
- `chore:` Maintenance tasks

## Pull Request Process

1. Update documentation for any changed functionality
2. Ensure all CI checks pass
3. Request review from maintainers
4. Squash commits before merging

## Questions?

Open a discussion or issue if you need help!
