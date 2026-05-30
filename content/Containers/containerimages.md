# 8 {#Chapter_8.xhtml#h1_199 .chapterNumber}

# Choosing the Container Base Image {#Chapter_8.xhtml#h1_200 .chapterTitle}

The fastest and easiest way to learn about and get some experience with
containers is to start working with pre-built container images, as we
saw in the previous chapters.[]{.sentence-end} After a deep dive into
container management, we discovered that sometimes, the available
service, its configuration, or even the application version is not the
one that our project requires.[]{.sentence-end} Then, we introduced
Buildah and its feature for building custom container
images.[]{.sentence-end} In this chapter, we are going to address
another important topic that is often questioned in community and
enterprise projects: the choice of[]{#Chapter_8.xhtml#idx_fb0e07ef
.index-entry index-entry="container base image"} a **container base
image**.

Choosing the right container base image is an important task in the
container journey: a container base image is the underlying operating
system layer that our system\'s service, application, or code will rely
on.[]{.sentence-end} Due to this, we should choose one that fits our
best practices concerning security and updates.

In this chapter, we\'re going to cover the following main topics:

-   The Open Container Initiative image format
-   Where do container images come from?
-   Trusted container image sources
-   Introducing Universal Base Image

# Technical requirements {#Chapter_8.xhtml#h1_201 .heading-1}

To complete this chapter, you will need a machine with a working Podman
installation.[]{.sentence-end} As stated in *[Chapter
3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the* *First
Container*, all the examples in this book have been executed on a Fedora
40 system or later but can be reproduced on an operating system of your
choice.

Having a good understanding of the topics that we covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}*, *Managing Running* *Containers*,
will help you easily grasp concepts regarding container images.

# The Open Container Initiative image format {#Chapter_8.xhtml#h1_202 .heading-1}

