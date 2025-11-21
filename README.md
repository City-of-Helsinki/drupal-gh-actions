# Drupal GitHub Actions

Provides re-usable workflow files for Drupal projects.

## Artifact build

See [documentation](https://github.com/City-of-Helsinki/drupal-helfi-platform/blob/main/documentation/automatic-updates.md#2-enable-artifact-action) on enabling the artifact action
for details on how to use this workflow.

Available configuration:
- `sentry_crons`: The full URL to Sentry monitoring. Example: `https://example.com/api/0/cron/<monitor_slug>/examplePublicKey/`


### Sentry monitoring

Sentry Cron monitoring ensures the Artifact build runs as expected. To set it up:

1. In Sentry, go to **Crons → Add Monitor** and use the following settings:
   - **Name**: GitHub Artifact
   - **Project**: Your project
   - **Schedule**: Select Cron. The value should match your project's schedule in the `artifact.yml` file. Default is `0 0 * * 0`.
   - **Owner**: `#drupal-developers`
2. In your project’s repository, go to: **Settings → Secrets and variables → Actions → New repository secret**
3. Add a secret called `SENTRY_CRONS`. Its value is the Sentry ingest URL combined with your monitor slug and public key, e.g.: `https://example.com/api/0/cron/<monitor_slug>/examplePublicKey/`
4. Ensure your project’s `artifact.yml` passes the `sentry_crons` secret to the parent workflow:
```yaml
# .github/workflows/artifact.yml
...
jobs:
  build:
    ...
    secrets:
      sentry_crons: ${{ secrets.SENTRY_CRONS }}
```

## Update config

See https://github.com/City-of-Helsinki/drupal-helfi-platform/blob/main/documentation/automatic-updates.md#adding-update-bot-to-your-project for documentation how to use this workflow.

Required secrets:
- `automatic_update_token`: Token used to trigger `peter-evans/create-pull-request` action. This is required so we can run tests against the PR.

## Project tests

Available configuration:

- `check_security_updates`: Whether to run `composer audit` command to scan security updates. Defaults to `true`.
- `check_config_language`:  Check if configuration in `conf/cmi` folder is exported using correct `langcode` (`und` or `en`). Defaults to `true`.
- `check_config_language_ignore_files`: Comma separated list of patterns to exclude files from language config check
  - Useful for e.g. sites with webforms only on Finnish pages
  - Use `*` as wildcard to exclude multiple config files with a pattern

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
      check_config_language_ignore_files: "webform.webform.*,views.view.content.yml"
```

## Module tests

Available configuration:

- `php_version`: Required. The PHP version used to run tests. For example `8.3`.
- `composer_dev_dependencies`: Optional. Use this to install module's dev dependencies. For example: `drupal/redirect drupal/openid_connect`.
- `phpcs_args`: Optional. Arguments passed to `phpcs` command.

Required secrets:

- `sonarcloud_token`: Project's SonarCloud token.

Optional secrets:

- `base64_app_secrets`: A base64 encoded JSON string. For example `{"my_key": "my value"}`. These will be written to project's root in a `.secrets.json` file and can be accessed in tests using `Drupal\Tests\helfi_api_base\Traits\SecretsTrait`.

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
      sonarcloud_token: ${{ secrets.SONARCLOUD_TOKEN }}
```

## NPM Audit

Updates HDBT Subtheme Node.js dependencies using `npm audit`.
