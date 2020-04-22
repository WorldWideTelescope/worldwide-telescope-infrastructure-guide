+++
title = "Documentation and Other Static HTML"
weight = 400
+++

WWT documentation, such as the page that you’re reading right now, is
version-controlled in the [WWT GitHub organization][github-org] just like any
other project assets. The WWT documentation was historically split among
several documents, and the current collection has synthesized many different
materials, and so the number of repositories is substantial. The
[Documentation Hub][docs.wwt.o] attempts to provide an exhaustive list of
documents, with links to source repositories found in each document’s
navigation links.

[github-org]: https://github.com/WorldWideTelescope/
[docs.wwt.o]: https://docs.worldwidetelescope.org/

At present, the Git repositories for WWT guides and manuals are essentially
simple [static HTML websites][static-sites] generated from [Markdown] files.
In particular, they’re generated using a tool called [Zola], which is easy to
install, self-contained, and very fast. Since the bulk of the content is in
standard Markdown, conversion to other formats should be straightforward if
needed.

[static-sites]: https://gohugo.io/about/benefits/
[Markdown]: https://commonmark.org/
[Zola]: https://www.getzola.org/

Certain other WWT websites that aren’t quite “documentation”, such as the
[Documentation Hub][docs.wwt.o] index page, are also generated using Zola.


# docs.worldwidetelescope.org

Most WWT documentation is served off of the
[docs.worldwidetelescope.org][docs.wwt.o] domain. This domain is essentially a
purely static website — the domain is a CDN endpoint that serves content off
of `_docs` subdirectory of the `wwtwebstatic` Azure Storage account associated
with the WWT Azure subscription. Content is updated on this storage account
using the CI/CD process described below.


# WWT Guide Zola Theme

Most WWT documentation uses the same
[Zola theme](https://www.getzola.org/themes/) to provide a consistent look and
feel. The theme is version-controlled in the [zola-wwtguide] repository.

[zola-wwtguide]: https://github.com/WorldWideTelescope/zola-wwtguide

For each individual guide, the theme is imported as a [Git submodule] at the
path `themes/wwtguide`, which means that all of the usual pitfalls from Git
submodules come into play. In particular, each guide repository specifies the
exact commit from the theme repository that it references. So, if an important
change is made to the theme that should be applied globally, you need to go to
each and every guide repository and update it to point to the latest version
of the theme repository. This is a bit of a hassle, but the flip side is
enhanced reproducibility and reliability, and we don’t expect the theme to
undergo major upgrades very often.

[Git submodule]: https://git-scm.com/book/en/v2/Git-Tools-Submodules


# Deployment Process

Deployment of WWT guides and manuals follows the general scheme seen elsewhere
in the project. Merges to the `master` branch of most documentation
repositories trigger a CI/CD pipeline that runs Zola and uploads the generated
files to a WWT Azure Storage account that then serves the files as a static
website. For most guides this is the `wwtwebstatic` account backing
[docs.worldwidetelescope.org][docs.wwt.o] as described above.

The details of the CI/CD pipeline for guides are also tracked in the
[zola-wwtguide] repository, in the file [azure-build-and-publish.yml]. The
Azure Pipelines configurations for the indiviual guide repositories pretty
much just provide configuration that is fed into this template. Unlike the way
that each repository tracks the theme, the way that Azure Pipelines supports
this functionality means that every build automatically uses the version of
this template on the `master` branch of the [zola-wwtguide] repository on
GitHub — the specific commit referenced in the `themes/wwtguide` submodule
does not come into play.

[azure-build-and-publish.yml]: https://github.com/WorldWideTelescope/zola-wwtguide/blob/master/azure-build-and-publish.yml