As we described in *[Chapter 1](#Chapter_1.xhtml#h1_14){.chapref}*,
*Introduction to Container Technology*, back in 2013, Docker was
introduced in the container landscape and became very popular rapidly.

At a high level, the Docker team introduced the concept of container
images and container registries, which was a
game-changer.[]{.sentence-end} Another important step was being able to
*extract* containerd projects from Docker and donate them to the **Cloud
Native Computing Foundation** (**CNCF**).[]{.sentence-end} This
[]{#Chapter_8.xhtml#idx_054dd493 .index-entry
index-entry="Cloud Native Computing Foundation (CNCF)"}motivated the
open source community to start working seriously on container engines
that could be injected into an orchestration layer, such as Kubernetes.

Similarly, in 2015, Docker, with the help of many other companies (Red
Hat, AWS, Google, Microsoft, IBM, and others), started
[]{#Chapter_8.xhtml#idx_1abce2ec .index-entry
index-entry="Open Container Initiative (OCI)"}the **Open Container
Initiative** (**OCI**) under the Linux Foundation umbrella.

These contributors developed the Runtime Specification (runtime-spec)
and the Image Specification (image-spec) for describing how the API and
the architecture for new container engines should be created in the
future.

After a few months of work, the OCI team released its first
implementation of a container runtime that adhered to their
specifications; the project was []{#Chapter_8.xhtml#idx_9e74b4bd
.index-entry index-entry="runc"}named **runc**.

It\'s worth looking at the container image specification in detail and
going over some theory behind the practice that we introduced in
*[Chapter 2](#Chapter_2.xhtml#h1_52){.chapref}*, *Podman* *and*
*Docker*.

The specification defines an OCI container image that consists of the
following:

-   **Manifest**: This []{#Chapter_8.xhtml#idx_fee1d176 .index-entry
    index-entry="OCI container image:manifest"}contains the metadata of
    the contents and dependencies of the image.[]{.sentence-end} This
    also includes the ability to identify one or more filesystem
    archives that will be unpacked to get the final runnable filesystem.
-   **Image** **index (optional)**: This represents a list of manifests
    and descriptors that can []{#Chapter_8.xhtml#idx_2b30f8a8
    .index-entry index-entry="OCI container image:image index"}provide
    different implementations of the image, depending on the target
    platform.
-   **Set of** **filesystem** **layers**: The actual set of layers that
    should be merged to build the final[]{#Chapter_8.xhtml#idx_cfe5e8fa
    .index-entry
    index-entry="OCI container image:set of filesystem layers"}
    container filesystem.
-   **Configuration**: This contains[]{#Chapter_8.xhtml#idx_36291cd3
    .index-entry index-entry="OCI container image:configuration"} all
    the information that\'s required by the container runtime engine to
    effectively run the application, such as arguments and environment
    variables.

We will not deep dive into every element of the OCI Image Specification
as it is out of scope, but the Image Manifest deserves a closer look.

## OCI Image Manifest {#Chapter_8.xhtml#h2_203 .heading-2}

The Image Manifest defines a []{#Chapter_8.xhtml#idx_652281e8
.index-entry index-entry="OCI Image Manifest"}set of layers and the
configuration for a single container image that is built for a specific
architecture and an operating system.

Let\'s explore the details of the OCI Image Manifest by looking at the
following example:

``` {.programlisting .snippet-code}
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 7023,
    "digest": "sha256:b5b2b2c507a0944348e0303114d8d93aaaa081732b86451d9bce1f432a537bc7"
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 32654,
      "digest": "sha256:9834876dcfb05cb167a5c24953eba58c4ac89b1adf57f28f2f9d09af107ee8f0"
    }
  ],
  "annotations": {
    "com.example.key1": "value1",
    "com.example.key2": "value2"
  }
}
```

Here, we are using the following keywords:

-   `schemaVersion`{.inlineCode}: A property that must be set to a value
    of `2`{.inlineCode}.[]{.sentence-end} This ensures backward
    compatibility with Docker.
-   `config`{.inlineCode}: A property that references an image\'s
    configuration through a digest:
    -   `mediaType`{.inlineCode}: This property defines the actual
        configuration format (just one currently)
-   `layers`{.inlineCode}: This property provides an array of descriptor
    objects:
    -   `m`{.inlineCode}`ediaType`{.inlineCode}: In this case, this
        descriptor should be one of the media types that are allowed for
        the layer\'s descriptors
-   `annotations`{.inlineCode}: This property defines additional
    metadata for the Image Manifest.

To summarize, the main goal of[]{#Chapter_8.xhtml#idx_61cced1f
.index-entry index-entry="OCI Image Manifest"} the specification is to
make interoperable tools for building, transporting, and preparing a
container image to be run.

The Image Manifest Specification has three main goals:

-   To enable hashing for the image\'s configuration, thereby generating
    a unique ID
-   To allow multi-architecture images due to its high-level manifest
    (image index) that references platform-specific versions of the
    Image Manifest
-   To be able to easily []{#Chapter_8.xhtml#idx_001f9c54 .index-entry
    index-entry="OCI Image Manifest"}translate the container image into
    the OCI Runtime Specification

Now, let\'s learn where these container images come from.

# Where do container images come from? {#Chapter_8.xhtml#h1_204 .heading-1}

In the previous chapters, we[]{#Chapter_8.xhtml#idx_f18ba8fa
.index-entry index-entry="container images"} used pre-built images to
run, build, or manage a container, but where do these container images
come from?

How can we dig into their source commands or into the
Dockerfile/Containerfile that\'s used to build them?

Well, as we\'ve mentioned previously, Docker introduced the concept of a
container image and container registry for storing these images -- even
publicly.[]{.sentence-end} The most famous container registry is Docker
Hub, but after Docker\'s introduction, other cloud container registries
were released too.

We can choose between the following cloud container registries:

-   **Docker** **Hub**: This is the[]{#Chapter_8.xhtml#idx_514556c7
    .index-entry index-entry="Docker Hub"} hosted registry solution by
    Docker Inc.[]{.sentence-end} This registry also hosts official
    repositories and security-verified images for some popular open
    source projects.
-   **Quay.io**: This is the[]{#Chapter_8.xhtml#idx_568b42aa
    .index-entry index-entry="Quay.io"} hosted registry solution that
    was born under the CoreOS company, though it is now part of Red
    Hat.[]{.sentence-end} It offers private and public repositories,
    automated scanning for security purposes, image builds, and
    integration with popular Git public repositories.
-   **Linux** **distribution** **registries**: Popular Linux
    distributions are typically community-baked, such as
    []{#Chapter_8.xhtml#idx_154c4278 .index-entry
    index-entry="Red Hat Enterprise Linux (RHEL)"}Fedora Linux,
    or[]{#Chapter_8.xhtml#idx_e0487935 .index-entry
    index-entry="Linux distribution registries"} enterprise-based, such
    as **Red Hat Enterprise Linux** (**RHEL**).[]{.sentence-end} They
    usually offer public container registries, though these are often
    only available for projects or packages that have already been
    provided as system packages.[]{.sentence-end} These registries are
    not available to end users and they are fed by the Linux
    distributions\' maintainers.
-   **Public** **cloud** **registries**: Amazon, Google, Microsoft, and
    other public cloud providers offer public and
    []{#Chapter_8.xhtml#idx_e8f30684 .index-entry
    index-entry="public cloud registries"}private container registries
    for their customers.

We will explore these registries in more detail in *[Chapter
9](#Chapter_9.xhtml#h1_219){.chapref}*, *Pushing Images to a Container
Registry*.

Docker Hub, as well as Quay.io, are []{#Chapter_8.xhtml#idx_7398705a
.index-entry index-entry="Quay.io"}public container registries where we
can find container images that []{#Chapter_8.xhtml#idx_52eb8976
.index-entry index-entry="Docker Hub"}have been created by
anyone.[]{.sentence-end} These registries are full of useful custom
images that we can use as starting points for testing container images
quickly and easily.

Just downloading and running a container image is not always the best
thing to do -- we could hit very old and outdated software that could be
vulnerable to some known public vulnerability; or, even worse, we could
download and execute some malicious code that could compromise our whole
infrastructure.

For this reason, Docker Hub and Quay.io usually offer features to
underline where such images come from.[]{.sentence-end} Let\'s inspect
them.

## Docker Hub container registry service {#Chapter_8.xhtml#h2_205 .heading-2}

As we introduced earlier, Docker Hub[]{#Chapter_8.xhtml#idx_d9d4da4f
.index-entry index-entry="Docker Hub"} is the most famous container
registry available.[]{.sentence-end} It hosts multiple container images
for community or enterprise products.

By looking at the detail page of a container image, we can easily
discover all the required information about that project and its
container images.[]{.sentence-end} The following screenshot shows Alpine
Linux\'s Docker Hub page:

<figure class="mediaobject">
<img src="images/B31467_8_1.png"
style="width:528.0px; height:275.99009083402143px;" alt="Image 1" />
</figure>

Figure 8.1 -- Alpine Linux container image on Docker Hub

As you can see, at the top of the page, we can find helpful information,
the latest tags, the supported architectures, and useful links to the
project\'s documentation and the issue-reporting system.

On the []{#Chapter_8.xhtml#idx_0569ac14 .index-entry
index-entry="Docker Hub"}Docker Hub page, we can find the *Official
Image* tag, just after the image\'s name, when that image is part of
Docker\'s Official Images program.[]{.sentence-end} The images in this
program, which are reported as official, are curated directly by the
Docker team in collaboration with the upstream projects\' maintainers.

 packt_tip
**Important note**

If you want to look at this page in more depth, point your web browser
to
[[https://hub.docker.com/\_/alpine]{.url}](https://hub.docker.com/_/alpine){style="text-decoration: none;"}.


Another important feature that\'s offered
by[]{#Chapter_8.xhtml#idx_01b76baa .index-entry
index-entry="Docker Hub"} Docker Hub (not only for official images) is
the ability to look into the Dockerfile that was used to create a
certain image.

If we click on one of the available tags on the container image page, we
can easily look at the Dockerfile of that container image tag.

Clicking on the tag named **edge** on that page will redirect us to the
GitHub page of the docker-alpine project, which is defined as the
following Dockerfile:
[[https://github.com/alpinelinux/docker-alpine/blob/edge/x86_64/Dockerfile]{.url}](https://github.com/alpinelinux/docker-alpine/blob/edge/x86_64/Dockerfile){style="text-decoration: none;"}.

We should always look out for and prefer official
images.[]{.sentence-end} If an official image is not available or it
does not fit our needs, then we need to inspect the Dockerfile that the
content creator published, as well as the container image.

Docker Hub also includes an additional service called
Scout.[]{.sentence-end} It is a tool that helps developers find and fix
security issues in their Docker images.[]{.sentence-end} It scans images
for vulnerabilities, provides remediation advice, and integrates with
CI/CD pipelines to ensure that security is addressed throughout the
development process.

 note
**Important note**

At the time of writing, Docker Hub has several limitations for
non-paying authenticated users and for unauthenticated ones as
well.[]{.sentence-end} A common limitation is the number of images
pulled per hour per user.[]{.sentence-end} You can learn more at the
following URL:
[[https://docs.docker.com/docker-hub/download-rate-limit/]{.url}](https://docs.docker.com/docker-hub/download-rate-limit/){style="text-decoration: none;"}.


## Quay.io container registry service {#Chapter_8.xhtml#h2_206 .heading-2}

Quay[]{#Chapter_8.xhtml#idx_6eaff381 .index-entry index-entry="Quay"} is
a container registry service that was acquired by CoreOS in 2014 and is
now part of the Red Hat ecosystem.[]{.sentence-end} **Quay.io** is
the[]{#Chapter_8.xhtml#idx_b36f8728 .index-entry index-entry="Quay.io"}
managed service, offered as SaaS, of the open source project Quay.

The online registry allows its users to be more cautious once they\'ve
chosen a container image by providing security scanning software.

Quay, and Quay.io as well, adopts the Clair project, a leading container
vulnerability scanner that displays reports on the repository tags web
page, as shown in the following screenshot:

<figure class="mediaobject">
<img src="images/B31467_8_2.png"
style="width:528.0px; height:200.56151940545004px;" alt="Image 2" />
</figure>

Figure 8.2 -- Quay vulnerability security scan page

On this page, we can []{#Chapter_8.xhtml#idx_2804f592 .index-entry
index-entry="Quay:vulnerability security scan page"}click on
**SECURITY** **SCAN** to inspect the details of that security
scan.[]{.sentence-end} If you want to learn more about this feature,
please go to
[[https://quay.io/repository/openshift-release-dev/ocp-release?tab=tags]{.url}](https://quay.io/repository/openshift-release-dev/ocp-release?tab=tags){style="text-decoration: none;"}.

As we\'ve seen, using a public registry that offers every user the
security scan feature could help ensure that we choose the right and
most secure flavor of the container image we are searching for.

At the moment of writing, Quay.io has no limits on the number of
container images pulled; the free tier offers five private repositories
and unlimited public repositories.

## Red Hat Container Registry {#Chapter_8.xhtml#h2_207 .heading-2}

Red Hat Container Registry[]{#Chapter_8.xhtml#idx_1a297474 .index-entry
index-entry="Red Hat Container Registry"} is the
[]{#Chapter_8.xhtml#idx_cafafecc .index-entry
index-entry="OpenShift Container Platform (OCP)"}default container
registry for RHEL and Red Hat **OpenShift** **Container Platform**
(**OCP**) users.[]{.sentence-end} The web interface of this registry is
publicly accessible to any user, whether they are authenticated or not,
although almost all the images that are provided are reserved for paid
users (RHEL or OCP users).

We are talking about this registry because it combines all the features
we talked about previously.[]{.sentence-end} This registry offers the
following to its users:

-   Official container images by Red Hat
-   Containerfile/Dockerfile sources to inspect the content of the image
-   Security reports (index) about every container image that\'s
    distributed

The following screenshot []{#Chapter_8.xhtml#idx_ec302df1 .index-entry
index-entry="Red Hat Container Registry"}shows what this information
looks like on the **Red Hat Ecosystem Catalog** page:

<figure class="mediaobject">
<img src="images/B31467_8_3.png"
style="width:528.0px; height:239.36581337737405px;" alt="Image 3" />
</figure>

Figure 8.3 -- Mariadb container image description page on Red Hat
Ecosystem Catalog

As we can see, the page shows the description of the container image we
have selected (MariaDB database), the version, the available
architectures, and various tags that can be selected from the respective
drop-down menu.[]{.sentence-end} Some tabs also mention the keywords we
are interested in: **Security** and **Dockerfile**.

By clicking on the **Security** tab, we can see the status of the
vulnerability scan that was executed for that image tag, as shown in the
following screenshot:

<figure class="mediaobject">
<img src="images/B31467_8_4.png"
style="width:528.0px; height:275.5732009925559px;" alt="Image 4" />
</figure>

Figure 8.4 -- Mariadb container image Security page on Red Hat Ecosystem
Catalog

As we can[]{#Chapter_8.xhtml#idx_9d948382 .index-entry
index-entry="Red Hat Container Registry"} see, at the time of writing,
for this latest image tag, a security vulnerability has already been
identified that\'s affecting three packages.[]{.sentence-end} To the
right, we can find the Red Hat Advisory ID, which is
linked[]{#Chapter_8.xhtml#idx_03606bab .index-entry
index-entry="Common Vulnerabilities and Exposures (CVEs)"} to the public
**Common Vulnerabilities and Exposures** (**CVEs**).

By clicking on the **Dockerfile** tab, we can look at the source
Containerfile that was used to build that container image:

<figure class="mediaobject">
<img src="images/B31467_8_5.png"
style="width:528.0px; height:275.5732009925559px;" alt="Image 5" />
</figure>

Figure 8.5 -- Mariadb container image Dockerfile page on Red Hat
Ecosystem Catalog

We can look at the source []{#Chapter_8.xhtml#idx_c2210816 .index-entry
index-entry="Red Hat Container Registry"}Containerfile that was used to
build the container image that we are going to pull and
run.[]{.sentence-end} This is a great feature that we can access by
clicking on the same description page of the container image we are
looking for.

If you want to take a closer look at the preceding container image, you
can visit this URL:
[[https://catalog.redhat.com/software/containers/rhel9/mariadb-105/61a6084dbfd4a5234d596220]{.url}](https://catalog.redhat.com/software/containers/rhel9/mariadb-105/61a6084dbfd4a5234d596220){style="text-decoration: none;"}.

Another interesting container image available on the Red Hat enterprise
registry is the UBI:

<figure class="mediaobject">
<img src="images/B31467_8_6.png"
style="width:528.0px; height:275.5732009925559px;" alt="Image 6" />
</figure>

Figure 8.6 -- UBI 9 container image Dockerfile page on Red Hat Ecosystem
Catalog

**UBI** stands for **Universal Base Image**.[]{.sentence-end} It
is[]{#Chapter_8.xhtml#idx_af4f9a8d .index-entry
index-entry="Universal Base Image (UBI)"} an initiative that was
launched by Red Hat that lets every user (Red Hat customer or not) open
Red Hat container images.[]{.sentence-end} This allows the Red Hat
ecosystem to expand by leveraging all the previously mentioned services
that are offered by the Red Hat Ecosystem Catalog, as
well[]{#Chapter_8.xhtml#idx_e24ee2fa .index-entry
index-entry="Red Hat Container Registry"} as by leveraging the updated
packages that are directly from Red Hat.[]{.sentence-end} Depending on
the RHEL release version, you will find a respective UBI version
release.

You can take a look at the container image by visiting this URL:
[[https://catalog.redhat.com/software/containers/ubi9/ubi/615bcf606feffc5384e8452e]{.url}](https://catalog.redhat.com/software/containers/ubi9/ubi/615bcf606feffc5384e8452e){style="text-decoration: none;"}.

We will talk more about UBI and its container images later in this
chapter.

# Trusted container image sources {#Chapter_8.xhtml#h1_208 .heading-1}

In the previous section, we[]{#Chapter_8.xhtml#idx_31501b9a .index-entry
index-entry="trusted container image sources"} defined the central role
of the image registry as a source of truth for valid, usable
images.[]{.sentence-end} In this section, we want to stress the
importance of adopting trusted images that come from trusted sources.

An OCI image[]{#Chapter_8.xhtml#idx_dd004f93 .index-entry
index-entry="OCI image"} is used to package binaries and runtimes in a
structured filesystem with the purpose of delivering a specific
service.[]{.sentence-end} When we pull that image and run it on our
systems without any kind of control, we implicitly trust the author not
to have tampered with its content by using malicious
components.[]{.sentence-end} But nowadays, trust is something that
cannot be granted so easily.

As we will see in *[Chapter 10](#Chapter_10.xhtml#h1_248){.chapref}*,
*Securing Containers*, there are many attack use cases and malicious
behaviors that can be conducted from a container: privilege escalation,
data exfiltration, and miners are just a few examples.[]{.sentence-end}
These behaviors can be amplified when containers that are run inside
Kubernetes clusters (many thousands of clusters) can spawn malicious
Pods across the infrastructure easily.

To help security teams mitigate this, the MITRE Corporation periodically
releases **MITRE** **ATT&CK** matrices[]{#Chapter_8.xhtml#idx_3a43a9b7
.index-entry index-entry="MITRE ATT&CK matrices"} to identify all the
possible attack strategies and their related techniques, with real-life
use cases, and their detection and mitigation best
practices.[]{.sentence-end} One of these matrices is dedicated to
containers, where many techniques are implemented based on insecure
images, where malicious behaviors can be conducted successfully.

 packt_tip
**Important** **note**

You should prefer images that come from a registry that supports
vulnerability scans.[]{.sentence-end} If the scan results are available,
check them carefully and avoid using images that spot critical
vulnerabilities.


With this in mind, what is the first step of creating a secure
cloud-native infrastructure? The answer is choosing images that only
come from trusted sources, and the first thing to do is to configure
trusted registries and patterns to block disallowed
ones.[]{.sentence-end} We will cover this in the following subsection.

## Managing trusted registries {#Chapter_8.xhtml#h2_209 .heading-2}

As shown in *[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running
the* *First Container*, in the *Preparing your environment* section,
Podman can manage []{#Chapter_8.xhtml#idx_3a45c008 .index-entry
index-entry="trusted registries:managing"}trusted registries with config
files.

The
`/`{.inlineCode}`etc`{.inlineCode}`/containers/`{.inlineCode}`registries.conf`{.inlineCode}
file (overridden by the user-related
`$HOME/.`{.inlineCode}`config`{.inlineCode}`/containers/`{.inlineCode}`registries.conf`{.inlineCode}
file, if present) manages a list of trusted registries that Podman can
safely contact to search and pull images.

Let\'s look at an example of this file:

``` {.programlisting .snippet-code}
unqualified-search-registries = ["docker.io", "quay.io"]

[[registry]]
location = "registry.example.com:5000"
insecure = false
```

This file helps us define the trusted registries that can be used by
Podman, so it deserves a detailed analysis.

Podman accepts both **unqualified** and **fully** **qualified**
images.[]{.sentence-end} The difference is quite simple and can be
illustrated as follows:

-   A fully qualified image includes a registry server FQDN, namespace,
    image name, and tag.[]{.sentence-end} For example,
    `docker.io`{.inlineCode}`/library/`{.inlineCode}`nginx:latest`{.inlineCode}
    is a fully qualified image.[]{.sentence-end} It has a full name that
    cannot be confused with any other Nginx image.
-   An unqualified image only includes the image\'s
    name.[]{.sentence-end} For example, the `nginx`{.inlineCode} image
    can have multiple instances in the searched
    registries.[]{.sentence-end} The majority of the images that result
    from the basic
    `podman`{.inlineCode}` search `{.inlineCode}`nginx`{.inlineCode}
    command will not be official and should be analyzed in detail to
    ensure they\'re trusted.[]{.sentence-end} The output can be filtered
    by the `OFFICIAL`{.inlineCode} flag and by the number of stars (more
    is better).

The first global setting of the registries configuration file is the
`unqualified-search-registry`{.inlineCode} array, which defines the
search list of registries for unqualified images.[]{.sentence-end} When
the user runs the
`podman`{.inlineCode}` search `{.inlineCode}`<`{.inlineCode}`image`{.inlineCode}`_name`{.inlineCode}`>`{.inlineCode}
command, Podman will search across the registries defined in this list.

By removing a registry[]{#Chapter_8.xhtml#idx_e102b4df .index-entry
index-entry="trusted container image sources"} from the list, Podman
will stop searching the registry.[]{.sentence-end} However, Podman will
still be able to pull a fully qualified image from a foreign registry.

To manage single registries and create matching patterns
for[]{#Chapter_8.xhtml#idx_160b0bbc .index-entry
index-entry="Tom's Obvious, Minimal Language (TOML) tables"} specific
images, we can use the `[[registry]]`{.inlineCode} **Tom\'s Obvious,
Minimal Language** (**TOML**) tables.[]{.sentence-end} The main settings
of these tables are as follows:

-   `prefix`{.inlineCode}: This is used to define the image names and
    can support multiple formats.[]{.sentence-end} In general, we can
    define images by following the
    `host[`{.inlineCode}`:port]/namespace[/_namespace_…]/repo(:_tag|@digest)`{.inlineCode}
    pattern, though simpler patterns, such as
    `host[:port]`{.inlineCode}, `host[:port]/namespace`{.inlineCode},
    and even `[*.]`{.inlineCode}`host`{.inlineCode}, can be
    applied.[]{.sentence-end} Following this approach, users can define
    a generic prefix for a registry or a more detailed prefix to match a
    specific image or tag.[]{.sentence-end} Given a fully qualified
    image, if two `[[registry]]`{.inlineCode} tables have a prefix with
    a partial match, the longest matching pattern will be used.
-   `insecure`{.inlineCode}: This is a Boolean (`true`{.inlineCode} or
    `false`{.inlineCode}) that allows unencrypted HTTP connections or
    TLS connections based on untrusted certificates.
-   `blocked`{.inlineCode}: This is a Boolean (`true`{.inlineCode} or
    `false`{.inlineCode}) that\'s used to define blocked
    registries.[]{.sentence-end} If it\'s set to `true`{.inlineCode},
    the registries or images that match the prefix are blocked.
-   `location`{.inlineCode}: This field defines the registry\'s
    location.[]{.sentence-end} By default, it is equal to
    `prefix`{.inlineCode}, but it can have a different
    value.[]{.sentence-end} In that case, a pattern that matches a
    custom prefix namespace will resolve to the `location`{.inlineCode}
    value.

Along with the main `[[registry]]`{.inlineCode} table, we can define an
array of
`[[`{.inlineCode}`registry.mirror`{.inlineCode}`]]`{.inlineCode} TOML
tables to provide alternate paths to the main registry or registry
namespace.

When multiple mirrors are provided, Podman will search across them first
and then fall back to the location that\'s defined in the main
`[[registry]]`{.inlineCode} table.

The following example extends the previous one by defining a namespaced
registry entry and its mirror:

``` {.programlisting .snippet-code}
unqualified-search-registries = ["docker.io", "quay.io"]

[[registry]]
location = "registry.example.com:5000/foo"
insecure = false
[[registry.mirror]]
location = "mirror1.example.com:5000/bar"
[[registry.mirror]]
location = "mirror2.example.com:5000/bar"
```

According to this []{#Chapter_8.xhtml#idx_aa4a85cc .index-entry
index-entry="trusted container image sources"}example, if a user tries
to pull the image tagged as
`registry.example.com:5000`{.inlineCode}`/foo/`{.inlineCode}`app:latest`{.inlineCode},
Podman will try
`mirror1.example.com:5000`{.inlineCode}`/bar/`{.inlineCode}`app:latest`{.inlineCode},
then
`mirror2.example.com:5000`{.inlineCode}`/bar/`{.inlineCode}`app:latest`{.inlineCode},
and fall back to
`registry.example.com:5000`{.inlineCode}`/foo/`{.inlineCode}`app:latest`{.inlineCode}
in case a failure occurs.

Using a prefix provides even more flexibility.[]{.sentence-end} In the
following example, all the images that match
`example.com`{.inlineCode}`/foo`{.inlineCode} will be redirected to
mirror locations and fall back to the main location at the end:

``` {.programlisting .snippet-code}
unqualified-search-registries = ["docker.io", "quay.io"]
[[registry]]
prefix = "example.com/foo"
location = "registry.example.com:5000/foo"
insecure = false
[[registry.mirror]]
location = "mirror1.example.com:5000/bar"
[[registry.mirror]]
location = "mirror2.example.com:5000/bar"
```

In this example, when we pull the
`example.com`{.inlineCode}`/foo/`{.inlineCode}`app:latest`{.inlineCode}
image, Podman will attempt
`mirror1.example.com:5000`{.inlineCode}`/bar/`{.inlineCode}`app:latest`{.inlineCode},
followed by
`mirror2.example.com:5000`{.inlineCode}`/bar/`{.inlineCode}`app:latest`{.inlineCode}
and
`registry.example.com:5000`{.inlineCode}`/foo/`{.inlineCode}`app:latest`{.inlineCode}.

It is possible to use mirroring in a more advanced way, such as
replacing public registries with private mirrors in disconnected
environments.[]{.sentence-end} The following example remaps the
`docker.io`{.inlineCode} and `quay.io`{.inlineCode} registries to
a[]{#Chapter_8.xhtml#idx_17f1557b .index-entry
index-entry="trusted container image sources"} private mirror with
different namespaces:

``` {.programlisting .snippet-code}
[[registry]]
prefix="quay.io"
location="mirror-internal.example.com/quay"
[[registry]]
prefix="docker.io"
location="mirror-internal.example.com/docker"
```

 note
**Important** **note**

Mirror registries should be kept up to date with mirrored
repositories.[]{.sentence-end} For this reason, administrators or SRE
teams should implement an image sync policy to keep the repositories
updated.


Finally, we are going to learn how to block a source that is not
considered trusted.[]{.sentence-end} This behavior could impact a single
image, a namespace, or a whole registry.

The following example tells Podman not to search for or pull images from
a blocked registry:

``` {.programlisting .snippet-code}
[[registry]]
location = "registry.rogue.io"
blocked = true
```

It is possible to refine the blocking policy by passing a specific
namespace without blocking the whole registry.[]{.sentence-end} In the
following example, every image search or pull that matches the
`quay.io`{.inlineCode}`/foo`{.inlineCode} namespace pattern defined in
the `prefix`{.inlineCode} field is blocked:

``` {.programlisting .snippet-code}
[[registry]]
prefix = "quay.io/foo/"
location = "quay.io"
blocked = true
```

According to this pattern, if[]{#Chapter_8.xhtml#idx_782da9af
.index-entry index-entry="trusted container image sources"} the user
tries to pull an image called
`quay.io`{.inlineCode}`/foo/`{.inlineCode}`nginx:latest`{.inlineCode} or
`quay.io`{.inlineCode}`/foo/`{.inlineCode}`httpd:v2.4`{.inlineCode}, the
prefix is matched and the pull is blocked.[]{.sentence-end} No blocking
action occurs when the
`quay.io`{.inlineCode}`/bar/`{.inlineCode}`fedora:latest`{.inlineCode}
image is pulled.

Users can also define a very specific blocking rule for a single image
or even a single tag by using the same approach that was described for
namespaces.[]{.sentence-end} The following example blocks a specific
image tag:

``` {.programlisting .snippet-code}
[[registry]]
prefix = "internal-registry.example.com/dev/app:v0.1"
location = "internal-registry.example.com "
blocked = true
```

It is possible to combine many blocking rules and add mirror tables on
top of them.

 packt_tip
**Important** **note**

In a complex infrastructure with many machines running Podman (for
example, developer workstations), a clever idea would be to keep the
registry\'s configuration file updated using configuration management
tools and declaratively apply the registry\'s filters.


**Fully qualified image names** (**FQNs**) can become quite long if we
sum up the registry FQDN, namespace(s), repository, and tags.

While using FQNs such as
`quay.io`{.inlineCode}`/`{.inlineCode}`podman`{.inlineCode}`/`{.inlineCode}`stable:latest`{.inlineCode}
is the gold standard for security and automation, they are often
cumbersome to type manually.[]{.sentence-end} To bridge the gap between
convenience and security, Podman utilizes a short-name aliasing system.

Normally, when you provide an *unqualified* name (for example,
`podman`{.inlineCode}` pull fedora`{.inlineCode}), Podman must iterate
through a list of registries defined in the
`[`{.inlineCode}`registries.search`{.inlineCode}`]`{.inlineCode} table
of your configuration.[]{.sentence-end} This can be slow and, if not
configured correctly, potentially insecure.

Aliases change this behavior by short-circuiting the
search.[]{.sentence-end} If a short name matches an entry in an alias
table, Podman immediately resolves it to the specific, fully qualified
name provided in the mapping, bypassing the search registry list
entirely.

On Fedora and RHEL, you don\'t have to build this list from
scratch.[]{.sentence-end} The system comes pre-configured with an
[]{#Chapter_8.xhtml#idx_7d70c80e .index-entry
index-entry="trusted container image sources"}extensive library of
trusted mappings located at the following:

``` {.programlisting .snippet-code}
/etc/containers/registries.conf.d/000-shortnames.conf
```

This file is maintained by the community and operating system vendors to
ensure that common names point to their most logical and secure upstream
sources.[]{.sentence-end} For example, an entry in this file might look
as follows:

``` {.programlisting .snippet-code}
[aliases]
  "fedora" = "registry.fedoraproject.org/fedora"
  "ubi8" = "registry.access.redhat.com/ubi8"
  "alpine" = "docker.io/library/alpine"
```

When an alias matches a short name, it is immediately used without the
registries defined in the `unqualified-search-registries`{.inlineCode}
list being searched.

 note
**Important** **note**

We can create custom files inside the
`/`{.inlineCode}`etc`{.inlineCode}`/containers/`{.inlineCode}`registries.conf.d`{.inlineCode}`/`{.inlineCode}
folder to define aliases without bloating the main configuration file.

While aliases simplify management, they have specific boundaries:

-   **No** **tags or** **digests**: Aliases resolve the repository path,
    not a specific version.[]{.sentence-end} For example, if you alias
    `my-app`{.inlineCode} to
    `quay.io`{.inlineCode}`/org/my-app`{.inlineCode}, running
    `podman`{.inlineCode}` pull `{.inlineCode}`my-app`{.inlineCode}`:v2`{.inlineCode}
    will correctly resolve to
    `quay.io`{.inlineCode}`/org/`{.inlineCode}`my-app:v2`{.inlineCode}.[]{.sentence-end}
    The alias handles the prefix, while the tag remains dynamic.
-   **Precedence**: Local user-defined aliases (often found in
    `$HOME/.`{.inlineCode}`config`{.inlineCode}`/containers/`{.inlineCode}`registries.conf.d`{.inlineCode}`/`{.inlineCode})
    will typically override the global system aliases found in
    `/`{.inlineCode}`etc`{.inlineCode}`/containers/`{.inlineCode}.[]{.sentence-end}


With that, we have learned how to manage trusted sources and block
unwanted images, registries, or namespaces.[]{.sentence-end} This is a
security best practice, but it does not relieve us from the
responsibility of choosing a valid image that fits our needs while being
trustworthy and having the lowest attack surface
possible.[]{.sentence-end} This is also true when we\'re building a new
application, where base images must be lightweight and
secure.[]{.sentence-end} Red Hat UBI images can be a helpful solution
for this problem.

# Introducing the Universal Base Image {#Chapter_8.xhtml#h1_210 .heading-1}

When working on enterprise []{#Chapter_8.xhtml#idx_1908d2f7 .index-entry
index-entry="Universal Base Image (UBI)"}environments, many users and
companies adopt RHEL as the operating system of choice to execute
workloads reliably and securely.[]{.sentence-end} RHEL-based container
images are available too, and they take advantage of the same package
versioning as the operating system release.[]{.sentence-end} All the
security updates that are released for RHEL are immediately applied to
OCI images, making them wealthy, secure images to build production-grade
applications with.

Unfortunately, RHEL images are not publicly available without a Red Hat
subscription.[]{.sentence-end} Users who have activated a valid
subscription can use them freely on their RHEL systems and build custom
images on top of them, but they are not freely redistributable without
breaking the Red Hat enterprise agreement.

So, why worry? There are plenty of commonly used images that can replace
them.[]{.sentence-end} This is true, but when it comes to reliability
and security, many companies choose to stick to an enterprise-grade
solution, and this is no exception for containers.

For these reasons, and to address the redistribution limitations of RHEL
images, Red Hat created **UBI**.[]{.sentence-end} UBI images are freely
redistributable; can be used to build containerized applications,
middleware, and utilities; and are constantly maintained and upgraded by
Red Hat.

UBI images are based on the currently supported versions of
RHEL.[]{.sentence-end} At the time of writing, UBI 8, UBI 9, and UBI 10
images are available, based on RHEL 8, RHEL 9, and RHEL 10,
respectively.[]{.sentence-end} In general, UBI images can be considered
a subset of the RHEL operating system.

All UBI images are available on the public Red Hat registry
([[registry.access.redhat.com]{.url}](https://registry.access.redhat.com){style="text-decoration: none;"})
and Docker Hub
([[docker.io]{.url}](https://docker.io){style="text-decoration: none;"}).

There are currently four different flavors of UBI images, each one
specialized for a particular use case:

-   **Standard**: This is[]{#Chapter_8.xhtml#idx_694ea6a6 .index-entry
    index-entry="Universal Base Image (UBI):standard"} the standard UBI
    image.[]{.sentence-end} It has the most features and package
    availability.
-   **Minimal**: This is a[]{#Chapter_8.xhtml#idx_43031c9e .index-entry
    index-entry="Universal Base Image (UBI):minimal"} stripped-down
    version of the standard image with minimalistic package management.
-   **Micro**: This is a[]{#Chapter_8.xhtml#idx_d112baf2 .index-entry
    index-entry="Universal Base Image (UBI):micro"} UBI version with the
    smallest footprint possible, without a package manager.
-   **Init**: This is a[]{#Chapter_8.xhtml#idx_f3d27bfd .index-entry
    index-entry="Universal Base Image (UBI):init"} UBI image that
    includes the `systemd`{.inlineCode} init system so that you can
    manage the execution of multiple services in a single container.

All of these are *free to use and redistribute* inside custom
images.[]{.sentence-end} Unlike standard RHEL images, UBI is freely
redistributable, allowing developers to build applications on a trusted
foundation and share them anywhere, including public registries, without
requiring a Red Hat subscription.[]{.sentence-end} In addition to that,
UBI bridges the gap between development and production; you can build on
a laptop running Fedora or a CI/CD pipeline in the cloud and deploy to a
production RHEL or OpenShift environment with zero compatibility
concerns.

Let\'s describe each in detail, starting with the UBI Standard image.

## The UBI Standard image {#Chapter_8.xhtml#h2_211 .heading-2}

The UBI Standard image is the[]{#Chapter_8.xhtml#idx_7d9d5331
.index-entry index-entry="UBI Standard image"} most complete UBI image
version and the closest one to standard RHEL images.[]{.sentence-end}
It[]{#Chapter_8.xhtml#idx_5e7b4927 .index-entry
index-entry="yum package manager"} includes the **yum** package manager,
which is available in RHEL, and can be customized by installing the
packages that are available in its dedicated software repositories, that
is, `ubi`{.inlineCode}`-8-`{.inlineCode}`baseos`{.inlineCode} and
`ubi`{.inlineCode}`-8-`{.inlineCode}`appstream`{.inlineCode}.

The following example shows a Dockerfile/Containerfile that uses a
standard UBI 8 image to build a minimal `httpd`{.inlineCode} server:

``` {.programlisting .snippet-code}
FROM registry.access.redhat.com/ubi8

# Update image and install httpd
RUN yum update -y && yum install -y httpd && yum clean all

# Expose the default httpd port 80
EXPOSE 80

# Run the httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

The UBI Standard image was designed for generic applications and
packages that are available on RHEL and already includes a curated list
of basic system tools (including `curl`{.inlineCode},
`tar`{.inlineCode}, `vi`{.inlineCode}, `sed`{.inlineCode}, and
`gzip`{.inlineCode}) and OpenSSL libraries while still retaining a small
size (around 230 MiB): fewer packages mean more lightweight images and a
smaller attack surface.

In *[Chapter 6](#Chapter_6.xhtml#h1_163){.chapref}*, *Meet* *Buildah*
*-- Building Containers from Scratch*, we learned how to build a new
container image starting from a Containerfile or a Dockerfile, so you
can leverage Buildah to test the previous example.

If the UBI Standard image is still considered too big, UBI Minimal can
be a good fit.

## The UBI Minimal image {#Chapter_8.xhtml#h2_212 .heading-2}

The UBI Minimal image is a[]{#Chapter_8.xhtml#idx_f209bae0 .index-entry
index-entry="UBI Minimal image"} stripped-down version of UBI Standard
and was designed for self-consistent applications and their runtimes
(Python, Ruby, Node.js, and so on).[]{.sentence-end} For this reason,
it\'s smaller in size, has a small selection of packages, and doesn\'t
include the yum package manager; this has been replaced with a minimal
tool called `microdnf`{.inlineCode}.[]{.sentence-end} The UBI Minimal
image is smaller than UBI Standard and is roughly half its size.

The following example shows a Dockerfile/Containerfile using a UBI 8
minimal image to build a proof-of-concept Python web server:

``` {.programlisting .snippet-code}
# Based on the UBI8 Minimal image

FROM registry.access.redhat.com/ubi8-minimal

# Upgrade and install Python 3
RUN microdnf upgrade && microdnf install python3

# Copy source code
COPY entrypoint.sh http_server.py /

# Expose the default httpd port 80
EXPOSE 8080

# Configure the container entrypoint
ENTRYPOINT ["/entrypoint.sh"]

# Run the httpd
CMD ["/usr/bin/python3", "-u", "/http_server.py"]
```

By looking at the source code of the Python web server that\'s been
executed by the container, we can see that the web server handler prints
a *Hello World!* string when an HTTP GET request is
received.[]{.sentence-end} The server also manages signal termination
using the Python `signal`{.inlineCode} module, allowing the container to
be stopped gracefully:

``` {.programlisting .snippet-code}
#!/usr/bin/python3
import http.server
import socketserver
import logging
import sys
import signal
from http import HTTPStatus

port = 8080
message = b'Hello World!\n'
logging.basicConfig(
  stream = sys.stdout,
  level = logging.INFO
)

def signal_handler(signum, frame):
  sys.exit(0)


class Handler(http.server.SimpleHTTPRequestHandler):
  def do_GET(self):
    self.send_response(HTTPStatus.OK)
    self.end_headers()
    self.wfile.write(message)

if __name__ == "__main__":
  signal.signal(signal.SIGTERM, signal_handler)
  signal.signal(signal.SIGINT, signal_handler)
  try:
    httpd = socketserver.TCPServer(('', port), Handler)
    logging.info("Serving on port %s", port)
    httpd.serve_forever()
  except SystemExit:
    httpd.shutdown()
    httpd.server_close()
```

Finally, the Python executable is called by a minimal entry point
script:

``` {.programlisting .snippet-con}
#!/bin/bash
set -e
exec $@
```

The script launches the []{#Chapter_8.xhtml#idx_71da92d3 .index-entry
index-entry="UBI Minimal image"}command that\'s passed by the array in
the `CMD`{.inlineCode} instruction.[]{.sentence-end} Also, notice the
`-u`{.inlineCode} option that\'s passed to the Python executable in the
command array.[]{.sentence-end} This enables unbuffered output and has
the container print access logs in real time.

Let\'s try to build and run the container to see what happens:

``` {.programlisting .snippet-con}
$ buildah build -t python_httpd .

$ podman run -p 8080:8080 python_httpd
INFO:root:Serving on port 8080
```

With that, our minimal Python `http`{.inlineCode} server is ready to
operate and serve a lot of barely useful but warming *Hello World!*
responses.[]{.sentence-end} You can find all these examples in the
GitHub repository of the book.

UBI Minimal works best for these kinds of use cases.[]{.sentence-end}
However, an even smaller image may be necessary.[]{.sentence-end} This
is the perfect use case for the UBI Micro image.

## The UBI Micro image {#Chapter_8.xhtml#h2_213 .heading-2}

The UBI Micro image is[]{#Chapter_8.xhtml#idx_9ab14788 .index-entry
index-entry="UBI Micro image"} the latest arrival to the UBI
family.[]{.sentence-end} Its basic idea is to provide a distroless image
and a stripped-down package manager, without all the unnecessary
packages, to provide a very small image that could also offer a minimal
attack surface.[]{.sentence-end} Reducing the attack surface is required
to achieve secure, minimal images that are more complex to
exploit.[]{.sentence-end} In addition to that, the small size also
minimizes the amount of data that must be downloaded when using the
image for the first time or after an update.

The UBI 8 Micro image []{#Chapter_8.xhtml#idx_7899d69e .index-entry
index-entry="UBI Micro image"}is great in multi-stage builds, where the
first stage creates the finished artifact(s) and the second stage copies
them inside the final image.[]{.sentence-end} The following example
shows a basic multi-stage Dockerfile/Containerfile where a minimal
Golang application is being built inside a UBI Standard container while
the final artifact is copied inside a UBI Micro image:

``` {.programlisting .snippet-code}
# Builder image
FROM registry.access.redhat.com/ubi8-minimal AS builder

# Install Golang packages
RUN microdnf upgrade && \
    microdnf install golang && \
    microdnf clean all

# Copy files for build
COPY go.mod /go/src/hello-world/
COPY main.go /go/src/hello-world/

# Set the working directory
WORKDIR /go/src/hello-world

# Download dependencies
RUN go get -d -v ./...

# Install the package
RUN go build -v ./...

# Runtime image
FROM registry.access.redhat.com/ubi8/ubi-micro:latest
COPY --from=builder /go/src/hello-world/hello-world /

EXPOSE 8080

CMD ["/hello-world"]
```

The build\'s []{#Chapter_8.xhtml#idx_189340fe .index-entry
index-entry="UBI Micro image"}output results in an image that\'s
approximately 45 MB in size.

The UBI Micro image has no built-in package manager, but it is still
possible to install additional packages using Buildah native
commands.[]{.sentence-end} This works effectively on an RHEL system,
where all Red Hat GPG certificates are installed.

The following example shows a build script that can be executed on RHEL
8.[]{.sentence-end} Its purpose is to install additional Python packages
using the host\'s `yum`{.inlineCode} package manager, on top of a UBI
Micro image:

``` {.programlisting .snippet-code}
#!/bin/bash

set -euo pipefail

if [ $UID -ne 0 ]; then
    echo "This script must be run as root"
    exit 1
fi

container=$(buildah from registry.access.redhat.com/ubi8/ubi-micro)
mount=$(buildah mount $container)

yum install -y \
  --installroot $mount \
  --setopt install_weak_deps=false \
  --nodocs \
  --noplugins \
  --releasever 8 \
  python3

yum clean all --installroot $mount

buildah umount $container
buildah commit $container micro_python3
```

Notice that the `yum install`{.inlineCode} command is executed by
passing the
`--`{.inlineCode}`installroot`{.inlineCode}` $mount`{.inlineCode}
option, which tells the installer to use the working container mount
point as the temporary root to install the packages.

UBI Minimal and UBI Micro images are great for implementing
microservices architectures where we []{#Chapter_8.xhtml#idx_589535ca
.index-entry index-entry="UBI Micro image"}need to orchestrate multiple
containers together, with each running a specific microservice.

Now, let\'s look at the UBI Init image, which allows us to coordinate
the execution of multiple services inside a container.

## The UBI Init image {#Chapter_8.xhtml#h2_214 .heading-2}

A common pattern in[]{#Chapter_8.xhtml#idx_da411fb0 .index-entry
index-entry="UBI Init image"} container development is to create highly
specialized images with a single component running inside them.

To implement multi-tier applications, such as those with a frontend,
middleware, and a backend, the best practice is to create and
orchestrate multiple containers, each one running a specific
component.[]{.sentence-end} The goal is to have minimal and very
specialized containers, each one running its own service/process while
following[]{#Chapter_8.xhtml#idx_c6b3d1c4 .index-entry
index-entry="Keep It Simple, Stupid (KISS)"} the **Keep It Simple,
Stupid** (**KISS**) philosophy, which has been carried out in UNIX
systems since their inception.

Despite being great for most use cases, this approach does not always
suit some special scenarios where many processes need to be orchestrated
together.[]{.sentence-end} An example is when we need to share all the
container namespaces across processes, or when we just want a single,
*uber* image.

Container images are normally created without an init system, and the
process that\'s executed inside the container (invoked by the
`CMD`{.inlineCode} instruction) usually []{#Chapter_8.xhtml#idx_e34ad062
.index-entry index-entry="PID 1"}gets **PID** **1**.

For this reason, Red Hat introduced the UBI Init image, which runs a
minimal **systemd** init process inside the container, allowing multiple
systemd[]{#Chapter_8.xhtml#idx_da1de8bc .index-entry
index-entry="systemd"} units that are governed by the systemd process
with a PID of `1`{.inlineCode} to be executed.

The UBI Init image is slightly smaller than the Standard image but has
more packages available than the Minimal image.

The default CMD is set to
`/`{.inlineCode}`sbin`{.inlineCode}`/`{.inlineCode}`init`{.inlineCode},
which corresponds to the systemd process.[]{.sentence-end} systemd
ignores the `SIGTERM`{.inlineCode} and `SIGKILL`{.inlineCode} signals,
which are used by Podman to stop running containers.[]{.sentence-end}
For this reason, the image is configured to send
`SIGRTMIN+3`{.inlineCode} signals for termination by passing the
`STOPSIGNAL`{.inlineCode}` `{.inlineCode}`SIGRTMIN+3`{.inlineCode}
instruction inside the image Dockerfile.

The following example shows a Dockerfile/Containerfile that installs the
`httpd`{.inlineCode} package and configures a `systemd`{.inlineCode}
unit[]{#Chapter_8.xhtml#idx_c9696057 .index-entry
index-entry="UBI Init image"} to run the `httpd`{.inlineCode} service:

``` {.programlisting .snippet-code}
FROM registry.access.redhat.com/ubi8/ubi-init

RUN yum -y install httpd && \
         yum clean all && \
         systemctl enable httpd

RUN echo "Successful Web Server Test" > /var/www/html/index.html

RUN mkdir /etc/systemd/system/httpd.service.d/ && \
         echo -e '[Service]\nRestart=always' > /etc/systemd/system/httpd.service.d/httpd.conf

EXPOSE 80
CMD [ "/sbin/init" ]
```

Notice the `RUN`{.inlineCode} instruction, where we create the
`/`{.inlineCode}`etc`{.inlineCode}`/`{.inlineCode}`systemd`{.inlineCode}`/system/`{.inlineCode}`httpd.service.d`{.inlineCode}`/`{.inlineCode}
folder and the systemd unit file.[]{.sentence-end} This minimal example
could be replaced with a copy of pre-edited unit files, which is
particularly useful when multiple services must be created.

We can build and run the image and inspect the behavior of the
`init`{.inlineCode} system inside the container using the
`ps`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ buildah build -t init_httpd .
$ podman run -d --name httpd_init -p 8080:80 init_httpd
$ podman exec -ti httpd_init /bin/bash
[root@b4fb727f1907 /]# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.1  0.0  89844  9404 ?        Ss   10:30   0:00 /sbin/init
root          10  0.0  0.0  95552 10636 ?        Ss   10:30   0:00 /usr/lib/systemd/systemd-journald
root          20  0.1  0.0 258068 10700 ?        Ss   10:30   0:00 /usr/sbin/httpd -DFOREGROUND
dbus          21  0.0  0.0  54056  4856 ?        Ss   10:30   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
apache        23  0.0  0.0 260652  7884 ?        S    10:30   0:00 /usr/sbin/httpd -DFOREGROUND
apache        24  0.0  0.0 2760308 9512 ?        Sl   10:30   0:00 /usr/sbin/httpd -DFOREGROUND
apache        25  0.0  0.0 2563636 9748 ?        Sl   10:30   0:00 /usr/sbin/httpd -DFOREGROUND
apache        26  0.0  0.0 2563636 9516 ?        Sl   10:30   0:00 /usr/sbin/httpd -DFOREGROUND
root         238  0.0  0.0  19240  3564 pts/0    Ss   10:30   0:00 /bin/bash
root         247  0.0  0.0  51864  3728 pts/0    R+   10:30   0:00 ps aux
```

Note that the
`/`{.inlineCode}`sbin`{.inlineCode}`/`{.inlineCode}`init`{.inlineCode}
process is executed with a PID of `1`{.inlineCode} and that it spawns
the `httpd`{.inlineCode} processes.[]{.sentence-end} The container also
executed `dbus`{.inlineCode}`-daemon`{.inlineCode}, which is used by
systemd to expose its API, along with
`systemd`{.inlineCode}`-journald`{.inlineCode} to handle logs.

Following this approach, we []{#Chapter_8.xhtml#idx_e1916f51
.index-entry index-entry="UBI Init image"}can add multiple services that
are supposed to work together in the same container and have them
orchestrated by systemd.

So far, we have looked at the four currently available UBI images and
demonstrated how they can be used to create custom
applications.[]{.sentence-end} Many public Red Hat images are based on
UBI.[]{.sentence-end} Let\'s take a look.

## Other UBI-based images {#Chapter_8.xhtml#h2_215 .heading-2}

Red Hat uses UBI[]{#Chapter_8.xhtml#idx_7a754a36 .index-entry
index-entry="UBI-based images"} images to produce many pre-built
specialized images, especially for runtimes.[]{.sentence-end} They are
usually expected not to have redistribution limitations.

This allows runtime images to be created for languages, runtimes, and
frameworks such as Python, Quarkus, Golang, Perl, PDP, .NET, Node.js,
Ruby, and OpenJDK.

UBI is also used as the base image for[]{#Chapter_8.xhtml#idx_cee5390c
.index-entry index-entry="Source-to-Image (S2I) framework"} the
**Source-to-Image** (**S2I**) framework, which is used to build
applications natively in OpenShift without the use of
Dockerfiles.[]{.sentence-end} With S2I, it is possible to assemble
images from user-defined custom scripts and, obviously, application
source code.

Last but not least, Red Hat\'s supported container releases of Buildah,
Podman, and Skopeo are packaged using UBI 8 images.

Moving beyond Red Hat\'s offering, other vendors use UBI images to
release their images too -- Intel, IBM, Isovalent, Cisco, Aqua Security,
and many others adopt UBI as the base for their official images on the
Red Hat Marketplace.

# Summary {#Chapter_8.xhtml#h1_216 .heading-1}

In this chapter, we learned about the OCI Image Specification and the
role of container registries.[]{.sentence-end} After that, we learned
how to adopt secure image registries and how to filter out those
registries using custom policies that allow us to block specific
registries, namespaces, or images.[]{.sentence-end} Finally, we
introduced UBI as a solution to create lightweight, reliable, and
redistributable images based on RHEL packages.

With the knowledge you\'ve gained in this chapter, you should be able to
understand the OCI Image Specification in more detail and manage image
registries securely.

In the next chapter, we will explore the difference between private and
public registries and how to create a private registry
locally.[]{.sentence-end} Finally, we will learn how to manage container
images with the specialized **Skopeo** tool.

# Further reading {#Chapter_8.xhtml#h1_217 .heading-1}

To learn more about the topics that were covered in this chapter, take a
look at the following resources:

-   \[1\] MITRE ATT&CK® matrix:
    [[https://attack.mitre.org/matrices/enterprise/containers/]{.url}](https://attack.mitre.org/matrices/enterprise/containers/){style="text-decoration: none;"}
-   \[2\] Things you should know about the Kubernetes threat matrix:
    [[https://cloud.redhat.com/blog/2021-kubernetes-threat-matrix-updates-things-you-should-know]{.url}](https://cloud.redhat.com/blog/2021-kubernetes-threat-matrix-updates-things-you-should-know){style="text-decoration: none;"}
-   \[3\] How to manage Linux container registries:
    [[https://www.redhat.com/sysadmin/manage-container-registries]{.url}](https://www.redhat.com/sysadmin/manage-container-registries){style="text-decoration: none;"}
-   \[4\] (Re)Introducing the Red Hat Universal Base Image:
    [[https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image]{.url}](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image){style="text-decoration: none;"}
-   \[5\] Introduction to Red Hat\'s UBI Micro:
    [[https://www.redhat.com/en/blog/introduction-ubi-micro]{.url}](https://www.redhat.com/en/blog/introduction-ubi-micro){style="text-decoration: none;"}

# Join us on Discord {#Chapter_8.xhtml#h1_218 .heading-1}

For discussions around the book and to connect with your peers, join us
on Discord at
[[packt.link/discordcloud]{.url}](https://packt.link/discordcloud){style="text-decoration: none;"}
or scan the QR code below:

![Image](images/B31467_8_7.png){style="width:25%;"}


[]{#Chapter_9.xhtml}

 {.section .chapter}
