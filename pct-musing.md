# PDK Templates ToC


## config_defaults.yml

contains a lot of default config values for all templates, including making some optional

## moduleroot/

Templates that are updated on every `pdk update`/`convert` run:

### CI-as-a-Service configs

appveyor is a CI-as-a-Service for windows machines.
gitlab is a self-hosted git repo server and CI system.
travis is a (pre-dominantely) linux-based CI service.

Each of these files configures unit and/or acceptance tests for the respective platform.

    moduleroot/appveyor.yml.erb
    moduleroot/.gitlab-ci.yml.erb
    moduleroot/.travis.yml.erb

Global/shared config:
- branch selection for running CI
- code coverage enabled y/n
- requires specific entries in the Gemfile, .fixtures.yml.erb to work

Maintenance status: IAC doesn't use any of these services

Recommendation: move to unsupported test modules, one per provider

### GitHub Actions

Two groups of workflows here: Testing and Releasing.
The two groups need to be handled separately.

The testing workflows cover PR testing (unit and acceptance) as well as scheduled testing (acceptance).
Acceptance tests rely on puppet-private infrastructure (Cloud CI) for some OSs.
We can improve litmus' `matrix_from_metadata` scripts to support a docker-only mode to make this useful for the community.
We can investigate using appveyor-style `localhost` testing for windows to give some coverage for that platform by default.
We can investigate charging money for access to Cloud CI for non-open-source repos.

All testing workflows use honeycomb integrations for global tracking.
This is optional and can be disabled by default.

    moduleroot/.github/workflows/nightly.yml.erb
    moduleroot/.github/workflows/pr_test.yml.erb
    moduleroot/.github/workflows/spec.yml.erb

Global/shared config:
- `owner` for avoiding running the workflow on forks
- `honeycomb` settings
- `service_url` for cloud ci
- `slack-notifications`
- requires specific entries in the Gemfile, .fixtures.yml.erb to work

Maintenance status: in active use by IAC

Recommendation: move to semi-supported ci-github template; might require additional work to make fully useful for community

The auto_release workflow creates a release-prep PR with changelog, reference.md and a version number bump.
The release workflow takes a branch, and builds and pushes the module to the forge.

The auto_release workflow uses honeycomb integrations for global tracking.
This is optional and can be disabled by default.

    moduleroot/.github/workflows/auto_release.yml.erb
    moduleroot/.github/workflows/release.yml.erb

Global/shared config:
- `owner` for avoiding running the workflow on forks
- `honeycomb` settings
- requires specific entries in the Gemfile, Rakefile to work

Maintenance status: in active use by IAC

Recommendation: move to semi-supported github-release template

### VSCode infrastructure

Infrastructure for VSCode and its Remote extension

    moduleroot/.devcontainer/devcontainer.json
    moduleroot/.devcontainer/Dockerfile
    moduleroot/.vscode/extensions.json

Maintenance status: supported by James(?)

Global/shared config: none

Recommendation: move to supported editor-vscode template

### editorconfig

