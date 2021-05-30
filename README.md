# Heroku Node.js Deploy Subfolder

![GitHub issues](https://img.shields.io/github/issues/onekiloparsec/heroku-node-deploy-subfolder.svg)
![GitHub](https://img.shields.io/github/license/onekiloparsec/heroku-node-deploy-subfolder.svg)

This is a simple GitHub action that allows you to deploy a Node.js project to Heroku. It is a slightly simpler, more focused, and safer fork from [AkhileshNS/heroku-deploy](https://github.com/AkhileshNS/heroku-deploy).

It is focused on doing only one thing: deploy through `git` (not Docker) a Node.js project built inside a subfolder (e.g. `dist`). It uses `git subtree split` to achieve that. Note that you must specify all parameters. The Heroku process is the `web` one, obviously.

The  `bash/git` command executed is the following (assuming the `heroku` remote has been sucessfully set):

```
git push heroku \`git subtree split --prefix ${subfolder} HEAD\`:${heroku_branch} --force""
```

Note that:
* We **do not** double-check for the correct branch anymore (we used to do it), because this is something that the CI file must take care of.
* We always use `--force` to ensure the push is always working, even if in the meantime some manual deploy have been triggered from another place.

# Usage

In order to use the action in your workflow, just add in your _.github/workflows/YOURACTION.yml_ and fill the `with` parameters. **Make sure your `dist` subfolder is commited (otherwise the `git subtree split` won't detect any change).**

# Example

Below is an example with two build-and-deploy files, one using staging and one for prod.

```yaml
name: my-awesome-actions-jobs-staging

on:
  push:
    branches:
      - develop
      
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    needs: test-unit
    if: github.ref == 'refs/heads/develop'
    steps:
      - uses: actions/checkout@v2
      # possibly setup node action here...
      # possibly cache of node modules action here...
      # build steps here...
      - uses: onekiloparsec/heroku-node-deploy-subfolder@v1.1.0
        with:
          api_key: ${{secrets.HEROKU_API_KEY}}
          email: ${{secrets.HEROKU_EMAIL}}
          app_name: <APP_NAME_STAGING>
          heroku_branch: "master"
          subfolder: "dist-staging"
```

```yaml
name: my-awesome-actions-jobs-prod

on:
  push:
    branches:
      - master
      
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    needs: test-unit
    if: github.ref == 'refs/heads/master'
    steps:
      - uses: actions/checkout@v2
      # possibly setup node action here...
      # possibly cache of node modules action here...
      # build steps here...
      - uses: onekiloparsec/heroku-node-deploy-subfolder@v1.1.0
        with:
          api_key: ${{secrets.HEROKU_API_KEY}}
          email: ${{secrets.HEROKU_EMAIL}}
          app_name: <APP_NAME_PROD>
          heroku_branch: "master"
          subfolder: "dist-prod"
```

# License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/onekiloparsec/heroku-node-deploy-subfolder/blob/master/LICENSE) file for details
