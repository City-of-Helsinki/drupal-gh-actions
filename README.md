# Drupal GitHub Actions

Provides re-usable workflow files for Drupal projects.

## Artifact build

See https://github.com/City-of-Helsinki/drupal-helfi-platform/blob/main/documentation/automatic-updates.md#2-enable-artifact-action for documentation how to use this workflow.

## Update config

See https://github.com/City-of-Helsinki/drupal-helfi-platform/blob/main/documentation/automatic-updates.md#adding-update-bot-to-your-project for documentation how to use this workflow.

Required secrets:
- `automatic_update_token`: Token used to trigger `peter-evans/create-pull-request` action. This is required so we can run tests against the PR.

## Project tests

Available configuration:

- `check_security_updates`: Whether to run `composer audit` command to scan security updates. Defaults to `true`.
- `check_config_language`:  Check if configuration in `conf/cmi` folder is exported using correct `langcode` (`und` or `en`). Defaults to `true`.

```yaml
# .github/workflows/test.yml
on:
  pull_request:
  push:
    branches: ['main', 'dev']
name: CI
jobs:
  tests:
    uses: city-of-helsinki/drupal-gh-actions/.github/workflows/project-tests.yml@main
    with:
      check_security_updates: true
      check_config_language: true
```

## Module tests

Available configuration:

- `php_version`: Required. The PHP version used to run tests. For example `8.3`.
- `composer_dev_dependencies`: Optional. Use this to install module's dev dependencies. For example: `drupal/redirect drupal/openid_connect`.
- `phpcs_args`: Optional. Arguments passed to `phpcs` command.

Required secrets:

- `codecov_token`: Project's codecov token. Can be found from project's Configuration -> General on Codecov.

```yaml
# .github/workflows/ci.yml
on:
  pull_request:
  push:
    branches:
      - main
name: CI
jobs:
  tests:
    strategy:
      matrix:
        php-versions: ['8.3']
    uses: city-of-helsinki/drupal-gh-actions/.github/workflows/module-tests.yml@main
    with:
      php_version: ${{ matrix.php-versions }}
    secrets:
      codecov_token: ${{ secrets.CODECOV_TOKEN }}
```

## NPM Audit

Updates HDBT Subtheme Node.js dependencies using `npm audit`.
