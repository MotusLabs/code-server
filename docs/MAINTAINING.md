<!-- prettier-ignore-start -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
# Maintaining

- [Releasing](#releasing)
    - [Release Candidates](#release-candidates)
    - [Release assets](#release-assets)
    - [Docker](#docker)
- [Testing](#testing)
- [Documentation](#documentation)
  - [Troubleshooting](#troubleshooting)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
<!-- prettier-ignore-end -->

We keep code-server up to date with VS Code releases (there are usually two or
three a month) but we are not generally actively developing code-server aside
from fixing regressions.

Most of the work is keeping on top of issues and discussions.

## Releasing

1. Check that the changelog lists all the important changes.
2. Make sure the changelog entry lists the current version of VS Code.
3. Go to GitHub Actions > Draft release > Run workflow on the commit you want to
   release. For the version we match VS Code's minor and patch version. The
   patch number may become temporarily out of sync if we need to put out a
   patch, but if we make our own minor change then we will not release it until
   the next minor VS Code release.
4. CI will build an NPM package and platform-specific packages, and upload those
   to a draft release.
5. Update the resulting draft release with the changelog contents.
6. Publish the draft release after validating it.
7. Update the changelog with the release date and bump the Helm chart version
   once the Docker images have published.

#### Release Candidates

We prefer to do release candidates so the community can test things before a
full-blown release. To do this follow the same steps as above but:

1. Add a `-rc.<number>` suffix to the version.
2. When you publish the release select "pre-release". CI will not automatically
   publish pre-releases.
3. Do not update the chart version or merge in the changelog until the final
   release.

#### Release assets

Every release carries the platform-agnostic `package.tar.gz` plus the
platform-specific `.deb`, `.rpm` and standalone tarballs, attached to the
release in this repository. `install.sh` pulls from here, so it installs this
fork's build and not upstream's.

#### Docker

We publish code-server as a Docker image to this repository's GitHub Container
Registry namespace, `ghcr.io/motuslabs/code-server`, tagging it both with the
version and latest. This is automated with the release process and runs off the
`.deb` and `.rpm` from the release above.

Consumers need a GitHub login with read access to the organization's packages,
or the package has to be made visible to the organization in its package
settings.

This fork does not publish to npm, Docker Hub, the AUR, Homebrew or nixpkgs.
Nothing in CI should push outside the organization; if a build needs to be
shared elsewhere, that is a decision to make deliberately rather than a default.

## Testing

Our testing structure is laid out under our [Contributing
docs](https://coder.com/docs/code-server/latest/CONTRIBUTING#test).

If you're ever looking to add more tests, here are a few ways to get started:

- run `npm run test:unit` and look at the coverage chart. You'll see all the
  uncovered lines. This is a good place to start.
- look at `test/scripts` to see which scripts are tested. We can always use more
  tests there.
- look at `test/e2e`. We can always use more end-to-end tests.

Otherwise, talk to a current maintainer and ask which part of the codebase is
lacking most when it comes to tests.

## Documentation

### Troubleshooting

Our docs are hosted on [Vercel](https://vercel.com/). Vercel only shows logs in
realtime, which means you need to have the logs open in one tab and reproduce
your error in another tab. Since our logs are private to Coder the organization,
you can only follow these steps if you're a Coder employee. Ask a maintainer for
help if you need it.

Taking a real scenario, let's say you wanted to troubleshoot [this docs
change](https://github.com/coder/code-server/pull/4042). Here is how you would
do it:

1. Go to https://vercel.com/codercom/codercom
2. Click "View Function Logs"
3. In a separate tab, open the preview link from github-actions-bot
4. Now look at the function logs and see if there are errors in the logs
