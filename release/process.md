# Spinnaker Release Process Technical Design Document

## Objective

This document describes the technical approach to implementing the release process specified in the +Spinnaker Releases - Process Doc.    Instead of “reinventing the wheel” we’ve taken some processes from the Kubernetes release process and other open source projects and have applied what is applicable for the Spinnaker community below.  The first part of the process is create a spinnaker `release candidate` that will then be tested & verified by a subsequent process.  Once that process has succeeded then the `release candidate` it will be promoted as an official release.


### Goals
1.  It should be clear to the user which version of Spinnaker they’re currently using, installing or configuring.
2. If a user want to install/configure/update sub-components it should be clear which set of components are correspond to the same release.
3. User should be able to easily rollback to previous version for all sub-components
4. The user should have a high degree of confidence that upgrading will succeed and their Spinnaker system will continue working as expected.
5. It should be easy to add patch fixes to any official release of Spinnaker within the period specified by the release documentation.

# Branching

At the specified time period by the release process document an automated process will create a branch for each component (e.g. clouddriver, echo, deck, etc) for the given release.  This will lock-down the given features for a given release to those that were already merged to master and help keep the upcoming release stable.

For spinnaker release `1.4` we would create a branch called  `release-1.4`.  This branch will be long-lived and used for release candidates, releases and subsequent patch fixes.

# Tagging

At the time of the initial release branch we will use `HEAD`  of the branch, e.g. `release-1.4` as the release candidate to be tested.    Once the candidate has been verified as a good release, it will be tagged with the official release tag, e.g. `release-1.4.0` .  Note that we prefix the tag with `release` as to not collide with the existing continuous release process that works today.


# Patches & Bug Fixes

If we need to make changes or fixes bug for an official release, for instance `release-1.4.0` , a developer will clone/branch from the master of the existing `release-1.4` branch, fix the bug and then issue a pull request back into the upstream `release-1.4` branch.

The build process, which runs at the specified period by the release document, will be responsible for merging any new commits from the `release-1.4`  branch into the master branch, building & verifying the package, and create a new tag.  The new tag will be based off the last tag created and increments the patch number.  For example, if the last tag was `release-1.4.1`  it will increment the patch number and create a new tag `release-1.4.2`

# Packaging & Building

The automated will be responsible for kicking-off the build process for the official Spinnaker release.  The build process should look for the latest branch version, find if there are any changes between the last sementic tag and if so, we will need to cut a new release for all components of Spinnaker.

## Release Notes

The notes will be curated as part of another process described in the Spinnaker Release Process doc.  Once that document is curated and signed off by all parties, we’ll append the release notes to a file called `CHANGELOG`.  Gitlab has a good example of a concise changelog which tracks all version over time.   

## External Dependencies

These dependencies are defined as services that are not created or actively developed by the Spinnaker community.  These will not be part of the release process nor packaged.  They will however be tested as part of the release process.  This will prevent tight coupling of external dependencies that might be managed with external providers, e.g. redis, or swapped for another dependency such as choosing nginx over apache in the deployment.

## Publishing

These new release candidate release should be placed in a new repository under the `unstable` target in the Debian repository until it’s ready to be promoted, see the section +Spinnaker Release Process Technical Design Document: Testing below for more details.  

Currently spinnaker packages are published to `bintray.com/spinnaker/debians` but we suggest that publishing to a new repo would reduce confusion.   We suggest that the repo be named: `bintray.com/spinnaker/deb-releases` .

This helps a user differentiate between an official “Spinnaker” release a group of tested micro-services vs micro-services that are incrementing versions independently.

Users can then independently install the components independently but at the same time making sure they’re all tied to a release:

`apt-get install spinnaker-release-orca=1.1.0 `

`apt-get install spinnaker-release-clouddriver=1.1.0`  

`apt-get install spinnaker-release-echo=1.1.0`

`apt-get install spinnaker-release-fiat=1.1.0`

`apt-get install spinnaker-release-deck=1.1.0`

`apt-get install spinnaker-release-igor=1.1.0`

`apt-get install spinnaker-release-gate=1.1.0`

`apt-get install spinnaker-release-front50=1.1.0`

# Testing

Once we have a release candidate, we’ll want to run a series of integration tests against it to make sure the sub-services work well together for all the supported cloud providers.

## Configure a Spinnaker Instance

We will stand up and configure a Spinnaker instance.  We can leverage the existing packer templates that exist for GCP and extend them to other cloud environments.  In the future we hope that Halyard will be used to deploy and configure the instances.

## Run Integration Tests

Google’s work on the ci-test as testing the framework is a good basis to begin adding tests for other cloud environments.  Currently the project is highly focus on deployments for GCP but we will need to build out functionality for other target cloud environments.

## Publishing the verified versions

Once the release candidate has been validated and passes all tests, we can migrate the package from the `unstable`  deploy target to the `stable` deploy target.
