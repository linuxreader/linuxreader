# 

# 9 {#Chapter_9.xhtml#h1_219 .chapterNumber}

# Pushing Images to a Container Registry {#Chapter_9.xhtml#h1_220 .chapterTitle}

In the previous chapter, we went through the very important concept of
the container base image.[]{.sentence-end} As we saw, it is really
important to choose the base image wisely for our containers, using
official container images from trusted container registries and
development communities.

But once we choose the preferred base image and then build our final
container image, we need a way to further distribute our work to the
various target hosts that we plan to let it run on.

The best option to distribute a container image is to push it to a
container registry, and after that, let all the target hosts pull the
container image and run it.

For this reason, in this chapter, we\'re going to cover the following
main topics:

-   What is a container registry?
-   Cloud-based and on-premises container registries
-   Managing container images with Skopeo
-   Running a local container registry

# Technical requirements {#Chapter_9.xhtml#h1_221 .heading-1}

Before proceeding with the chapter and its examples, a machine with a
working Podman installation is required.[]{.sentence-end} As stated in
*[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running the First
Container*, all the examples in the book are executed on a Fedora 40
system or later, but can be reproduced on your OS of choice.

A good understanding of the topics covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}*, *Managing Running Containers*,
and *[Chapter 8](#Chapter_8.xhtml#h1_199){.chapref}*, *Choosing* *the*
*Container Base Image*, is useful to easily grasp concepts regarding
container registries.

# What is a container registry? {#Chapter_9.xhtml#h1_222 .heading-1}

A container registry is a collection of container images\' repositories,
used in conjunction with systems that []{#Chapter_9.xhtml#idx_e9d759d5
.index-entry index-entry="container registry"}need to pull and run
container images in a dynamic way.

The main features available on a container registry are as follows:

-   Repository management
-   Pushing container images
-   Tag management
-   Pulling container images
-   Authentication management

Let\'s look at every feature in detail in the following sections.

## Repository management {#Chapter_9.xhtml#h2_223 .heading-2}

One of the most important features of container registries is managing
container images through []{#Chapter_9.xhtml#idx_180099eb .index-entry
index-entry="container registry:repository management"}repositories.[]{.sentence-end}
Depending on the container registry implementation that we choose, we
will be sure to find a web interface or a command-line
[]{#Chapter_9.xhtml#idx_b26144e4 .index-entry
index-entry="repository management"}interface that will let us handle
the creation of a sort of *folder* that will act as a repository for our
container images.

According to the **Open Container Initiative** (**OCI**) Distribution
Specification *\[1\]*, the container images are
[]{#Chapter_9.xhtml#idx_65cd1139 .index-entry
index-entry="Open Container Initiative (OCI)"}organized in a repository
that is identified by name.[]{.sentence-end} A repository name is
usually composed of a user/organization name and the container image
name in this way:
`myorganization`{.inlineCode}`/`{.inlineCode}`mycontainerimage`{.inlineCode}.[]{.sentence-end}
It must respect the following regular expression check:

``` {.programlisting .snippet-code}
[a-z0-9]+([._-][a-z0-9]+)*(/[a-z0-9]+([._-][a-z0-9]+)*)*
```

 note
**Important** **definition**

A **regular expression** (**regex**) is a search
[]{#Chapter_9.xhtml#idx_9598fae9 .index-entry
index-entry="regular expression (regex)"}pattern defined by a sequence
of characters.[]{.sentence-end} This pattern definition leverages
several notations that let the user define in detail the target keyword,
line, or multiple lines to find in a text document.


Once we\'ve created a repository on our container registry, we should be
able to start pushing, pulling, and handling different versions
(identified by a label) for our container images.

## Pushing container images {#Chapter_9.xhtml#h2_224 .heading-2}

The act of pushing container images to a container registry is handled
by the container tool that we are using, which respects the OCI
Distribution Specification.

In this []{#Chapter_9.xhtml#idx_0a58d0ea .index-entry
index-entry="container registry:container images, pushing"}process, the
blobs, which are the binary form of the content, are uploaded first,
and, usually at the end, the manifest is then uploaded.[]{.sentence-end}
This order []{#Chapter_9.xhtml#idx_d6bcf23f .index-entry
index-entry="container images:pushing"}is not strict and mandatory by
the specification, but a registry may refuse a manifest that references
blobs that it does not know.

Using a container management tool to push a container image to a
registry, we must again specify the name of the repository in the form
shown before and the container image\'s tag we want to upload.

## Tag management {#Chapter_9.xhtml#h2_225 .heading-2}

As introduced in *[Chapter 4](#Chapter_4.xhtml#h1_114){.chapref}*,
*Managing* *Running Containers*, container images are identified by a
name []{#Chapter_9.xhtml#idx_016c66c6 .index-entry
index-entry="container registry:tag management"}and a
tag.[]{.sentence-end} Thanks to the tag mechanism, we can store several
different versions of the container images on a system\'s local cache or
on a container registry.

The container []{#Chapter_9.xhtml#idx_b5c89187 .index-entry
index-entry="tag management"}registry should be able to expose the
feature of content discovery, providing the list of the container
images\' tags to the client requesting it.[]{.sentence-end} This feature
can give the opportunity to the container registry\'s users to choose
the right container image to pull and run on the target systems.

## Pulling container images {#Chapter_9.xhtml#h2_226 .heading-2}

In the process of pulling container images, the client first requests
the manifest to identify the blobs, which are
[]{#Chapter_9.xhtml#idx_393de1ac .index-entry
index-entry="container images:pulling"}the binary form of the content,
to pull to get the final []{#Chapter_9.xhtml#idx_a0268181 .index-entry
index-entry="container registry:container images, pulling"}container
image.[]{.sentence-end} The order is strict because, without pulling and
parsing the manifest file of the container image, the client would not
be able to know which binary data it has to request from the registry.

Using a container management tool to pull a container image from a
registry, we have to use the fully-qualified name of the image, which
includes both tag and repository.

## Authentication management {#Chapter_9.xhtml#h2_227 .heading-2}

All the previous []{#Chapter_9.xhtml#idx_d3402df3 .index-entry
index-entry="container registry:authentication management"}operations
may require authentication.[]{.sentence-end} In many
[]{#Chapter_9.xhtml#idx_36e25df9 .index-entry
index-entry="authentication management"}cases, public container
registries may allow anonymous pulling and content discovery, but for
pushing container images, they require a valid authentication.

Depending on the container registry chosen, we might find basic or
advanced features to authenticate to a container
registry.[]{.sentence-end} This lets our client store a token and then
use it for every operation that could require it.

This ends our brief deep dive into container registry
theory.[]{.sentence-end} If you want to know more about the OCI
Distribution Specification, you can investigate the URL *\[1\]*
available at the end of this chapter in the *Further reading* section.

 packt_tip
**Nice to** **know**

The OCI []{#Chapter_9.xhtml#idx_6029a223 .index-entry
index-entry="OCI Distribution Specification"}Distribution Specification
also defines a set of conformance tests that anyone could run against a
container registry to check whether that particular implementation
respects all the rules defined in the specification:
[[https://github.com/opencontainers/distribution-spec/tree/main/conformance]{.url}](https://github.com/opencontainers/distribution-spec/tree/main/conformance){style="text-decoration: none;"}.


The various implementations of a container registry available on the
web, in addition to the basic functions we described in earlier
chapters, offer additional features that we will discover soon in the
next section.

# Cloud-based and on-premises container registries {#Chapter_9.xhtml#h1_228 .heading-1}

As we introduced in the previous sections, OCI defined a standard to
adhere to for container registries.[]{.sentence-end} This initiative
allowed the rise of many other container registries apart from the
initial Docker Registry and its online service, Docker Hub.

We can group the available container registries into two main
categories:

-   Cloud-based container registries
-   On-premises container registries

Let\'s see these two categories in detail in the following subsections.

## On-premises container registries {#Chapter_9.xhtml#h2_229 .heading-2}

On-premises []{#Chapter_9.xhtml#idx_312d3d56 .index-entry
index-entry="on-premises container registries"}container registries are
often used for creating a private repository for enterprise
purposes.[]{.sentence-end} The main use cases include the following:

-   Distributing images in a private or isolated network
-   Deploying a new container image at a large scale over several
    machines
-   Keeping any sensitive data in our own data center
-   Improving the speed of pulling and pushing images using an internal
    network

Of course, running an on-premises registry requires several skills to
ensure availability, monitoring, logging, and security.

This is a non-comprehensive list of the available container registries
that we can install on-premises:

-   **Docker Registry**: Docker\'s[]{#Chapter_9.xhtml#idx_d83c7047
    .index-entry index-entry="Docker Registry"} project, which is
    currently at version 2, provides all the basic features described in
    the earlier sections, and we will learn how to run it in the last
    section of this chapter, *Running a local container registry*.
-   **Harbor**: This is a []{#Chapter_9.xhtml#idx_2358def5 .index-entry
    index-entry="Harbor"}VMware open source project that provides high
    availability, image auditing, and integration with authentication
    systems.
-   **GitLab Container Registry**: This is strongly integrated with the
    GitLab product, so it requires minimal
    []{#Chapter_9.xhtml#idx_911df3f8 .index-entry
    index-entry="GitLab Container Registry"}setup, but it depends on the
    main project.
-   **JFrog** **Artifactory**: This[]{#Chapter_9.xhtml#idx_8ab5a170
    .index-entry index-entry="JFrog Artifactory"} manages more than just
    containers; it provides management for any artifact.
-   **Quay**: This is the[]{#Chapter_9.xhtml#idx_558a2fa9 .index-entry
    index-entry="Quay"} open source distribution of the Red Hat product
    called **Quay**.[]{.sentence-end} This project offers a fully
    featured web UI, a service for image vulnerability scanning, data
    storage, and protection.

We will not go []{#Chapter_9.xhtml#idx_74ee72f5 .index-entry
index-entry="on-premises container registries"}into every detail of
these container registries.[]{.sentence-end} What we can suggest for
sure is to pay attention and choose the product or project that fits
well with your use cases and support needs.[]{.sentence-end} Many of
these products have support plans or enterprise editions (license
required) that could easily save your skin in the event of a disaster.

Let\'s now see what the cloud-based container registries are, which
could make our lives easier, offering a complete managed service, with
which our operational skills could be reduced to zero.

## Cloud-based container registries {#Chapter_9.xhtml#h2_230 .heading-2}

As anticipated in the previous section, cloud-based container registries
could be the fastest way to start []{#Chapter_9.xhtml#idx_7c6b751e
.index-entry index-entry="cloud-based container registries"}working with
container images through a registry.

As described in *[Chapter 8](#Chapter_8.xhtml#h1_199){.chapref}*,
*Choosing the Container Base Image*, there are several cloud-based
container registry services on the web.[]{.sentence-end} We will
concentrate only on a small subset, taking out of the analysis the ones
provided by a public cloud provider and the ones offered by the Linux
distribution, which are usually only available to pull images, preloaded
by the distribution maintainers.

Let\'s take a look at these cloud container registries:

-   **Docker Hub**: This is a[]{#Chapter_9.xhtml#idx_47f47d0d
    .index-entry index-entry="Docker Hub"} hosted registry solution by
    Docker Inc.[]{.sentence-end} This registry also hosts official
    repositories and security-verified images for some popular open
    source projects.
-   **Quay.io**: This is the[]{#Chapter_9.xhtml#idx_f2753ae0
    .index-entry index-entry="Quay.io"} hosted registry solution born
    under the CoreOS company, now part of Red Hat.[]{.sentence-end} It
    offers private and public repositories, automated scanning for
    security purposes, image builds, and integration with popular Git
    public repositories.

### Docker Hub cloud registry {#Chapter_9.xhtml#h3_231 .heading-3}

Docker Hub cloud registry was born together with the Docker project, and
it represented one of the []{#Chapter_9.xhtml#idx_06d177e1 .index-entry
index-entry="cloud-based container registries:Docker Hub"}key features
added to this project, giving containers in general the right attention
they deserved.

Talking []{#Chapter_9.xhtml#idx_7f1ab550 .index-entry
index-entry="Docker Hub"}about features, Docker Hub has free and paid
plans:

-   **Anonymous access**: Only 10 image pulls per IP address per hour
-   **A registered user account with the free tier**: 40 image pulls per
    hour
-   **Pro, Team, and Business accounts**: Thousands of image pulls per
    day, automated builds, support, and so on

As we just reported, if we try to log in with a registered user account
on the free tier, we can only create public repositories and a private
repository.[]{.sentence-end} This could be enough for communities or
individual developers, but once you start using it at the enterprise
level, you may need the additional features provided by the paid plans.

To avoid a significant limitation in terms of image pulls, we should at
least use a registered user account and log in to both the web portal
and the container registry using our beloved container engine,
Podman.[]{.sentence-end} We will see in the following sections how to
authenticate to a registry and interact with it for pushing and pulling
images.

### Red Hat Quay.io cloud registry {#Chapter_9.xhtml#h3_232 .heading-3}

The Quay.io []{#Chapter_9.xhtml#idx_c182a827 .index-entry
index-entry="Software-as-a-Service (SaaS)"}cloud registry is the Red Hat
on-premises registry, but offered as **Software-as-a-Service**
(**SaaS**).

Quay.io []{#Chapter_9.xhtml#idx_9a2e7893 .index-entry
index-entry="Quay.io"}cloud registry, like Docker Hub, also offers paid
plans to unlock additional features.

But the []{#Chapter_9.xhtml#idx_ce1f1506 .index-entry
index-entry="cloud-based container registries:Quay.io"}good news is that
Quay\'s free tier has a lot of features included:

-   Build from a Dockerfile, manually uploaded, or even linked through
    GitHub/Bitbucket/Gitlab or any Git repository
-   Security scans for images pushed to the registry
-   Usage/auditing logs
-   Robot user accounts/tokens for integrating any external software
-   Five private repositories
-   There is no limit on image pulls

On the other hand, the paid plans will unlock more private repositories
and team-based permissions.

Let\'s look at the Quay.io cloud registry by creating a public
repository and linking it to a GitHub repository in which we pushed a
Dockerfile to build our target container image:

1.  First, we need []{#Chapter_9.xhtml#idx_4e4a740b .index-entry
    index-entry="Quay.io:reference link"}to register or log in to the
    Quay.io portal at
    [[https://quay.io]{.url}](https://quay.io){style="text-decoration: none;"}.[]{.sentence-end}

    After that, we []{#Chapter_9.xhtml#idx_f7f69fc0 .index-entry
    index-entry="Quay.io"}can click on the **+ Create New Repository**
    button []{#Chapter_9.xhtml#idx_76a579f9 .index-entry
    index-entry="cloud-based container registries:Quay.io"}in the
    upper-right corner:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_1.png"
    style="width:453.11249999999995px; height:365.41330645161287px;"
    alt="Image 1" />
    </figure>

    Figure 9.1 -- Quay Create New Repository button

2.  Once done, the web portal will request some basic information about
    the new repository we want to create:

    -   A name
    -   A description
    -   Public or private (we are using a free account, so public is
        fine)
    -   How to initialize the repository:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_2.png"
    style="width:460.79999999999995px; height:336.64962406015036px;"
    alt="Image 2" />
    </figure>

    Figure 9.2 -- The Create New Repository page

    We just []{#Chapter_9.xhtml#idx_7ee16040 .index-entry
    index-entry="cloud-based container registries:Quay.io"}defined a
    name for []{#Chapter_9.xhtml#idx_14b0c758 .index-entry
    index-entry="Quay.io"}our repo, `ubi8-httpd`{.inlineCode}, and we
    chose to link this repository to a GitHub repository push.

3.  Once confirmed, the Quay.io registry cloud portal will redirect us
    to GitHub to allow the authorization, and then it will ask us to
    select the right organization and GitHub repository to link with:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_3.png"
    style="width:460.79999999999995px; height:286.9466794995188px;"
    alt="Image 3" />
    </figure>

    Figure 9.3 -- Selecting the GitHub repository to link with our
    container repo

    We just []{#Chapter_9.xhtml#idx_33d2b0bd .index-entry
    index-entry="Quay.io"}selected the default organization and the Git
    repository []{#Chapter_9.xhtml#idx_e3df1273 .index-entry
    index-entry="cloud-based container registries:Quay.io"}we created,
    holding our Dockerfile.[]{.sentence-end} The Git repository is named
    `ubi8-httpd`{.inlineCode}, and it is available here:
    [[https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/Chapter09/ubi8-httpd]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps-Second-Edition/tree/main/Chapter09/ubi8-httpd){style="text-decoration: none;"}

    ::: packt_tip-one
    **Important** **note**

    The repository used in this example belongs to the author\'s own
    project.[]{.sentence-end} You can do the same by forking the
    repository on GitHub and creating your own copy with read/write
    permissions in order to be able to make changes and experiment with
    commits and automated builds.
    :::

4.  Finally, it will ask us to further configure the trigger:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_4.png"
    style="width:460.79999999999995px; height:162.7572553430821px;"
    alt="Image 4" />
    </figure>

    Figure 9.4 -- Build trigger customization

    We just []{#Chapter_9.xhtml#idx_acc70bfd .index-entry
    index-entry="cloud-based container registries:Quay.io"}left the
    default option, which []{#Chapter_9.xhtml#idx_206a5fa2 .index-entry
    index-entry="Quay.io"}will trigger a new build every time a push is
    made on the Git repository for any branches and tags.

5.  Once done, we will be redirected to the main repository page:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_5.png"
    style="width:460.79999999999995px; height:265.617679558011px;"
    alt="Image 5" />
    </figure>

    Figure 9.5 -- Main repository page

    Once created, the repository is empty with no information or
    activity, of course.

6.  On the left bar, we can easily access the **Build**
    section.[]{.sentence-end} It\'s the fourth icon starting from the
    top.[]{.sentence-end} In the following figure, we just executed two
    pushes on our Git repository, which triggered two different builds:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_6.png"
    style="width:460.79999999999995px; height:232.49627791563273px;"
    alt="Image 6" />
    </figure>

    Figure 9.6 -- Container image build section

7.  If we try []{#Chapter_9.xhtml#idx_342611cf .index-entry
    index-entry="Quay.io"}clicking on one of the builds, the cloud
    registry []{#Chapter_9.xhtml#idx_e7669976 .index-entry
    index-entry="cloud-based container registries:Quay.io"}will show the
    details of the build:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_7.png"
    style="width:460.79999999999995px; height:243.7807622504537px;"
    alt="Image 7" />
    </figure>

    Figure 9.7 -- Container image build details

    As we can see, the build worked as expected, connecting to the
    GitHub repository, downloading the Dockerfile, executing the build,
    and finally, pushing the image to the container registry, all in an
    automated way.[]{.sentence-end} The Dockerfile contains just a few
    commands for installing an `httpd`{.inlineCode} server on a UBI8
    base image, as we learned in *[Chapter
    8](#Chapter_8.xhtml#h1_199){.chapref}*, *Choosing* *the* *Container
    Base Image*.

8.  Finally, the latest section that is worth mentioning is the included
    security scanning functionality.[]{.sentence-end} This feature is
    accessible by clicking the **Tag** icon, the second from the top in
    the left panel:

    <figure class="mediaobject-one">
    <img src="images/B31467_9_8.png"
    style="width:460.79999999999995px; height:234.04496253122397px;"
    alt="Image 8" />
    </figure>

    Figure 9.8 -- Container image tags page

As you will notice, there is a **SECURITY SCAN** column (the third)
reporting the status of the scan executed on that particular container
image associated with the tag name reported in the first
column.[]{.sentence-end} By clicking on the value of that column (in the
previous screenshot, it is **Passed**), we can obtain further details.

We just got []{#Chapter_9.xhtml#idx_849b3955 .index-entry
index-entry="cloud-based container registries:Quay.io"}some experience
leveraging []{#Chapter_9.xhtml#idx_40208ed0 .index-entry
index-entry="Quay.io"}a container registry offered as a managed
service.[]{.sentence-end} This could make our lives easier, reducing our
operational skills, but they are not always the best option for our
projects or companies.

In the next section, we will explore in detail how to manage container
images with Podman\'s companion Skopeo, and then we\'ll learn how to
configure and run a container registry on-premises.

# Managing container images with Skopeo {#Chapter_9.xhtml#h1_233 .heading-1}

So far, we have learned about many container registry concepts,
including the differences between []{#Chapter_9.xhtml#idx_19d6135d
.index-entry
index-entry="container images:managing, with Skopeo"}private and public
registries, their compliance []{#Chapter_9.xhtml#idx_decaa6b0
.index-entry index-entry="Skopeo:container images, managing"}with OCI
image specifications, and how to consume images with Podman and Buildah
to build and run containers.

However, sometimes we need to implement simple image manipulation tasks
such as moving an image from a registry to a mirror, inspecting a remote
image without the need to pull it locally, or even signing images.

The community []{#Chapter_9.xhtml#idx_098ab216 .index-entry
index-entry="Skopeo:reference link"}that gave birth to Podman and
Buildah developed a third amazing tool, **Skopeo**
([[https://github.com/containers/skopeo]{.url}](https://github.com/containers/skopeo){style="text-decoration: none;"}),
which exactly implements the features described previously.

Skopeo was designed as an image and registry manipulation tool for
DevOps teams and is not intended to run containers (the main role of
Podman) nor build OCI images (the main role of
Buildah).[]{.sentence-end} Instead, it offers a minimal and
straightforward command-line interface with basic image manipulation
commands that will prove to be extremely useful in different contexts.

Let\'s inspect the most interesting features in the next subsections.

## Installing Skopeo {#Chapter_9.xhtml#h2_234 .heading-2}

Skopeo is a Go binary tool that is already packaged and available for
many distributions.[]{.sentence-end} It can also be built
[]{#Chapter_9.xhtml#idx_1edecbb3 .index-entry
index-entry="Skopeo:installing"}and installed from source directly.

This section provides a non-exhaustive list of installation examples on
the major distributions.[]{.sentence-end} For the sake
[]{#Chapter_9.xhtml#idx_f14ab1f8 .index-entry
index-entry="CentOS Stream 8/9:Skopeo, installing on"}of clarity, it is
important to reiterate that the book lab environments were all based on
Fedora 40:

-   **Fedora,** **RHEL 8/9, CentOS 8,** **and CentOS Stream 8/9**: To
    []{#Chapter_9.xhtml#idx_eda9ec0a .index-entry
    index-entry="Skopeo:installing, on Fedora"}install
    []{#Chapter_9.xhtml#idx_2a3fe371 .index-entry
    index-entry="Fedora:Skopeo, installing on"}Skopeo
    []{#Chapter_9.xhtml#idx_28dc1c52 .index-entry
    index-entry="Skopeo:installing, on RHEL 8/9"}on
    []{#Chapter_9.xhtml#idx_4a75dcca .index-entry
    index-entry="RHEL 8/9:Skopeo, installing on"}RHEL-like
    []{#Chapter_9.xhtml#idx_fc352d18 .index-entry
    index-entry="Skopeo:installing, on CentOS 8"}systems, run
    []{#Chapter_9.xhtml#idx_f5d2f4d8 .index-entry
    index-entry="Skopeo:installing, on CentOS Stream 8/9"}the
    []{#Chapter_9.xhtml#idx_a2fbe4e0 .index-entry
    index-entry="CentOS 8:Skopeo, installing on"}following
    `dnf`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ sudo dnf -y install skopeo
    ```

-   **Debian**: To install []{#Chapter_9.xhtml#idx_e33bcd0a .index-entry
    index-entry="Skopeo:installing, on Debian"}Skopeo on Debian
    Bullseye, Testing, and []{#Chapter_9.xhtml#idx_448cbac8 .index-entry
    index-entry="Debian:Skopeo, installing on"}Unstable (Sid), run the
    following `apt-get`{.inlineCode} commands:

    ``` {.programlisting .snippet-con-one}
    $ sudo apt-get update
    $ sudo apt-get -y install skopeo
    ```

-   **Ubuntu**: To install []{#Chapter_9.xhtml#idx_b71920dc .index-entry
    index-entry="Skopeo:installing, on Ubuntu"}Skopeo on Ubuntu 20.10
    and []{#Chapter_9.xhtml#idx_a2e38599 .index-entry
    index-entry="Ubuntu:Skopeo, installing on"}newer, run the following
    command:

    ``` {.programlisting .snippet-con-one}
    $ sudo apt-get -y update
    $ sudo apt-get -y install skopeo
    ```

-   **Arch Linux**: To install []{#Chapter_9.xhtml#idx_7f90573f
    .index-entry index-entry="Skopeo:installing, on Arch Linux"}Skopeo
    on Arch Linux, run []{#Chapter_9.xhtml#idx_3c00de41 .index-entry
    index-entry="Arch Linux:Skopeo, installing on"}the following
    `pacman`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ sudo pacman -S skopeo
    ```

-   **openSUSE**: To install []{#Chapter_9.xhtml#idx_92493e65
    .index-entry index-entry="Skopeo:installing, on openSUSE"}Skopeo on
    openSUSE, run []{#Chapter_9.xhtml#idx_469ab05a .index-entry
    index-entry="openSUSE:Skopeo, installing on"}the following
    `zypper`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ sudo zypper install skopeo
    ```

-   **macOS**: To install []{#Chapter_9.xhtml#idx_2b6a52ce .index-entry
    index-entry="macOS:Skopeo, installing on"}Skopeo on
    []{#Chapter_9.xhtml#idx_0d6a8253 .index-entry
    index-entry="Skopeo:installing, on macOS"}macOS, run the following
    `brew`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ brew install skopeo
    ```

-   **Building from source**: Skopeo can also be built from
    source.[]{.sentence-end} As for Buildah, for the purposes
    []{#Chapter_9.xhtml#idx_fc181ea5 .index-entry
    index-entry="Skopeo:installing, on built-in source"}of this book, we
    will keep the focus on simple deployment methods, but if you\'re
    curious, you can find a dedicated *install* section in the main
    project repository that illustrates how to build Skopeo from source:
    [[https://github.com/containers/skopeo/blob/main/install.md#building-from-source]{.url}](https://github.com/containers/skopeo/blob/main/install.md#building-from-source){style="text-decoration: none;"}.[]{.sentence-end}

    The preceding link shows examples of containerized and
    non-containerized builds.

-   **Running Skopeo in a container**: Skopeo is also released as a
    container image that can be executed with Podman.[]{.sentence-end}
    To pull and run the latest version of Skopeo as a container, use the
    following `podman`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ podman run quay.io/skopeo/stable:latest <command> <options>
    ```

-   **Windows**: At the time of writing this book, there is no build
    available for Microsoft Windows.[]{.sentence-end} However, you
    []{#Chapter_9.xhtml#idx_323adbbe .index-entry
    index-entry="Windows Linux Subsystem (WSL)"}can install it in
    **Windows Linux Subsystem** (**WSL**).[]{.sentence-end} Since WSL is
    essentially a Linux environment, the installation process for Skopeo
    depends entirely on which distribution you have installed (Fedora,
    Ubuntu, and so on).

Skopeo uses the same system and local configuration files described for
Podman and Buildah; therefore, we can immediately focus on the
installation verification and the analysis of the most common use cases.

## Verifying the installation {#Chapter_9.xhtml#h2_235 .heading-2}

To verify []{#Chapter_9.xhtml#idx_b02a7d04 .index-entry
index-entry="Skopeo:installation, verifying"}the correct installation,
simply run the `skopeo`{.inlineCode} command with the `-h`{.inlineCode}
or `--help`{.inlineCode} option to view all available commands, as in
the following example:

``` {.programlisting .snippet-con}
$ skopeo -h
```

The expected output will show, among the utility options, all the
available commands, each one with a description of the command
scope.[]{.sentence-end} The full list of commands is as follows:

-   `copy`{.inlineCode}: Copy an image across locations, using different
    transports, such as the Docker Registry, local directories, OCI,
    tarballs, OSTree, and OCI archives
-   `delete`{.inlineCode}: Delete an image from a target location
-   `generate-sigstore-key`{.inlineCode}: Generate a
    `sigstore`{.inlineCode} public/private key pair
-   `help`{.inlineCode}: Print `help`{.inlineCode} commands
-   `inspect`{.inlineCode}: Inspect the metadata, tags, and
    configuration of an image in a target location
-   `list-tags`{.inlineCode}: Show []{#Chapter_9.xhtml#idx_878ab6cd
    .index-entry index-entry="Skopeo:installation, verifying"}the
    available tags for a specific image repository
-   `login`{.inlineCode}: Authenticate to a remote registry
-   `logout`{.inlineCode}: Log out from a remote registry
-   `manifest-digest`{.inlineCode}: Produce a manifest digest for a file
-   `standalone-sign`{.inlineCode}: A debugging tool to publish and sign
    an image using local files
-   `standalone-verify`{.inlineCode}: Verify an image signature using
    local files
-   `sync`{.inlineCode}: Synchronize one or more images across locations

Let\'s now inspect in greater detail some of the most interesting Skopeo
commands.

## Copying images across locations {#Chapter_9.xhtml#h2_236 .heading-2}

Podman, just like Docker, can be used not only to run containers but
also to pull images locally and push []{#Chapter_9.xhtml#idx_02cb2264
.index-entry index-entry="Skopeo:images, copying"}them to other
locations.[]{.sentence-end} However, one of the main caveats is the need
to run two commands, one to pull and one to push, while the local image
store remains filled with the pulled images.[]{.sentence-end} Therefore,
users should periodically clean up the local store.

Skopeo offers a smarter and simpler way to achieve this goal with the
`skopeo`{.inlineCode}` copy`{.inlineCode} command.[]{.sentence-end} The
command implements the following syntax:

``` {.programlisting .snippet-con}
skopeo copy [command options] SOURCE-IMAGE DESTINATION-IMAGE
```

In this generic description, `SOURCE-IMAGE`{.inlineCode} and
`DESTINATION-IMAGE`{.inlineCode} are images belonging to local or remote
locations and reachable using one of the following transports:

-   `docker://docker-reference`{.inlineCode}: This transport is related
    to images stored in registries implementing the *Docker Registry
    HTTP API V2*.[]{.sentence-end}

    This setting uses the
    `/`{.inlineCode}`etc`{.inlineCode}`/containers/`{.inlineCode}`registries.conf`{.inlineCode}
    or
    `$HOME/.`{.inlineCode}`config`{.inlineCode}`/containers/`{.inlineCode}`registries.conf`{.inlineCode}
    file to obtain further registry configurations.

    The `docker-reference`{.inlineCode} field follows the format
    `name[`{.inlineCode}`:tag|@digest]`{.inlineCode}.

-   `containers-storage`{.inlineCode}`:[[storage-specifier]]{image-id|docker-reference[@image-id]}`{.inlineCode}:
    This setting refers to an image in local container
    storage.[]{.sentence-end}

    The `storage-specifier`{.inlineCode} field is in the format
    `[[driver@`{.inlineCode}`]root`{.inlineCode}`[+run-root][:options]]`{.inlineCode}.

-   `dir:path`{.inlineCode}: This setting refers to an existing local
    directory that holds manifests, layers (in tarball format), and
    signatures

-   `docker-archive:path`{.inlineCode}`[:{`{.inlineCode}`docker`{.inlineCode}`-reference|@source-index}]`{.inlineCode}:
    This setting refers to a []{#Chapter_9.xhtml#idx_2ca58192
    .index-entry index-entry="Skopeo:images, copying"}Docker archive
    obtained with the `docker`{.inlineCode}` save`{.inlineCode} or
    `podman`{.inlineCode}` save`{.inlineCode} command

-   `docker-daemon:docker-reference|algo:digest`{.inlineCode}: This
    setting refers to image storage in the Docker daemon\'s internal
    storage

-   `oci:path`{.inlineCode}`[:tag]`{.inlineCode}: This setting refers to
    an image stored in a local path compliant with the OCI layout
    specifications

-   `oci-archive:path`{.inlineCode}`[:tag]`{.inlineCode}: This setting
    refers to an OCI layout specification-compliant image stored in
    tarball format

Let\'s inspect some usage examples of the
`skopeo`{.inlineCode}` copy`{.inlineCode} command in real-world
scenarios.[]{.sentence-end} The first example shows how to copy an image
from a remote registry to another remote registry:

``` {.programlisting .snippet-con}
$ skopeo copy \
   docker://docker.io/library/nginx:latest \
   docker://private-registry.example.com/lab/nginx:latest
```

The preceding example does not take care of registry authentication,
which is usually a requirement to push images to the remote
repository.[]{.sentence-end} In the next example, we show a variant
where both source and target registry are decorated with authentication
options:

``` {.programlisting .snippet-con}
$ skopeo copy \
   --src-creds USERNAME:PASSWORD \
   --dest-creds USERNAME:PASSWORD \
   docker://registry1.example.com/mirror/nginx:latest \
   docker://registry2.example.com/lab/nginx:latest
```

The previous approach, despite working perfectly, has the limitation of
passing username and password strings as clear-text
strings.[]{.sentence-end} To avoid this, we can use the
`skopeo`{.inlineCode}` login`{.inlineCode} command to authenticate to
our registries before running `skopeo`{.inlineCode}` copy`{.inlineCode}.

The third example shows a pre-authentication to the destination
registry, assuming that the source registry is publicly accessible for
pulls:

``` {.programlisting .snippet-con}
$ skopeo login private-registry.example.com
$ skopeo copy \
   docker://docker.io/library/nginx:latest \
   docker://private-registry.example.com/lab/nginx:latest
```

When we log []{#Chapter_9.xhtml#idx_7b4e1eff .index-entry
index-entry="Skopeo:images, copying"}in to the source/target registries,
the system persists the registry-provided auth tokens in dedicated auth
files that we can reuse later for further access.

By default, Skopeo looks at the
`${`{.inlineCode}`XDG_RUNTIME_DIR`{.inlineCode}`}/containers/`{.inlineCode}`auth.json`{.inlineCode}
path, but we can provide a custom location for the auth
file.[]{.sentence-end} For example, if we used the Docker container
runtime before, we could find it in the
`${HOME}/.`{.inlineCode}`docker`{.inlineCode}`/`{.inlineCode}`config.json`{.inlineCode}
path.[]{.sentence-end} This file contains a simple JSON object that
holds, for every used registry, the token obtained upon
authentication.[]{.sentence-end} The client (Podman, Skopeo, or Buildah)
will use this token to directly access the registry.[]{.sentence-end}
Basically, once one tool logs in, all tools can use the
token.[]{.sentence-end} `podman`{.inlineCode}` login`{.inlineCode}
allows `skopeo`{.inlineCode}` copy`{.inlineCode} and
`buildah`{.inlineCode}` push`{.inlineCode} to access the registry, for
example.

The following example shows the usage of the auth file, provided with a
custom path:

``` {.programlisting .snippet-con}
$ skopeo copy \
   --authfile ${HOME}/.docker/config.json \
   docker://docker.io/library/nginx:latest \
   docker://private-registry.example.com/lab/nginx:latest
```

Another common issue that can be encountered when working with a private
registry is the lack of certificates []{#Chapter_9.xhtml#idx_17b0d8c1
.index-entry index-entry="certification authority (CA)"}signed by a
known **certification authority** (**CA**) or the lack of HTTPS
communication (which means that all traffic is completely
unencrypted).[]{.sentence-end} If we consider these totally non-secure
scenarios safe to trust in a lab environment, we can skip the TLS
verification with the
`--`{.inlineCode}`dest`{.inlineCode}`-`{.inlineCode}`tls`{.inlineCode}`-verify`{.inlineCode}
and
`--`{.inlineCode}`src`{.inlineCode}`-`{.inlineCode}`tls`{.inlineCode}`-verify`{.inlineCode}
options, which accept a simple Boolean value.

The following example shows how to skip the TLS verification on the
target registry:

``` {.programlisting .snippet-con}
$ skopeo copy \
   --authfile ${HOME}/.docker/config.json \
   --dest-tls-verify false \
   docker://docker.io/library/nginx:latest \
   docker://private-registry.example.com/lab/nginx:latest
```

So far, we\'ve seen how to move images across public and private
registries, but we can use Skopeo to move images to and from local
stores easily.[]{.sentence-end} For example, we can use Skopeo as a
highly specialized push/pull tool for images inside our build pipelines.

The next example shows how to push a locally built image to a public
registry.[]{.sentence-end} The image already exists locally and is then
pushed to the remote registry:

``` {.programlisting .snippet-con}
$ podman images                                                           REPOSITORY                TAG         IMAGE ID      CREATED       SIZE
<namespace>/python_httpd  latest      4067fa24786a  12 days ago   318 MB
$ skopeo copy \
   --authfile ${HOME}/.docker/config.json \
   containers-storage:quay.io/<namespace>/python_httpd \
   docker://quay.io/<namespace>/python_httpd:latest
```

This is an amazing []{#Chapter_9.xhtml#idx_1a486c93 .index-entry
index-entry="Skopeo:images, copying"}way to manage an image push with
total control over the push/pull process and shows how the three tools
-- Podman, Buildah, and Skopeo -- can fulfill specialized tasks in our
DevOps environment, each one accomplishing the purpose it was designed
for at its best.

Let\'s see another example, this time showing how to pull an image from
a remote registry to an OCI-compliant local store:

``` {.programlisting .snippet-con}
$ skopeo copy \
   --authfile ${HOME}/.docker/config.json \
   docker://docker.io/library/nginx:latest \
   oci:/tmp/nginx
```

The output folder is compliant with the OCI image specifications and
will have the following structure (blob hashes cut for layout
reasons).[]{.sentence-end} This basically expands the image tarball at a
specified location, allowing for easy inspection:

``` {.programlisting .snippet-con}
$ tree /tmp/nginx
/tmp/nginx/
├─ blobs
│ └─sha256
│   ├──21e0df283cd68384e5e8dff7e6be1774c86ea3110c1b1e932[...]
│   ├──44be98c0fab60b6cef9887dbad59e69139cab789304964a19[...]
│   ├──77700c52c9695053293be96f9cbcf42c91c5e097daa382933[...]
│   ├──81d15e9a49818539edb3116c72fbad1df1241088116a7363a[...]
│   ├──881ff011f1c9c14982afc6e95ae70c25e38809843bb7d42ab[...]
│   ├──d86da3a6c06fb46bc76d6dc7b591e87a73cb456c990d814fd[...]
│   ├──e5ae68f740265288a4888db98d2999a638fdcb6d725f42767[...]
│   └──ed835de16acd8f5821cf3f3ef77a66922510ee6349730d89a[...]
├─ index.json
└─ oci-layout
```

The files inside the `blobs/sha256`{.inlineCode} folder include the
image manifest (in JSON format) and the image layers in compressed
tarball format

It\'s interesting to know that Podman can seamlessly run a container
based on a local folder compliant with []{#Chapter_9.xhtml#idx_32f2e5a1
.index-entry index-entry="Skopeo:images, copying"}the OCI image
specifications.[]{.sentence-end} The next example shows how to run an
NGINX container from the previously downloaded image:

``` {.programlisting .snippet-con}
$ podman run -d oci:/tmp/nginx
Getting image source signatures
Copying blob e5ae68f74026 done
Copying blob 21e0df283cd6 done
Copying blob ed835de16acd done
Copying blob 881ff011f1c9 done
Copying blob 77700c52c969 done
Copying blob 44be98c0fab6 done
Copying config 81d15e9a49 done
Writing manifest to image destination
Storing signatures
90493fe89f024cfffda3f626acb5ba8735cadd827be6c26fa44971108e09b54f
```

Notice the `oci`{.inlineCode}`:`{.inlineCode} prefix before the image
path, necessary to specify that the path provided is OCI compliant.

Besides, it is interesting to show that Podman copies and extracts the
blobs inside its local store (under
`$HOME/.local/share/containers/storage`{.inlineCode} for a rootless
container like the one in the previous example).

After learning how to copy images with Skopeo, let\'s see how to inspect
remote images without the need to pull them locally.

## Inspecting remote images {#Chapter_9.xhtml#h2_237 .heading-2}

Sometimes we need to verify the configurations, tags, or metadata of an
image before pulling and executing []{#Chapter_9.xhtml#idx_e32b9ffc
.index-entry index-entry="Skopeo:remote images, inspecting"}it
locally.[]{.sentence-end} For this purpose, Skopeo offers the useful
`skopeo`{.inlineCode}` inspect`{.inlineCode} command to inspect images
over supported transports.

The first example shows how to inspect the official NGINX image
repository:

``` {.programlisting .snippet-con}
$ skopeo inspect docker://docker.io/library/nginx
```

The `skopeo`{.inlineCode}` `{.inlineCode}`inspect`{.inlineCode} command
creates a JSON-formatted output with the following fields:

-   `Name`{.inlineCode}: The name of the image repository.
-   `Digest`{.inlineCode}: The SHA256 calculated digest.
-   `RepoTags`{.inlineCode}: The full list of available image tags in
    the repository.[]{.sentence-end} This list will be empty when
    inspecting local transports, such as
    `containers-storage:`{.inlineCode} or
    `oci`{.inlineCode}`:`{.inlineCode}, since they will be referred to
    as a single image.
-   `Created`{.inlineCode}: The creation date of the repository or
    image.
-   `DockerVersion`{.inlineCode}: The version of Docker used to create
    the image.[]{.sentence-end} This value is empty for images created
    with Podman, Buildah, or other tools.
-   `Labels`{.inlineCode}: Additional labels applied to the image at
    build time.
-   `Architecture`{.inlineCode}: The target system architecture for
    which the image was built.[]{.sentence-end} This value is
    `amd64`{.inlineCode} for x86-64 systems.
-   `Os`{.inlineCode}: The target operating system the image was built
    for.
-   `Layers`{.inlineCode}: The list of layers that compose the image,
    along with their SHA256 digest.
-   `Env`{.inlineCode}: Additional environment variables defined in the
    image at build time.

The same []{#Chapter_9.xhtml#idx_abfd55d7 .index-entry
index-entry="Skopeo:remote images, inspecting"}considerations
illustrated previously about authentication and TLS verification apply
to the `skopeo`{.inlineCode}` inspect`{.inlineCode} command: it is
possible to inspect images on a private registry upon authentication and
skip the TLS verification.[]{.sentence-end} The next example shows this
use case:

``` {.programlisting .snippet-con}
$ skopeo inspect \
   --authfile ${HOME}/.docker/config.json \
   --tls-verify false \
   registry.example.com/library/test-image
```

Inspecting local images is possible by passing the correct
transport.[]{.sentence-end} The next example shows how to inspect a
local OCI image:

``` {.programlisting .snippet-con}
$ skopeo inspect oci:/tmp/custom_image
```

The output of this command will have an empty `RepoTags`{.inlineCode}
field.

In addition, it is possible to use the `--no-tags`{.inlineCode} option
to intentionally skip the repository tags, like in the following
example:

``` {.programlisting .snippet-con}
$ skopeo inspect --no-tags docker://docker.io/library/nginx
```

On the other hand, if we need to print only the available repository
tags, we can use the `skopeo`{.inlineCode}` list-tags`{.inlineCode}
command.[]{.sentence-end} The next example prints all the available tags
of the official NGINX repository:

``` {.programlisting .snippet-con}
$ skopeo list-tags docker://docker.io/library/nginx
```

The third use case we are going to analyze is the synchronization of
images across registries and local stores.

## Synchronizing registries and local directories {#Chapter_9.xhtml#h2_238 .heading-2}

When []{#Chapter_9.xhtml#idx_74c16694 .index-entry
index-entry="Skopeo:registries and local directories, synchronizing"}working
with disconnected environments, a quite common scenario is the need to
synchronize repositories from a remote registry locally.

To serve this purpose, Skopeo introduced the
`skopeo`{.inlineCode}` sync`{.inlineCode} command, which helps
synchronize content between a source and destination, supporting
different transport kinds.

We can use this command to synchronize a whole repository, with all the
available tags inside it, between a source and a
destination.[]{.sentence-end} Alternatively, it is possible to
synchronize only a specific image tag.

The first example shows how to synchronize the official
`busybox`{.inlineCode} repository from a private registry to the local
filesystem.[]{.sentence-end} This command pulls all the tags contained
in the remote repository to the local destination (the target directory
must already exist):

``` {.programlisting .snippet-con}
$ mkdir /tmp/images
$ skopeo sync \
  --src docker --dest dir \
  registry.example.com/lab/busybox /tmp/images
```

Notice the use of the `--`{.inlineCode}`src`{.inlineCode} and
`--`{.inlineCode}`dest`{.inlineCode} options to define the kind of
transport.[]{.sentence-end} Supported transport types are as follows:

-   **Source**: `docker`{.inlineCode}, `dir`{.inlineCode}, and
    `yaml`{.inlineCode} (covered later in this section)
-   **Destination**: `docker`{.inlineCode} and `dir`{.inlineCode}

By default, Skopeo syncs the repository content to the destination
without the whole image source path.[]{.sentence-end} This could
represent a limitation when we need to sync repositories with the same
name from multiple sources.[]{.sentence-end} To solve this limitation,
we can add the `--scoped`{.inlineCode} option and get the full image
source path copied in the destination tree.

The second example shows a scoped synchronization of the
`busybox`{.inlineCode} repository:

``` {.programlisting .snippet-con}
$ skopeo sync \
   --src docker --dest dir --scoped \
   registry.example.com/lab/busybox /tmp/images
```

The resulting path in the destination directory will contain the
registry name and the related namespace, with a new folder named after
the image tag.

The next example shows the directory structure of the destination after
a successful synchronization:

``` {.programlisting .snippet-con}
ls -A1 /tmp/images/docker.io/library/
busybox:1
busybox:1.21.0-ubuntu
busybox:1.21-ubuntu
busybox:1.23
busybox:1.23.2
busybox:1-glibc
busybox:1-musl
busybox:1-ubuntu
busybox:1-uclibc
[...omitted output...]
```

If we need to synchronize only a specific image tag, it is possible to
specify the tag name in the source argument, as in this third example:

``` {.programlisting .snippet-con}
$ skopeo sync --src docker --dest dir docker.io/library/busybox:latest /tmp/images
```

We can []{#Chapter_9.xhtml#idx_356564ff .index-entry
index-entry="Skopeo:registries and local directories, synchronizing"}directly
synchronize two registries using Docker, both for the source and
destination transport.[]{.sentence-end} This is especially useful in
disconnected environments where systems are allowed to reach a local
registry only.[]{.sentence-end} The local registry can mirror
repositories from other public or private registries, and the task can
be scheduled periodically to keep the mirror updated.

 packt_tip
**Important note**

Docker Hub imposes strict pull rate limits on unauthenticated
(anonymous) users.[]{.sentence-end} If you are following along with
these exercises and are not logged into a Docker account, you may
quickly hit these limits, causing your image pulls to
fail.[]{.sentence-end} To avoid *Too Many Requests* errors and ensure a
more reliable experience, we recommend using the images hosted on Quay,
which currently offers a more generous path for public, unauthenticated
pulls.


The next example shows how to synchronize the UBI9 image and all its
tags from the public Red Hat repository to a local mirror registry:

``` {.programlisting .snippet-con}
$ skopeo sync \
   --src docker --dest docker \
   --dest-tls-verify=false \
   registry.access.redhat.com/ubi9 \
   mirror-registry.example.com
```

The preceding command will mirror all the UBI9 image tags to the target
registry.

Notice the
`--`{.inlineCode}`dest`{.inlineCode}`-`{.inlineCode}`tls`{.inlineCode}`-verify=false`{.inlineCode}
option to disable TLS certificate checks on the destination.

The `skopeo`{.inlineCode}` sync`{.inlineCode} command is great to mirror
repositories and single images between locations, but when it comes to
mirroring full registries or a large set of repositories, we should run
the command many times, passing different source arguments.

To avoid this limitation, the source transport can be defined as a YAML
file to include an exhaustive list of registries, repositories, and
images.[]{.sentence-end} It is also possible to use regular expressions
to capture only selected subsets of image tags.

The following []{#Chapter_9.xhtml#idx_05f53da2 .index-entry
index-entry="Skopeo:registries and local directories, synchronizing"}is
an example of a custom YAML file that will be passed as a source
argument to Skopeo
(`Chapter09`{.inlineCode}`/`{.inlineCode}`example_sync.yaml`{.inlineCode}):

``` {.programlisting .snippet-code}
docker.io:
  tls-verify: true
  images:
    alpine: []
    nginx:
      - "latest"
  images-by-tag-regex:
    httpd: ^2\.4\.[0-9]*-alpine$
quay.io:
  tls-verify: true
  images:
    fedora/fedora:
      - latest
registry.access.redhat.com:
  tls-verify: true
  images:
    ubi9     :
      - "9     .4"
      - "9     .5"
```

In the preceding example, different images and repositories are defined,
and therefore, the file content deserves a detailed description.

The whole `alpine`{.inlineCode} repository is pulled from
`docker.io`{.inlineCode}, along with the
`nginx`{.inlineCode}`:latest`{.inlineCode} image tag.[]{.sentence-end}
Also, a regular expression is used to define a pattern of tags for the
`httpd`{.inlineCode} image, in order to pull Alpine-based image version
2.4.z only.

The file also defines a specific tag (`latest`{.inlineCode}) for the
`fedora`{.inlineCode} image stored under
[[https://quay.io/]{.url}](https://quay.io/){style="text-decoration: none;"}
and the `9.4`{.inlineCode} and `9.5`{.inlineCode} tags for the
`ubi9`{.inlineCode} image stored under the
`registry.access.redhat.com`{.inlineCode} registry.

Once defined, the file is passed as an argument to Skopeo, along with
the destination:

``` {.programlisting .snippet-con}
$ skopeo sync \
  --src yaml --dest dir \
  --scoped example_sync.yaml /tmp/images
```

All the contents listed in the `example_sync.yaml`{.inlineCode} file
will be copied to the destination directory, following the previously
mentioned filtering rules.

The next example shows a larger mirroring use case, applied to the
OpenShift release images.[]{.sentence-end} The following
`openshift_sync.yaml`{.inlineCode} file defines a regular expression to
sync all the images for version 4.17.z of OpenShift built for the x86_64
architecture
(`Chapter09`{.inlineCode}`/`{.inlineCode}`openshift_sync.yaml`{.inlineCode}):

``` {.programlisting .snippet-code}
quay.io:
  tls-verify: true
  images-by-tag-regex:
    openshift-release-dev/ocp-release: ^4\.17     \..*-x86_64$
```

 note
**Important note**

The **z stream** in a []{#Chapter_9.xhtml#idx_1fb548ca .index-entry
index-entry="z stream"}version number (such as `x.y.z`{.inlineCode})
that typically represents patch releases containing bug fixes and minor
improvements that do not introduce new features.[]{.sentence-end}
Increasing the `z`{.inlineCode} value indicates a stable,
backward-compatible update focused on enhancing the existing version\'s
quality.


We can use []{#Chapter_9.xhtml#idx_a08d7409 .index-entry
index-entry="Skopeo:registries and local directories, synchronizing"}this
file to mirror a whole minor release of OpenShift to an internal
registry accessible from disconnected environments and use this mirror
to successfully conduct an air-gapped installation of OpenShift
Container Platform.[]{.sentence-end} The next command example shows this
use case:

``` {.programlisting .snippet-con}
$ skopeo sync \
  --src yaml --dest docker \
  --dest-tls-verify=false \
  --src-authfile pull_secret.json \
  openshift_sync.yaml mirror-registry.example.com:5000
```

It is worth noticing the usage of a pull secret file, passed with the
`--`{.inlineCode}`src-authfile`{.inlineCode} option, to authenticate on
the Quay public registry and pull images from the
`ocp`{.inlineCode}`-release`{.inlineCode} repository.

There is a final Skopeo feature that captures our interest: the remote
deletion of images, covered in the next subsection.

## Deleting images {#Chapter_9.xhtml#h2_239 .heading-2}

A registry can []{#Chapter_9.xhtml#idx_96fa5d60 .index-entry
index-entry="Skopeo:images, deleting"}be imagined as a specialized
object store that implements a set of HTTP APIs to manipulate its
content and push/pull objects in the form of image layers and metadata.

The **Docker Registry v2** protocol is []{#Chapter_9.xhtml#idx_e8da722e
.index-entry index-entry="Docker Registry v2"}a standard API
specification that is widely adopted among many registry projects
*\[3\]*.[]{.sentence-end} This set of API specifications covers all the
registry functions that are expected to be exposed to an external client
through standard HTTP `GET`{.inlineCode}, `PUT`{.inlineCode},
`DELETE`{.inlineCode}, `POST`{.inlineCode}, and `PATCH`{.inlineCode}
methods.

This means that we could interact with a registry with any kind of HTTP
client capable of managing the requests correctly, for example, the
`curl`{.inlineCode} command.

Any container engine uses, at a lower level, HTTP client libraries to
execute the various methods against the registry (for example, for an
image pull).

The Docker v2 protocol also supports the remote deletion of images, and
any registry that implements this protocol supports the following
`DELETE`{.inlineCode} request for images:

``` {.programlisting .snippet-con}
DELETE /v2/<name>/manifests/<reference>
```

The following example represents a theoretical `DELETE`{.inlineCode}
command issued with the `curl`{.inlineCode} command against a local
registry:

``` {.programlisting .snippet-con}
$ curl -v --silent \
   -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
   -X DELETE http://127.0.0.1:5000/v2/<name>/manifests/sha256:<image_tag_digest>
```

The preceding []{#Chapter_9.xhtml#idx_555db6d3 .index-entry
index-entry="Skopeo:images, deleting"}example intentionally avoids
including the management of authorization tokens for readability.

Podman and Docker, designed to work as registry engines, do not
implement a remote `delete`{.inlineCode} feature among their command
interfaces.

Fortunately, Skopeo comes to the rescue with its built-in
`skopeo`{.inlineCode}` delete`{.inlineCode} command to manage remote
image deletion with a simple and user-friendly syntax.

The following example deletes an image on a hypothetical internal
`mirror-registry.example.com:5000`{.inlineCode} registry:

``` {.programlisting .snippet-con}
$ skopeo delete \
  docker://mirror-registry.example.com:5000/foo:bar
```

The command immediately deletes the image tag references in the remote
registry.

 note
**Important** **note**

When deleting images with Skopeo, it is necessary to enable image
deletion in the remote registry, as covered in the next section,
*Running a local container registry*.


In this section, we have learned how to use Skopeo to copy, delete,
inspect, and sync images or even whole repositories across different
transports, including private local registries, gaining control over
daily image manipulation operations.

We did not cover the container image\'s signature process here; we will
explore it in depth with practical examples in *[Chapter
10](#Chapter_10.xhtml#h1_248){.chapref}*, *Securing* *Containers*.

In the next section, we will learn how to run and configure a local
container registry to directly manage image storage in our lab or
development environments.

# Running a local container registry {#Chapter_9.xhtml#h1_240 .heading-1}

Most companies and organizations adopt enterprise-grade registries to
rely on secure and resilient solutions for their container image
storage.[]{.sentence-end} Most enterprise registries also offer advanced
features []{#Chapter_9.xhtml#idx_92af3c05 .index-entry
index-entry="role-based access control (RBAC)"}such as **role-based
access control** (**RBAC**), an image vulnerability scanner, mirroring,
geo-replication, and high availability, becoming the
[]{#Chapter_9.xhtml#idx_1a4fa5f1 .index-entry
index-entry="local container registry:running"}default choice for
production and mission-critical environments.

However, sometimes it is very useful to run a simple local registry, for
example, in development environments or training labs.[]{.sentence-end}
Local registries can also be helpful in disconnected environments to
mirror the main public or private registries.

This section aims to illustrate how to run a simple local registry and
how to apply basic configuration settings.

## Running a containerized registry {#Chapter_9.xhtml#h2_241 .heading-2}

Like every application, a local registry can be installed on the host by
its administrators.[]{.sentence-end} Alternatively, a commonly preferred
approach is to run the registry itself inside a container.

The most []{#Chapter_9.xhtml#idx_b235676b .index-entry
index-entry="local container registry:containerized registry, running"}used
containerized registry solution is []{#Chapter_9.xhtml#idx_4ecbfa2e
.index-entry index-entry="Docker Registry 2.0"}based on the official
**Docker Registry 2.0** image, which offers all the
[]{#Chapter_9.xhtml#idx_3c4ed9b0 .index-entry
index-entry="containerized registry:running"}necessary functionalities
for a basic registry and is very easy to use.

When running a local registry, containerized or not, we must define a
destination directory to host all image layers and
metadata.[]{.sentence-end} The next example shows the first execution of
a containerized registry, with the `/var/lib/registry`{.inlineCode}
volume created and mounted to hold image data:

``` {.programlisting .snippet-con}
# podman volume create registry_data
# podman run -d \
   --name local_registry \
   -p 5000:5000 \
   -v registry_data:/var/lib/registry:Z \

   --restart=always registry:2
```

The registry will be reachable at the host address on port
`5000/`{.inlineCode}`tcp`{.inlineCode}, which is also the default port
for this service.[]{.sentence-end} If we run the registry on our local
workstation, it will be reachable at `localhost:5000`{.inlineCode}, and
exposed []{#Chapter_9.xhtml#idx_8a973085 .index-entry
index-entry="Fully Qualified Domain Name (FQDN)"}to the external
connection using the assigned IP address or its **Fully Qualified Domain
Name** (**FQDN**) if the workstation/laptop is resolved by a local DNS
service.

For example, if a host has the IP address `10.10.2.30`{.inlineCode} and
FQDN `registry.example.com`{.inlineCode} correctly resolved by DNS
queries, the registry service will be reachable at
`10.10.2.30:5000`{.inlineCode} or at
`registry.example.com:5000`{.inlineCode}.

 packt_tip
**Important** **note**

If the host runs a local firewall service or is behind a corporate
firewall, do not forget to open the correct ports to expose the registry
externally.


We can try to build and push a test image to the new
registry.[]{.sentence-end} The following Containerfile builds a basic
UBI-based `httpd`{.inlineCode} server
(`Chapter09`{.inlineCode}`/`{.inlineCode}`local_registry`{.inlineCode}`/`{.inlineCode}`minimal_httpd`{.inlineCode}`/`{.inlineCode}`Containerfile`{.inlineCode}):

``` {.programlisting .snippet-code}
FROM registry.access.redhat.com/ubi8:latest
RUN dnf install -y httpd && dnf clean all -y
COPY index.html /var/www/html
RUN dnf install -y git && dnf clean all -y
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

We can build the new image with Buildah:

``` {.programlisting .snippet-con}
$ buildah build -t minimal_httpd .
```

To push []{#Chapter_9.xhtml#idx_768f2dc2 .index-entry
index-entry="local container registry:containerized registry, running"}the
image to the local registry, we can use Podman or its companion tools,
Buildah or Skopeo.[]{.sentence-end} Skopeo is very handy for these use
cases since we[]{#Chapter_9.xhtml#idx_3a027edd .index-entry
index-entry="containerized registry:running"} do not even need to scope
the image name with the registry name.

The next command shows how to push the new image to the registry:

``` {.programlisting .snippet-con}
$ skopeo copy --dest-tls-verify=false \
   containers-storage:localhost/minimal_httpd \
   docker://localhost:5000/minimal_httpd
```

Notice the use of
`--`{.inlineCode}`dest`{.inlineCode}`-`{.inlineCode}`tls`{.inlineCode}`-verify=false`{.inlineCode}:
this is necessary since the local registry doesn\'t have TLS or a
trusted certificate; it provides an HTTP transport by default.

Despite being simple to implement, the default registry configuration
has some limitations that must be addressed.[]{.sentence-end} To
illustrate one of those limitations, let\'s try to delete the
just-uploaded image:

``` {.programlisting .snippet-con}
$ skopeo delete \
  --tls-verify=false \
  docker://localhost:5000/minimal_httpd

FATA[0000] Failed to delete /v2/minimal_httpd/manifests/sha256:f8c0c374cf124e728e20045f327de30ce1f3c552b307945de9b911cbee103522: {"errors":[{"code":"UNSUPPORTED","message":"The operation is unsupported."}]}
(405 Method Not Allowed)
```

As we can see in the previous output, the registry did not allow us to
delete the image, returning an HTTP` 405`{.inlineCode} error
message.[]{.sentence-end} To alter this behavior, we need to edit the
registry configuration.

## Customizing the registry configuration {#Chapter_9.xhtml#h2_242 .heading-2}

The registry []{#Chapter_9.xhtml#idx_07194e16 .index-entry
index-entry="local container registry:registry configuration, customizing"}configuration
file
(`/`{.inlineCode}`etc`{.inlineCode}`/`{.inlineCode}`docker`{.inlineCode}`/registry/`{.inlineCode}`config.yml`{.inlineCode})
can be modified to alter its behavior.[]{.sentence-end} The default
content of this file is the following:

``` {.programlisting .snippet-code}
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

We soon realize that this is an extremely basic configuration with no
authentication, no allowed deletion of images, and no TLS
encryption.[]{.sentence-end} Our custom version will try to address
those limitations.

 note
**Important** **note**

The full documentation about the registry configuration has a wide range
of options that we\'re not mentioning []{#Chapter_9.xhtml#idx_b6edd42f
.index-entry index-entry="registry configuration:reference link"}here
since it is out of the scope of this book.[]{.sentence-end} More
configuration options can be found at this link:
[[https://docs.docker.com/registry/configuration/]{.url}](https://docs.docker.com/registry/configuration/){style="text-decoration: none;"}.


The following file contains a modified version of the
`config.yml`{.inlineCode} registry
(`Chapter09`{.inlineCode}`/`{.inlineCode}`local_registry`{.inlineCode}`/customizations/`{.inlineCode}`config.yml`{.inlineCode}):

``` {.programlisting .snippet-code}
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
  delete:
    enabled: true
auth:
  htpasswd:
    realm: basic-realm
    path: /var/lib/htpasswd
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
  tls:
    certificate: /etc/pki/certs/tls.crt
    key: /etc/pki/certs/tls.key
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

The highlighted []{#Chapter_9.xhtml#idx_ab7d03a5 .index-entry
index-entry="local container registry:registry configuration, customizing"}sections
in the previous example emphasize the following added features:

-   **Image deletion**: By default, this setting is
    disabled.[]{.sentence-end} We need to explicitly enable it.
-   **Basic authentication** **using an** `htpasswd`{.inlineCode}
    **file**: This approach is suitable for development and lab
    environments, while a token-based authentication relying on an
    external issuer would best fit in production use cases.
-   **HTTPS transport** **using self-signed certificates**: The
    `http`{.inlineCode} section configures the registry\'s HTTP server,
    including its listening address, custom headers, and TLS settings.

Before running the registry again with our custom configuration, we need
to generate an `htpasswd`{.inlineCode} file that holds at least one
valid login and the self-signed certificates for TLS
encryption.[]{.sentence-end} Let\'s start with the
`htpasswd`{.inlineCode} file -- we can generate it using the
`htpasswd`{.inlineCode} utility, as in the following example:

``` {.programlisting .snippet-con}
htpasswd -cBb ./htpasswd admin p0dman4Dev0ps#
```

The `-`{.inlineCode}`cBb`{.inlineCode} option enables batch mode (useful
to provide the password non-interactively), creates the file if it does
not exist, and enables the `bcrypt`{.inlineCode} hashing function
*\[2\]*.[]{.sentence-end} In this example, we create the user
`admin`{.inlineCode} with the password `p0dman4Dev0ps#`{.inlineCode}.

Finally, we need to create a self-signed server certificate with its
related private key, to be used for HTTPS connections.[]{.sentence-end}
As an example, a certificate []{#Chapter_9.xhtml#idx_fb7a4d3c
.index-entry index-entry="Common Name (CN)"}associated with the
`localhost`{.inlineCode} **Common Name** (**CN**) will be created.

 packt_tip
**Important** **note**

Bounding certificates to the `localhost`{.inlineCode} CN is a frequent
practice in development environments.[]{.sentence-end} However, if the
registry is meant to be exposed externally, the `CN`{.inlineCode} and
`SubjectAltName`{.inlineCode} fields should map to the host FQDN and
alternate names.


The following []{#Chapter_9.xhtml#idx_34a4ab55 .index-entry
index-entry="local container registry:registry configuration, customizing"}example
shows how to create a self-signed certificate with the
`openssl`{.inlineCode} utility:

``` {.programlisting .snippet-con}
$ mkdir certs
$ openssl req -newkey rsa:4096 -x509 -sha256 -nodes \
  -days 365 \
  -out certs/tls.crt \
  -keyout certs/tls.key \
  -subj '/CN=localhost' \
  -addext "subjectAltName=DNS:localhost"
```

The command will issue non-interactive certificate generation, without
any extra information about the certificate subject.[]{.sentence-end}
The `tls.key`{.inlineCode} private key is generated using a 4,096-bit
RSA algorithm.[]{.sentence-end} The certificate, named
`tls.crt`{.inlineCode}, is set to expire after 1 year.[]{.sentence-end}
Both the key and certificate are written inside the `certs`{.inlineCode}
directory.

To inspect the content of the generated certificate, we can run the
following command:

``` {.programlisting .snippet-con}
$ openssl x509 -in certs/tls.crt -text -noout
```

The command will produce a human-readable dump of the certificate data
and validity.

 packt_tip
**Hint**

For the purpose of this example, the self-signed certificate is
acceptable, but it should be avoided in production scenarios.

Solutions such as **Let\'s Encrypt** provide a free CA service for
everybody and can be used to reliably secure the
[]{#Chapter_9.xhtml#idx_21f4fa20 .index-entry
index-entry="Let's Encrypt:reference link"}registry or any other HTTPS
service.[]{.sentence-end} For further details, visit
[[https://letsencrypt.org/]{.url}](https://letsencrypt.org/){style="text-decoration: none;"}.


We now have all the requirements to run our custom
registry.[]{.sentence-end} Before creating the new container, make sure
the previous instance has been stopped and removed:

``` {.programlisting .snippet-con}
# podman stop local_registry && podman rm local_registry
```

The next command shows how to run the new custom registry using bind
mounts to pass the certificates folder, the `htpasswd`{.inlineCode}
file, the registry store, and, obviously, the custom config file:

``` {.programlisting .snippet-con}
# podman volume create registry_data
# podman run -d --name local_registry \
   -p 5000:5000 \
   -v $PWD/htpasswd:/var/lib/htpasswd:z \
   -v $PWD/config.yml:/etc/docker/registry/config.yml:z \
    -v registry_data:/var/lib/registry:Z \
   -v $PWD/certs:/etc/pki/certs:z \
   --restart=always \
   registry:2
```

We can []{#Chapter_9.xhtml#idx_2c4bc547 .index-entry
index-entry="local container registry:registry configuration, customizing"}now
test the login to the remote registry using the previously defined
credentials:

``` {.programlisting .snippet-con}
$ skopeo login -u admin -p p0dman4Dev0ps# --tls-verify=false localhost:5000
Login Succeeded!
```

Notice the
`--`{.inlineCode}`tls`{.inlineCode}`-verify=false`{.inlineCode} option
to skip TLS certificate validation.[]{.sentence-end} Since it is a
self-signed certificate, we need to bypass checks that would produce the
`x509: certificate signed by unknown authority`{.inlineCode} error
message.

We can try again to delete the image pushed before:

``` {.programlisting .snippet-con}
$ skopeo delete \
  --tls-verify=false \
  docker://localhost:5000/minimal_httpd
```

This time, the command will succeed since the deletion feature was
enabled in the config file.

A local registry can be used to mirror images from an external public
registry.[]{.sentence-end} In the next subsection, we will see an
example of registry mirroring using our local registry and a selected
set of repositories and images.

## Using a local registry to sync repositories {#Chapter_9.xhtml#h2_243 .heading-2}

Mirroring []{#Chapter_9.xhtml#idx_8220ec6d .index-entry
index-entry="local container registry:using, to sync repositories"}images
and repositories to a local registry can be very useful in disconnected
environments.[]{.sentence-end} This can also be very useful to keep an
`async`{.inlineCode} copy of selected images and be able to keep pulling
them during public service outages.

The next example shows simple mirroring using the
`skopeo`{.inlineCode}` sync`{.inlineCode} command with a list of images
provided by a YAML file and our local registry as the destination:

``` {.programlisting .snippet-con}
$ skopeo sync \
  --src yaml --dest docker \
  --dest-tls-verify=false \
  kube_sync.yaml localhost:5000
```

The YAML []{#Chapter_9.xhtml#idx_7ebf06a5 .index-entry
index-entry="local container registry:using, to sync repositories"}file
contains a list of the images that compose a Kubernetes control plane
for a specific release.[]{.sentence-end} Again, we take advantage of
regular expressions to customize the images to pull
(`Chapter09`{.inlineCode}`/`{.inlineCode}`kube_sync.yaml`{.inlineCode}):

``` {.programlisting .snippet-code}
k8s.gcr.io:
  tls-verify: true
  images-by-tag-regex:
    kube-apiserver: ^v1\.3     2\..*
    kube-controller-manager: ^v1\.3     2\..*
    kube-proxy: ^v1\.3     2\..*
    kube-scheduler: ^v1\.3     2\..*
    coredns/coredns: ^v1\.9     \..*
    etcd: 3\.5     .[0-9]*-[0-9]*
```

When synchronizing a remote and local registry, a lot of layers can be
mirrored in the process.[]{.sentence-end} For this reason, it is
important to monitor the storage used by the registry
(`/var/lib/registry`{.inlineCode} in our example) to avoid filling up
the filesystem.

When the filesystem is filled, deleting older and unused images with
Skopeo is not enough, and an extra garbage collection action is
necessary to free space.[]{.sentence-end} The next subsection
illustrates this process.

## Managing registry garbage collection {#Chapter_9.xhtml#h2_244 .heading-2}

When a `delete`{.inlineCode} command is issued on a container registry,
it only deletes the image manifests that reference a set of blobs (which
could be layers or further manifests), while keeping the blobs in the
filesystem.

If a blob []{#Chapter_9.xhtml#idx_00b0ae1d .index-entry
index-entry="local container registry:registry garbage collection, managing"}is
no longer referenced by any manifest, it can be eligible for garbage
collection by the registry.[]{.sentence-end} The garbage collection
process is managed with a dedicated command,
`registry garbage-collect`{.inlineCode}, issued inside the registry
container.[]{.sentence-end} This is not an automatic process and should
be executed manually or scheduled.

In the next example, we will run a simple garbage
collection.[]{.sentence-end} The `--dry-run`{.inlineCode} flag only
prints the eligible blobs that are no longer referenced by a manifest,
and thus they can be safely deleted:

``` {.programlisting .snippet-con}
# podman exec -it local_registry \
  registry garbage-collect --dry-run \
  /etc/docker/registry/config.yml
```

To delete the blobs, simply remove the `--dry-run`{.inlineCode} option:

``` {.programlisting .snippet-con}
# podman exec -it local_registry \
  registry garbage-collect /etc/docker/registry/config.yml
```

Garbage []{#Chapter_9.xhtml#idx_dad31939 .index-entry
index-entry="local container registry:registry garbage collection, managing"}collection
helps keep the registry free of unused blobs and saves storage
space.[]{.sentence-end} On the other hand, we must keep in mind that an
unreferenced blob could still be reused in the future by another
image.[]{.sentence-end} If deleted, it could be necessary to upload it
again eventually.

# Summary {#Chapter_9.xhtml#h1_245 .heading-1}

In this chapter, we explored how to interact with container registries,
which are the fundamental storage services for our
images.[]{.sentence-end} We started with a high-level description of
what a container registry is and how it works and interacts with our
container engines and tools.[]{.sentence-end} We then moved on to a more
detailed description of the differences between public, cloud-based
registries and private registries, usually executed
on-premises.[]{.sentence-end} It was especially useful to understand the
benefits and limitations of both and to help us to understand the best
approach for our needs.

To manage container images on registries, we introduced the Skopeo tool,
which is part of the Podman companion tools family, and illustrated how
it can be used to copy, sync, delete, or simply inspect images over
registries, giving users a higher degree of control over their images.

Finally, we learned how to run a local containerized registry using the
official community image of the Docker Registry v2.[]{.sentence-end}
After showing a basic usage, we went deeper into more advanced
configuration details by showing how to enable authentication, image
deletion, and HTTPS encryption.[]{.sentence-end} The local registry
proved to be useful to sync local images as well as remote
registries.[]{.sentence-end} The registry garbage collection process was
illustrated to keep things tidy inside the registry store.

With the knowledge gained in this chapter, you will be able to manage
images over registries and even local registry instances with a higher
degree of awareness of what happens under the hood.[]{.sentence-end}
Container registries are a crucial part of a successful container
adoption strategy and should be understood very well: with this
chapter\'s concepts in mind, you should also be able to understand and
design the best-fitting solutions and gain deep control over the tools
to manipulate images.

With this chapter, we have also completed the exploration of all the
basic tasks related to container management.[]{.sentence-end} We can now
move on to more advanced topics, such as container troubleshooting and
monitoring, covered in the next chapter.

# Further reading {#Chapter_9.xhtml#h1_246 .heading-1}

-   \[1\] OCI Distribution Specification:
    [[https://github.com/opencontainers/distribution-spec/blob/main/spec.md]{.url}](https://github.com/opencontainers/distribution-spec/blob/main/spec.md){style="text-decoration: none;"}
-   \[2\] Bcrypt description:
    [[https://en.wikipedia.org/wiki/Bcrypt]{.url}](https://en.wikipedia.org/wiki/Bcrypt){style="text-decoration: none;"}
-   \[3\] Docker Registry v2 API specifications:
    [[https://docs.docker.com/registry/spec/api/]{.url}](https://docs.docker.com/registry/spec/api/){style="text-decoration: none;"}

# Get This Book's PDF Version and Exclusive Extras {#Chapter_9.xhtml#h1_247 .heading-1}

Scan the QR code (or go to
[[packtpub.com/unlock]{.url}](https://packtpub.com/unlock){style="text-decoration: none;"}).[]{.sentence-end}
Search for this book by name, confirm the edition, and then follow the
steps on the page.

![Image](images/B31467_9_9.png){style="width:25%;"}

![Image](images/B31467_9_10.png){style="width:25%;"}

*Note: Keep your invoice handy.[]{.sentence-end} Purchases made directly
from Packt don't require one.*


[]{#Chapter_10.xhtml}

 {.section .chapter-first}