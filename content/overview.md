+++
title = "Overview"
weight = 100
+++

The WWT project can be hard to wrap your head around because it has so many
different uses: visualizing data for research, educating students in
classrooms, educating the broader public in planetariums and museums, spurring
delight in the beauty of our universe, and even creating art. While all of
these applications build on a common foundation, the overall system is
admittedly sprawling. To try to anchor your understanding of it all, we’ll
start by describing the more “tangible” components of WWT.


# Key Features of the WWT Experience

The original manifestation of WWT was the
[WWT Windows client][windows-client]. Originally demo’ed in 2008, the Windows
client lets you explore the sky visually in 2D or 3D modes, showing you a
scientifically accurate model of the universe that can overlay real
astronomical data from the world’s best telescopes.

[windows-client]: //worldwidetelescope.org/download#windows-client

The original WWT experience was a 2D sky exploration experience similar to
(but predating!) Google Maps, with imagery from the
[Digitized Sky Survey][dss] (DSS). The total DSS dataset weighs in at about a
terabyte, which was a lot for 2008. Therefore, the WWT Windows application was
— and still is — intimately linked with web services capable of streaming data
to client computers as needed. Many of the key technological breakthroughs of
WWT relate to this core functionality of efficiently streaming and rendering
terapixel-class imagery on the sphere.

[dss]: http://archive.stsci.edu/dss/

From the start, WWT was conceived as a product with educational applications.
Central to that vision is the concept of the **guided tour**, which uses a
scripted path through the simulated WWT universe to create an immersive
multimedia experience. The most familiar analogy is that of presentation
software like [PowerPoint]: a tour is fundamentally a sort of “document” file
that you can open and “play” in WWT. Planetarium shows would typically be
created as tours, but the format is more flexible than that: for instance, a
desktop user can pause a tour in the middle, explore the sky away from the
scripted path, and then return to the tour plan. Tours can include not only
text and image overlays as in standard presentation software, but also
additional astronomical data sets to augment the core WWT experience. To make
this possible, WWT was designed from the ground up with *extensibility* in
mind.

[PowerPoint]: https://www.microsoft.com/en-us/microsoft-365/powerpoint

Even though the original WWT client was a Windows application, the systems for
WWT’s extensibility were strongly inspired by the successes of the World Wide
Web — it’s not a coincidence that WWT [data collection files] have the
exension `wtml`. The first web-based WWT client (implemented using
[Silverlight]) was launched not long after the Windows client, and as
technology has become more and more web-centric this wisdom of this approach
has become more and more apparent. We expect that usage of WWT will become
increasingly dominated by its web manifestations, with usage of the Windows
client likely specializing to tasks that need the power of a full desktop
application, such as VR and planetarium shows. One of the major themes of
current WWT development effort is to strengthen the web-orientation of the WWT
ecosystem and take full advantage of all of the progress that has been made in
web-based software engineering since 2008.

[data collection files]: https://docs.worldwidetelescope.org/data-guide/1/data-file-formats/collections/
[Silverlight]: https://www.microsoft.com/silverlight/

Since the project was adopted by the [AAS] in 2016, there has been an
increased emphasis on WWT’s research applications. The same factors that have
been pushing an increased emphasis on the web components of WWT have been at
play in the broader research data visualization ecosystem, with particular
success enjoyed by the [Python] language inside the [Jupyter] web notebook
system. It was natural for the [pywwt] Python package to evolve to emphasize
this approach, allowing WWT viewers to be embedded in [Jupyter] notebooks and
[JupyterLab] workspaces. The work to support this effort has also helped draw
out the distinction between what is now called the [WebGL engine], the
reusable WWT rendering library in JavaScript, and the [AngularJS] WWT
[web client] application that uses that library to deliver an experience that
reproduces that of the Windows client.

[AAS]: https://aas.org/
[Python]: https://python.org/
[Jupyter]: https://jupyter.org/
[pywwt]: https://pywwt.readthedocs.io/
[JupyterLab]: https://jupyterlab.readthedocs.io/
[WebGL engine]: //worldwidetelescope.gitbook.io/webgl-engine-reference/
[AngularJS]: https://angularjs.org/
[web client]: https://github.com/WorldWideTelescope/wwt-web-client/


# Key Elements of WWT Infrastructure

Keeping all of these manifestations of WWT in mind, we can start describing
the infrastructure needed to support the ecosystem:

- As an open-source project, the source code is the foundation upon which
  everything else is built. The WWT source is version-controlled in the
  [WorldWideTelescope organization][github-org] on GitHub, and to the greatest
  extent possible the creation of derived products is automated using
  [continuous integration and deployment][ci-cd] (CI/CD) techniques, anchored
  in the [aasworldwidetelescope organization][devops-org] on Azure DevOps. An
  important part of this orientation is that the term “source code” is
  construed broadly; for instance, the
  [source code to this documentation][this-source] is managed on GitHub.
- While some software projects are essentially freestanding, the WWT software
  relies upon supporting web data services to function. These services are
  hosted on the [Microsoft Azure] cloud platform thanks to generous
  sponsorship from the [.NET Foundation][dnf].
- WWT is, in a certain sense, a media platform, even if much of its “content”
  takes the form of scientific data. Tools for producing and testing that
  content are vital to the ecosystem, as are the broader network of
  astronomical data standards like [VAMP] and the
  [Virtual Observatory (VO) protocols][vo-protocols].
- The other form of content that is essential to WWT is, of course,
  documentation such as what you’re reading right now and the user-facing WWT
  web collateral.

[github-org]: https://github.com/WorldWideTelescope/
[ci-cd]: https://www.redhat.com/en/topics/devops/what-is-ci-cd
[devops-org]: https://dev.azure.com/aasworldwidetelescope/
[this-source]: https://github.com/WorldWideTelescope/worldwide-telescope-infrastructure-guide/
[Microsoft Azure]: https://azure.microsoft.com/
[dnf]: https://dotnetfoundation.org/
[VAMP]: https://www.virtualastronomy.org/
[vo-protocols]: http://ivoa.net/

The systems underlying these elements will be described in the following
sections.
