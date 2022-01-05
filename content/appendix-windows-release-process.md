+++
title = "Appendix: Windows App Release Process"
weight = 600
+++

We generally find that automation of software release processes is truly a
life-changer. But there are still aspects of the release process for the [WWT
Windows client][wwt-windows-client] that aren't fully automated. Some of them
could definitely benefit from automation; others probably aren't worth it.

[wwt-windows-client]: https://github.com/WorldWideTelescope/wwt-windows-client/

**TODO:** it would be *really* nice to have a way to prototype dataset updates
without potentially affecting existing installs, but we don't have the
infrastructure to do that yet.

1. In a [wwt-windows-client] checkout on `master`, prepare a [Cranko] release
   request, including release notes for both app and datasets.
1. Upload any new data files to the cloud, updating the data version in
   `wwtfiles:catalogs/wwt6_login.txt`.
1. Generate new preloaded data cabinet file using a development build of the
   app:
   1. Delete `~/AppData/Local/American Astronomical Society/WorldWideTelescope/`
   1. Launch the app
      1. Turn on MPC asteroids in Solar System view
      1. Zoom all the way out to SDSS galaxies
      1. Switch to Sky mode
      1. Open Mars folder of Explore ribbon
      1. Click on Tours tab to load up its folder
      1. Switch to Earth mode, spin around
      1. Toggle 8k cloud maps in the Earth cloud layer properties
      1. Right-click Earth in Layer Manager and "Add WMS Layer"
      1. Register the built-in WMS server under the name "NASA"
      1. Open up the "View from Location" selector dialog
      1. Switch to Mars view, spin around
      1. Switch back to Earth mode to pull 8k cloud maps
      1. Open the language settings UI
   1. Exit the app
   1. Run the `MakeDataCabinetFile` tool
   1. Upload resulting file to `wwtfiles:devops/datafiles_release.cabinet`
1. Push Cranko release request to WWT `rc` branch. This triggers the main
   release processing.
1. Download the release installer from [GitHub][gh-releases] and duplicate to
   `wwtwebstatic:$web/releases/wwt/`.
1. Update the installer redirection URL:
   1. Update `temporary_redirects.map` in [wwt-nginx-core].
   2. Build local Docker image and test locally.
   3. Submit pull request, merge to `master`
   4. Verify that production service has updated with `curl -I
      https://worldwidetelescope.org/support/wwtsetup-latest.msi`.
1. Update the user website:
   1. Update either `download/_index.md` or `download/beta.md` in [wwt-user-website].
   2. Preview changes with [Zola].
   3. Submit pull request, merge to `master`.
   4. Verify that user website has updated.
1. Re-update the login file to announce the new version:
   1. Pull down [the login file][login6]
   1. Rename it to `wwt6_login.txt`
   1. Update the `ClientVersion:` line
   1. Upload to `wwtfiles:catalogs/wwt6_login.txt`
   1. Check that an older install detects the update.

[Cranko]: https://pkgw.github.io/cranko/
[gh-releases]: https://github.com/WorldWideTelescope/wwt-windows-client/releases
[wwt-nginx-core]: https://github.com/WorldWideTelescope/wwt-nginx-core/
[wwt-user-website]: https://github.com/WorldWideTelescope/wwt-user-website/
[Zola]: https://www.getzola.org/
[login6]: https://worldwidetelescope.org/wwtweb/login6.aspx
