# lerna-travis

A script for integrating lerna projects with Travis CI/CD.

## Installation
Install using

```bash
# If using npm
npm install -D @k88/lerna-travis

# If using yarn
yarn add -D @k88/lerna-travis
```

## Usage

Add the following scripts to your package.json's `scripts`:

```json
{
  "scripts": {
    "release:alpha": "lerna-travis-release alpha",
    "release:beta": "lerna-travis-release beta",
    "release:public": "lerna-travis-release public",
    "publish": "lerna-travis-publish"
  }
}
```

## Details

The `release:*` scripts are invoked from your local machine; it will create a release tag and push the changes up.

At this point, your CI/CD pipeline should be configured to run to invoke the `publish` script that would then perform the following tasks:

1) Check publish version (see below for more detail)
2) Removes all dist/lib/node_modules directory
3) Performs a fresh `npm install`
4) Runs `npm run lint`
5) Runs `npm run test`
6) Runs `npm run build`
7) Publishes the packages

### Publish types

This script safeguards performing `public/beta/alpha` publication based on:

* `public` may only run on `main` branch
* `beta` may only run on `v\d-beta` branch (i.e. `v1-beta`, `v2-beta`, `v3-beta`, etc)
* `alpha` may only run on non-beta/non-publich branches

### Distribution Tags

The following tags are published:

* The `public` publish pushes a `latest` dist tag
* The `beta` publish pushes a `beta` dist tag
* The `alpha` publish pushes a `alpha` dist tag

### Version bump

You can pass an optional `patch/minor/major` argument to change the version bump. By default, a `patch` is published. Some examples are:

```bash
# Publishes a patch beta
npm run release:beta

# Publishes a minor public
npm run release:public minor
```

## Setting up Travis

Generate an NPM token from your NPM profile and add it to Travis as `NPM_TOKEN`. Here is an example of your `.travis.yml` file:

```yaml
before_deploy:
  - npm config set access public
  - npm config set registry https://registry.npmjs.org
  - npm set //registry.npmjs.org/:_authToken "$NPM_TOKEN"

deploy:
  provider: script
  script: "npm run publish"
  skip_cleanup: true
  on:
    tags: true
    branch: main
    repo: YOUR-ORG/YOUR-REPO
branches:
  - main
  - /^v\d+\.\d+\.\d+.*$/
```