see [editorconfig.org](https://editorconfig.org/) for background.
tldr: ensures a common file ending, indentation style and other minutiae across many editors.

    moduleroot/.editorconfig.erb

Maintenance status: unknown

Global/shared config: none or all of it

Suggestion: move to supported editor-vscode template, add editorconfig plugin to recommended VSCode plugin set


### Gemfile

The Gemfile is a basic list of gems required for PDK commands and some hidden features (litmus).
A lot of the selection happens through puppet-module-gems, but some additional customization (like puppet-lint plugins, github_changelog_generator, puppet/facter version selection) regularly happens here.

    moduleroot/Gemfile.erb

Maintenance status: actively used

Global/shared config:
- `extra_gemfiles`: a list of extra gemfiles read in to allow user- and module-specific extensions; currently hardcoded to `[Gemfile.local, ~/.gemfile]`.
- `disable_legacy_facts`

Recommendation: keep in core module template

### Rakefile

The Rakefile implements some of the tasks the PDK calls and provides some configuration for other bits (e.g. github_changelog_generator).
Most of the actual functionality is pulled in through `require`s from other gems (puppet_litmus, puppetlabs_spec_helper, puppet-syntax, puppet_blacksmith, github_changelog_generator, puppet-strings)

    moduleroot/Rakefile.erb

Maintenance status: actively used

Global/shared config: none

Recommendation: keep in core module template

### RSpec

There's a number of default configurations shipped to make it less awful for newcomers.

    moduleroot/.rspec.erb
    moduleroot/spec/default_facts.yml.erb
    moduleroot/spec/spec_helper.rb.erb

Maintenance status: actively used

Global/shared config: none

Recommendation:
- keep in core module template???
- drop .rspec.erb as pdk provides its own UI for test results and doesn't use the options specified here
- move default default_facts to rspec-puppet(?) or rspec-puppet-facts(?) and leave empty example file(???)
- move many of the defaults in `spec_helper.rb.erb` to `puppetlabs_spec_helper/module_spec_helper` to reduce the amount of code carried here

### git infrastructure files

These files provide sensible defaults when using git.

    moduleroot/.gitattributes.erb
    moduleroot/.gitignore.erb

Maintenance status: actively used

Global/shared config: shares ignore defaults with .pdkignore

Recommendation: move to a vcs-git module

### .pdkignore

List files that must not be packed into a module package.

    moduleroot/.pdkignore.erb

Maintenance status: actively used

Global/shared config: shares ignore defaults with .gitignore

Recommendation: keep in core module template

### Gitpod

Support files for [gitpod.io](https://www.gitpod.io/).

    moduleroot/.gitpod.Dockerfile
    moduleroot/.gitpod.yml

Maintenance status: unmaintained

Global/shared config: none

Recommendation: move to unsupported editor-gitpod module; needs to be updated for recent pdk and vscode-plugin versions

### PDK validate

Configuration defaults for puppet-lint and rubocop

    moduleroot/.puppet-lint.rc.erb
    moduleroot/.rubocop.yml.erb

Maintenance status: actively used

Global/shared config:
- `.puppet-lint.rc.erb` takes some values from the Rakefile config

Recommendation: keep in core module template

### YARD

Configuration for puppet-strings to generate the REFERENCE.md document

    moduleroot/.yardopts.erb

Maintenance status: actively used(???)

Global/shared config: none

Recommendation:
- keep in core module template
- investigate ways to configure YARD defaults when generating docs instead of hardcoding

## moduleroot_init/

This directory contains some template files that get deployed on `pdk new module` and are not touched by the PDK afterwards.

### Newcomer documentation

These files contain various hints and tips on possible configuration touchpoints.

    moduleroot_init/CHANGELOG.md.erb
    moduleroot_init/.fixtures.yml.erb
    moduleroot_init/README.md.erb
    moduleroot_init/.sync.yml.erb

Maintenance status: unmaintained

Global/shared config: none

Recommendation: re-consider their existence

### Default hiera data-in-module config

see https://www.devco.net/archives/2013/12/08/better-puppet-modules-using-hiera-data.php

    moduleroot_init/data/common.yaml
    moduleroot_init/hiera.yaml.erb

Maintenance status: unmaintained

Global/shared config: none

Recommendation: re-consider their existence

## object_templates/

Object templates for the various `pdk new X` commands

    object_templates/class.erb
    object_templates/class_spec.erb
    object_templates/defined_type.erb
    object_templates/defined_type_spec.erb
    object_templates/fact.erb
    object_templates/fact_spec.erb
    object_templates/functions
    object_templates/functions/function_spec.erb
    object_templates/functions/native_function.erb
    object_templates/functions/v4_function.erb
    object_templates/plan.erb
    object_templates/provider.erb
    object_templates/provider_spec.erb
    object_templates/provider_type.erb
    object_templates/provider_type_spec.erb
    object_templates/task.erb
    object_templates/transport_device.erb
    object_templates/transport.erb
    object_templates/transport_spec.erb
    object_templates/transport_type.erb
    object_templates/transport_type_spec.erb

## rubocop tooling

Additional scripts to prepare the data needed to render the rubocop base profiles.

    rubocop/cleanup_cops.yml
    rubocop/defaults-1.6.1.yml
    rubocop/profile_builder.rb
    rubocop/README.md
