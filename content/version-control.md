+++
title = "Version Control"
weight = 200
+++

For both philosophical and practical reasons, any asset of the AAS WorldWide
Telescope project that can be managed in a [version control] system should be.
This includes not only computer source code but also documentation (in
plain-text formats like [Markdown]) and configuration files. In fact, the
advantages of being able to plug into the modern version control ecosystem are
so great that it is worth putting significant effort into finding or creating
schemes that allow project assets to be managed in the version-control
paradigm, or into transforming existing assets into version-control-friendly
formats.

[version control]: https://en.wikipedia.org/wiki/Version_control
[Markdown]: https://commonmark.org/


# Git

At present, WWT source assets are managed in a suite of numerous [Git]
repositories. While it is beyond the scope of this document to explain the
finer points of Git usage, suffice it to say that familiarity with Git will be
extremely valuable if one wants to contribute to WWT development.

[Git]: https://git-scm.com/


# GitHub

The WWT source repositories are made available through [GitHub], in the
[WorldWideTelescope organization][github-org].

[GitHub]: https://github.com/
[github-org]: https://github.com/WorldWideTelescope/

Besides making the WWT sources publicly available, GitHub enables
collaborative development of the WWT project through a standard
[fork-and-pull] workflow, as well as issue tracking and other standard
software development infrastructure support. For more information, see the
[Contributors’ Guide][contrib-guide].

[fork-and-pull]: https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-collaborative-development-models
[contrib-guide]: https://worldwidetelescope.github.io/contributing/


# Continuous Integration and Deployment

To the greatest extent possible, we aim to automate the creation and
deployment of products derived from WWT sources using
[continuous integration and deployment][ci-cd] (CI/CD) techniques. In fact, a
major motivation for tracking as many WWT source assets in Git as possible is
purely to be able to take advantage of the conveniences afforded by modern
CI/CD systems.

[ci-cd]: https://www.redhat.com/en/topics/devops/what-is-ci-cd

The bulk of WWT’s CI/CD tooling is powered by the [Azure Pipelines] service
and managed through the [aasworldwidetelescope organization][devops-org] on
Azure DevOps. You shouldn’t need to be a member of this organization to
perform most WWT development tasks, but certain administrative activities will
require membership privileges.

[Azure Pipelines]: https://azure.microsoft.com/en-us/services/devops/pipelines/
[devops-org]: https://dev.azure.com/aasworldwidetelescope/

The details of the specific CI/CD workflows used vary from one repository to
another and are documented elsewhere in this guide. The
[Azure Pipelines build logs][devops-pipelines] are generally located under the
[WWT][devops-project] project of the DevOps organization.

[devops-pipelines]: https://dev.azure.com/aasworldwidetelescope/WWT/_build
[devops-project]: https://dev.azure.com/aasworldwidetelescope/WWT/

The [wwt-windows-client] repository require the use of a custom “agent” to run
the builds, the construction of which is documented in [an
Appendix][vmss-appendix].

[wwt-windows-client]: https://github.com/WorldWideTelescope/wwt-windows-client/
[vmss-appendix]: @/appendix-build-agent.md

Some WWT repositories may be wired up to invoke other free and public CI/CD
services. In general, the goal is to transition these to use Azure Pipelines
through the Azure DevOps organization.
