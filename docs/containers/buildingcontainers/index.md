# 

# Meet Buildah -- Building Containers from Scratch 

In this chapter, we\'re going to cover the following main topics:

-   Basic image building with Podman
-   Meet Buildah, Podman\'s companion
-   Preparing our environment
-   Choosing our build strategy
-   Building images from scratch
-   Building images from a Dockerfile

# Technical requirements {#Chapter_6.xhtml#h1_165 .heading-1}

Before proceeding with this chapter, a machine with a working Podman
installation is required.[]{.sentence-end} As stated in *[Chapter
3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the First Container*,
all the examples in the book are executed on a Fedora 40 system or
later, but can be reproduced on your choice of OS.

A good understanding of the topics covered in *[Chapter
4](#Chapter_4.xhtml#h1_114){.chapref}*, *Managing* *Running Containers*,
is useful to easily grasp concepts regarding **Open Container
Initiative** (**OCI**) images.

# Building basic images with Podman {#Chapter_6.xhtml#h1_166 .heading-1}

A container\'s OCI image []{#Chapter_6.xhtml#idx_65ae9423 .index-entry
index-entry="basic images:building, with Podman"}is a set of immutable
layers stacked together with[]{#Chapter_6.xhtml#idx_7aa487b1
.index-entry index-entry="Podman:basic images, creating"} a
copy-on-write logic.[]{.sentence-end} When an image is built, all the
layers are created in a precise order and then pushed to the container
registry, which stores our layers as tar-based archives along with
additional image metadata.

As we learned in the *OCI* *images* section of *[Chapter
2](#Chapter_2.xhtml#h1_52){.chapref}*, *Comparing Podman and* *Docker*,
these manifests are necessary to correctly reassemble the image layers
(the image manifest and the image index) and to pass runtime
configurations to the container engine (the image configuration).

Before proceeding with the basic examples of image builds with Podman,
we need to understand how image builds generally work to grasp the
simple but very smart key concepts that lie beneath.

## Understanding builds under the hood {#Chapter_6.xhtml#h2_167 .heading-2}

Container images can be built in different ways, but the most common
approach, probably one of the keys to the huge success of containers, is
based on Dockerfiles.

A **Dockerfile**, as the []{#Chapter_6.xhtml#idx_dc46081c .index-entry
index-entry="Dockerfile"}name suggests, is the main configuration file
for Docker builds and is a plain list of actions to be executed in the
build process.

Over time, Dockerfiles became a standard in OCI image builds and today
are adopted in many use cases.

 note
Important note

To standardize and remove the association with the brand,
**Containerfiles\'** naming was also created; they have the very
same[]{#Chapter_6.xhtml#idx_867a35f8 .index-entry
index-entry="Containerfile"} syntax as Dockerfiles and are supported
natively by Podman.[]{.sentence-end} In this book, we will use the two
terms *Dockerfile* and *Containerfile* interchangeably.


We will learn in detail about Dockerfile syntax in the next
subsection.[]{.sentence-end} For now, let\'s just focus on a concept --
a Dockerfile []{#Chapter_6.xhtml#idx_4f3b3a43 .index-entry
index-entry="Dockerfile"}is a set of build instructions that the build
tool executes sequentially.[]{.sentence-end} Let\'s look at this
example:

``` {.programlisting .snippet-code}
FROM docker.io/library/fedora
RUN dnf install -y httpd && dnf clean all -y
COPY index.html /var/www/html
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

This basic example of a Dockerfile holds only four instructions:

-   The `FROM`{.inlineCode} instruction,
    which[]{#Chapter_6.xhtml#idx_61e2382a .index-entry
    index-entry="Dockerfile:FROM instruction"} defines the initial base
    image used to build the container image
-   The `RUN`{.inlineCode} instruction, which
    []{#Chapter_6.xhtml#idx_3a7d4c37 .index-entry
    index-entry="Dockerfile:RUN instruction"}executes some actions
    during the build (in this example, installing packages with the
    `dnf`{.inlineCode} package manager)
-   The `COPY`{.inlineCode} instruction,
    which[]{#Chapter_6.xhtml#idx_428aa9e2 .index-entry
    index-entry="Dockerfile:COPY instruction"} copies files or
    directories from the build working directory to the image
-   The `CMD`{.inlineCode} instruction, which
    []{#Chapter_6.xhtml#idx_07d131eb .index-entry
    index-entry="Dockerfile:CMD instruction"}defines the command to be
    executed when the container starts

When the `RUN`{.inlineCode} and the `COPY`{.inlineCode} actions of the
example are executed, new layers that hold the changes are cached in
intermediate layers, represented by temporary images.[]{.sentence-end}
This is a native feature in Docker that has the advantage of reusing
cached layers on further builds when no changes are requested on a
specific layer.[]{.sentence-end} All the intermediate containers will
produce read-only layers merged by the overlay graph driver.

Users don\'t need to manually manage the cached layers -- the engine
automatically implements the necessary actions by creating the temporary
containers, executing the actions defined by the Dockerfile
instructions, and then committing.[]{.sentence-end} By repeating the
same logic for all the necessary instructions, Podman creates a new
image with additional layers on top of the ones of the base image.

It is possible to squash the image layers into a single one to avoid a
negative impact on the overlay\'s performance.[]{.sentence-end} Podman
offers the same features and lets you choose between caching
intermediate layers or not.

Not all Dockerfile[]{#Chapter_6.xhtml#idx_32ea1067 .index-entry
index-entry="Dockerfile"} instructions change the filesystem, and only
the ones that do will create a new image layer; all the other
instructions, such as the `CMD`{.inlineCode} instruction in the
preceding example, produce an empty layer with metadata only and no
changes in the overlay filesystem.

In general, the only instructions that create new layers by effectively
changing the filesystem are the `RUN`{.inlineCode}, `COPY`{.inlineCode},
and `ADD`{.inlineCode} instructions.[]{.sentence-end} All the other
instructions in a Dockerfile or Containerfile just
[]{#Chapter_6.xhtml#idx_6f22542b .index-entry
index-entry="Containerfile"}create temporary intermediate images and do
not impact the final image filesystem.

This is also a good reason to keep the number of Dockerfile
`RUN`{.inlineCode}, `COPY`{.inlineCode}, and `ADD`{.inlineCode}
instructions limited, since having images cluttered with too many layers
is not a good pattern and impacts the graph driver performances.

We can inspect an image\'s history and the actions that have been
applied to every layer.[]{.sentence-end} The following example shows an
excerpt of the output from the `podman inspect`{.inlineCode} command,
with the target image being a potential one created from the previous
sample Dockerfile:

``` {.programlisting .snippet-con}
$ podman inspect myhttpd
[...omitted output]
        "History": [
               {
                    "created": "2024-10-31T11:44:19Z",
                    "created_by": "LABEL maintainer=Clement Verna <cverna@fedoraproject.org>",
                    "comment": "buildkit.dockerfile.v0",
                    "empty_layer": true
               },
               {
                    "created": "2024-10-31T11:44:19Z",
                    "created_by": "ENV DISTTAG=f41container FGC=f41 FBR=f41",
                    "comment": "buildkit.dockerfile.v0",
                    "empty_layer": true
               },
               {
                    "created": "2024-10-31T11:44:19Z",
                    "created_by": "ADD fedora-20241031.tar / # buildkit",
                    "comment": "buildkit.dockerfile.v0"
               },
               {
                    "created": "2024-10-31T11:44:19Z",
                    "created_by": "CMD [\"/bin/bash\"]",
                    "comment": "buildkit.dockerfile.v0",
                    "empty_layer": true
               },
               {
                    "created": "2024-11-01T10:26:32.41866158Z",
                    "created_by": "/bin/sh -c dnf install -y httpd && dnf clean all -y ",
                    "comment": "FROM docker.io/library/fedora:latest"
               },
               {
                    "created": "2024-11-01T10:26:33.243300733Z",
                    "created_by": "/bin/sh -c #(nop) COPY file:846c7ee0fa97be3c73c8d2fb0dd482f31c2ec12d7e74ad2323e6c5bde9ef68ad in /var/www/html ",
                    "comment": "FROM 97ad0fd6a1a3"
               },
               {
                    "created": "2024-11-01T10:26:33.317359389Z",
                    "created_by": "/bin/sh -c #(nop) CMD ["/usr/sbin/httpd", "-DFOREGROUND"]",
                    "comment": "FROM f737afe7b188",
                    "empty_layer": true
               }

            {
                "created": "2021-04-01T17:59:37.09884046Z",
                "created_by": "/bin/sh -c #(nop)  LABEL maintainer=Clement Verna \u003ccverna@fedoraproject.org\u003e",
                "empty_layer": true
            },
            {
                "created": "2021-04-01T18:00:19.741002882Z",
                "created_by": "/bin/sh -c #(nop)  ENV DISTTAG=f34container FGC=f34 FBR=f34",
                "empty_layer": true
            },
            {
                "created": "2021-07-23T11:16:05.060688497Z",
                "created_by": "/bin/sh -c #(nop) ADD file:85d7f2d8e4f31d81b27b8e18dfc5687b5dabfaafdb2408a3059e120e4c15307b in / "
            },
            {
                "created": "2021-07-23T11:16:05.833115975Z",
                "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/bash\"]",
                "empty_layer": true
            },
            {
                "created": "2021-10-24T21:27:18.783034844Z",
                "created_by": "/bin/sh -c dnf install -y httpd \u0026\u0026 dnf clean all -y  ",
                "comment": "FROM docker.io/library/fedora:latest"
            },
            {
                "created": "2021-10-24T21:27:21.095937071Z",
                "created_by": "/bin/sh -c #(nop) COPY file:78c6e1dcd6f819581b54094fd38a3fd8f170a2cb768101e533c964e04aacab2e in /var/www/html "
            },
            {
                "created": "2021-10-24T21:27:21.182063974Z",
                "created_by": "/bin/sh -c #(nop) CMD [\"/usr/sbin/httpd\", \"-DFOREGROUND\"]",
                "empty_layer": true
            }
        ]
[...omitted output]
```

Looking at the last three items of the image history, we can note the
exact instructions defined in the Dockerfile, including
the[]{#Chapter_6.xhtml#idx_d52157e3 .index-entry
index-entry="Dockerfile"} last `CMD`{.inlineCode} instruction, which
does not create any new layer but, instead, metadata that will persist
in the image config.

With this deeper awareness of the image build logic in mind, let\'s now
explore the most common Dockerfile instructions before proceeding with
the Podman build examples.

## Exploring Dockerfile and Containerfile instructions {#Chapter_6.xhtml#h2_168 .heading-2}

As stated before, Dockerfiles and Containerfiles share the same
syntax.[]{.sentence-end} The instructions in those files should be seen
as (and truly are) commands passed to the container engine or build
tool.[]{.sentence-end} This subsection provides an overview of the most
frequently used instructions.

All Dockerfile/Containerfile[]{#Chapter_6.xhtml#idx_54a58093
.index-entry index-entry="Dockerfile:instructions"} instructions follow
[]{#Chapter_6.xhtml#idx_b3a68743 .index-entry
index-entry="Containerfile:instructions"}the same pattern:

``` {.programlisting .snippet-code}
# Comment
INSTRUCTION arguments
```

The following list provides a non-exhaustive list of the most common
instructions:

-   `FROM`{.inlineCode}: This is the first instruction of a build stage
    and defines the base image used as the starting point of the
    build.[]{.sentence-end} It follows the
    `FROM `{.inlineCode}`<`{.inlineCode}`image`{.inlineCode}`>`{.inlineCode}`[:`{.inlineCode}`<`{.inlineCode}`tag`{.inlineCode}`>`{.inlineCode}`]`{.inlineCode}
    syntax to identify the correct image to use.

-   `RUN`{.inlineCode}: This instruction tells the engine to execute the
    commands passed as arguments inside a temporary
    container.[]{.sentence-end} This temporary container uses the
    filesystem of the image you\'re building.[]{.sentence-end} It
    follows the
    `RUN `{.inlineCode}`<`{.inlineCode}`command`{.inlineCode}`>`{.inlineCode}
    syntax.[]{.sentence-end} The invoked binary or script must exist in
    the base image or a previous layer.[]{.sentence-end}

    As stated before, the `RUN`{.inlineCode} instruction creates a new
    image layer; therefore, it is a frequent practice to concatenate
    commands into the same `RUN`{.inlineCode} instruction to avoid
    cluttering too many layers.

    This example[]{#Chapter_6.xhtml#idx_00e91d2d .index-entry
    index-entry="Dockerfile:instructions"} compacts three commands
    inside the same `RUN`{.inlineCode} instruction:

    ``` {.programlisting .snippet-code-one}
    RUN dnf upgrade -y && \
         dnf install httpd -y && \
         dnf clean all -y
    ```

-   `COPY`{.inlineCode}: This instruction
    copies[]{#Chapter_6.xhtml#idx_c3857398 .index-entry
    index-entry="Containerfile:instructions"} files and folders from the
    build working directory to the build sandbox.[]{.sentence-end}
    Copied resources are persisted in the final image.[]{.sentence-end}
    It follows the
    `COPY `{.inlineCode}`<`{.inlineCode}`src`{.inlineCode}`>`{.inlineCode}`… `{.inlineCode}`<`{.inlineCode}`dest`{.inlineCode}`>`{.inlineCode}
    syntax, and it has a very useful option that lets us define the
    destination user and group instead of manually changing ownership
    later --
    `--chown=`{.inlineCode}`<`{.inlineCode}`user`{.inlineCode}`>`{.inlineCode}`:`{.inlineCode}`<`{.inlineCode}`group`{.inlineCode}`>`{.inlineCode}.

-   `ADD`{.inlineCode}: This instruction copies files, folders, and
    remote URLs to the build destination target.[]{.sentence-end} It
    follows the
    `ADD `{.inlineCode}`<`{.inlineCode}`src`{.inlineCode}`>`{.inlineCode}`… `{.inlineCode}`<`{.inlineCode}`dest`{.inlineCode}`>`{.inlineCode}
    syntax.[]{.sentence-end} This instruction also supports the
    automatic extraction of tar files from a source directly into the
    target path.

-   `ENTRYPOINT`{.inlineCode}: The executed command in the
    container.[]{.sentence-end} It receives arguments from the command
    line (in the form of
    `podman run `{.inlineCode}`<`{.inlineCode}`image`{.inlineCode}`>`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`arguments`{.inlineCode}`>`{.inlineCode})
    or from the `CMD`{.inlineCode} instruction.[]{.sentence-end}

    An `ENTRYPOINT`{.inlineCode} image cannot be overridden from
    command-line arguments.[]{.sentence-end} The supported forms are the
    following:

    -   `ENTRYPOINT [`{.inlineCode}`"`{.inlineCode}`command`{.inlineCode}`"`{.inlineCode}`, `{.inlineCode}`"`{.inlineCode}`param1`{.inlineCode}`"`{.inlineCode}`, `{.inlineCode}`"`{.inlineCode}`paramN`{.inlineCode}`"`{.inlineCode}`]`{.inlineCode}
        (also known as the *exec* form)
    -   `ENTRYPOINT command param1 paramN`{.inlineCode} (the *shell*
        form)
    -   If not set, the default value for `ENTRYPOINT`{.inlineCode} is
        `bash -c`{.inlineCode}.[]{.sentence-end} When set to the default
        value, commands are passed as an argument to the
        `bash`{.inlineCode} process.[]{.sentence-end} For example, if a
        `ps aux`{.inlineCode} command is passed as an argument at
        runtime or in a `CMD`{.inlineCode} instruction, the container
        will execute `bash -c "ps aux"`{.inlineCode}.

    A frequent practice is to replace the default
    `ENTRYPOINT`{.inlineCode} command with a custom *script* that
    behaves in the same way and offers more granular control of the
    runtime execution.

-   `CMD`{.inlineCode}: The default[]{#Chapter_6.xhtml#idx_affa74c1
    .index-entry index-entry="Dockerfile:instructions"} argument(s)
    passed to the `ENTRYPOINT`{.inlineCode}
    instruction.[]{.sentence-end} It[]{#Chapter_6.xhtml#idx_2345c4d4
    .index-entry index-entry="Containerfile:instructions"} can be a full
    command or a set of plain arguments to be passed to a custom script
    or binary set as `ENTRYPOINT`{.inlineCode}.[]{.sentence-end} The
    supported forms are the following:
    -   `CMD [`{.inlineCode}`"`{.inlineCode}`command`{.inlineCode}`"`{.inlineCode}`, `{.inlineCode}`"`{.inlineCode}`param1`{.inlineCode}`"`{.inlineCode}`, `{.inlineCode}`"`{.inlineCode}`paramN`{.inlineCode}`"`{.inlineCode}`]`{.inlineCode}
        (the *exec* form)
    -   `CMD [`{.inlineCode}`"`{.inlineCode}`param1, `{.inlineCode}`"`{.inlineCode}`paramN`{.inlineCode}`"`{.inlineCode}`]`{.inlineCode}
        (the *parameter* form, used to pass arguments to a custom
        `ENTRYPOINT`{.inlineCode})
    -   `CMD command param1 paramN`{.inlineCode} (the *shell* form)

-   `LABEL`{.inlineCode}: This instruction is used to apply custom
    labels to the image.[]{.sentence-end} Labels are used as metadata at
    build time or runtime.[]{.sentence-end} It follows the
    `LABEL `{.inlineCode}`<`{.inlineCode}`key1`{.inlineCode}`>`{.inlineCode}`=`{.inlineCode}`<`{.inlineCode}`value1`{.inlineCode}`>`{.inlineCode}` … `{.inlineCode}`<`{.inlineCode}`keyN`{.inlineCode}`>`{.inlineCode}`=`{.inlineCode}`<`{.inlineCode}`valueN`{.inlineCode}`>`{.inlineCode}
    syntax.

-   `EXPOSE`{.inlineCode}: This sets metadata about listening ports
    exposed by the processes running in the container.[]{.sentence-end}
    It supports the
    `EXPOSE `{.inlineCode}`<`{.inlineCode}`port`{.inlineCode}`>`{.inlineCode}`/`{.inlineCode}`<`{.inlineCode}`protocol`{.inlineCode}`>`{.inlineCode}
    format.[]{.sentence-end} These ports are not forwarded by default
    when running a container based on the image and require explicit
    user action.

-   `ENV`{.inlineCode}: This configures environment variables that will
    be available to the next build commands and at runtime when the
    container is executed.[]{.sentence-end} This instruction supports
    the
    `ENV `{.inlineCode}`<`{.inlineCode}`key1`{.inlineCode}`>`{.inlineCode}`=`{.inlineCode}`<`{.inlineCode}`value1`{.inlineCode}`>`{.inlineCode}`… `{.inlineCode}`<`{.inlineCode}`keyN`{.inlineCode}`>`{.inlineCode}`=`{.inlineCode}`<`{.inlineCode}`valueN`{.inlineCode}`>`{.inlineCode}
    format.[]{.sentence-end}

    Environment variables can also be set inside a `RUN`{.inlineCode}
    instruction with a scope limited to the instruction itself.

-   `VOLUME`{.inlineCode}: This sets a volume that will be created at
    runtime during container execution.[]{.sentence-end} The volume will
    be automatically mapped by Podman inside the default volume storage
    directory.[]{.sentence-end} The supported formats are the following:

    -   `VOLUME [`{.inlineCode}`"`{.inlineCode}`/path/to/dir`{.inlineCode}`"`{.inlineCode}`]`{.inlineCode}
    -   `VOLUME /path/to/dir`{.inlineCode}

    See also the *Attaching host storage to a container* section in
    *[Chapter 5](#Chapter_5.xhtml#h1_133){.chapref}*, *Implementing
    Storage for the Containers\'* *Data*, for more details about
    volumes.

-   `USER`{.inlineCode}: This instruction defines the username and user
    group for the next `RUN`{.inlineCode}, `CMD`{.inlineCode}, and
    `ENTRYPOINT`{.inlineCode} instructions when you run the
    image.[]{.sentence-end} The `GID`{.inlineCode} value is not
    mandatory.[]{.sentence-end}

    The supported formats are the following:

    -   `USER `{.inlineCode}`<`{.inlineCode}`username`{.inlineCode}`>`{.inlineCode}`:[`{.inlineCode}`<`{.inlineCode}`groupname`{.inlineCode}`>`{.inlineCode}`]`{.inlineCode}
    -   `USER `{.inlineCode}`<`{.inlineCode}`UID`{.inlineCode}`>`{.inlineCode}`:[`{.inlineCode}`<`{.inlineCode}`GID`{.inlineCode}`>`{.inlineCode}`]`{.inlineCode}

-   `WORKDIR`{.inlineCode}: This[]{#Chapter_6.xhtml#idx_fe0a59fa
    .index-entry index-entry="Dockerfile:instructions"} sets the working
    directory during the build process.[]{.sentence-end} This value is
    retained during container execution.[]{.sentence-end} It supports
    the `WORKDIR /path/to/workdir`{.inlineCode} format.

-   `ONBUILD`{.inlineCode}: This[]{#Chapter_6.xhtml#idx_db121a89
    .index-entry index-entry="Containerfile:instructions"} instruction
    defines a trigger command to be executed once an image build has
    been completed.[]{.sentence-end} In this way, the image can be used
    as a parent for a new build by calling it with the
    `FROM`{.inlineCode} instruction.[]{.sentence-end} Its purpose is to
    allow the execution of some final command on a child container
    image.[]{.sentence-end}

    The supported formats are the following:

    -   `ONBUILD `{.inlineCode}`ADD .`{.inlineCode}` /opt/app`{.inlineCode}
    -   `ONBUILD RUN /opt/bin/custom-build /opt/app/src`{.inlineCode}

Now that we have learned the most common instructions, let\'s dive into
our first build examples with Podman.

## Running builds with Podman {#Chapter_6.xhtml#h2_169 .heading-2}

Good news -- Podman provides[]{#Chapter_6.xhtml#idx_f06c12dc
.index-entry index-entry="builds:running, with Podman"} the same build
commands and syntax as Docker.[]{.sentence-end}
If[]{#Chapter_6.xhtml#idx_ebb5f984 .index-entry
index-entry="Podman:builds, running"} you are switching from Docker,
there will be no learning curve to start building your images with
it.[]{.sentence-end} Under the hood, there is a notable advantage in
choosing Podman as a build tool -- Podman can build containers in
rootless mode, using a fork/exec model.

This is a step forward compared to Docker builds, where communication
with the daemon listening on the Unix socket is necessary to run the
build.

Let\'s start by running a simple build based on the `httpd`{.inlineCode}
Dockerfile illustrated in the first subsection, *Builds under the
hood*.[]{.sentence-end} We will use the following
`podman build`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ podman build -t myhttpd .
STEP 1/4: FROM docker.io/library/fedora
STEP 2/4: RUN dnf install -y httpd && dnf clean all -y
[...omitted output]
--> 50a981094eb
STEP 3/4: COPY index.html /var/www/html
--> 73f8702c5e0
STEP 4/4: CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
COMMIT myhttpd
--> e773bfee6f2
Successfully tagged localhost/myhttpd:latest
e773bfee6f289012b37285a9e559bc44962de3aeed001455231b5a8f2721b8f9
```

In the preceding example, the output of the `dnf install`{.inlineCode}
command was omitted for the sake of clarity and space.

The command runs the instructions sequentially and persists the
intermediate layers until the final image is committed and
tagged.[]{.sentence-end} These layers remain available as cache until
they\'re manually cleaned up with `podman image prune`{.inlineCode};
otherwise, we wouldn\'t have them available if the image was rebuilt.

The build steps are numbered (`1/4`{.inlineCode} to `4/4`{.inlineCode}),
and some of them (`RUN`{.inlineCode} and `COPY`{.inlineCode} here)
produce non-empty layers, forming part of the `lowerDirs`{.inlineCode}
image.

The first `FROM`{.inlineCode} instruction defines the base image, which
is pulled automatically if not present in the host.

The second instruction is `RUN`{.inlineCode}, which executes the
`dnf`{.inlineCode} command to install the `httpd`{.inlineCode} package
and clean up the system upon completion.[]{.sentence-end} Under the
hood, this line is executed as
`"bash -`{.inlineCode}`c `{.inlineCode}`'`{.inlineCode}`dnf ins`{.inlineCode}`tall -y httpd && dnf clean all -`{.inlineCode}`y'"`{.inlineCode}.

The third `COPY`{.inlineCode} instruction simply copies the
`index.html`{.inlineCode} file in the default `httpd`{.inlineCode}
document root.

Finally, the fourth[]{#Chapter_6.xhtml#idx_6c1812e5 .index-entry
index-entry="builds:running, with Podman"} step defines the default
container `CMD`{.inlineCode} instruction.[]{.sentence-end}
Since[]{#Chapter_6.xhtml#idx_7b9734dc .index-entry
index-entry="Podman:builds, running"} no `ENTRYPOINT`{.inlineCode}
instructions were set, this will translate into the following command:

``` {.programlisting .snippet-con}
"bash -c '/usr/sbin/httpd -DFOREGROUND'".
```

The next example is a custom Dockerfile/Containerfile where a custom web
server is built:

``` {.programlisting .snippet-code}
FROM docker.io/library/fedora
# Install required packages
RUN set -euo pipefail; \
    dnf upgrade -y; \
    dnf install httpd -y; \
    dnf clean all -y; \
    rm -rf /var/cache/dnf/*
# Custom webserver configs for rootless execution
RUN set -euo pipefail; \
    sed -i 's|Listen 80|Listen 8080|' \
           /etc/httpd/conf/httpd.conf; \
    sed -i 's|ErrorLog "logs/error_log"|ErrorLog /dev/stderr|' \
           /etc/httpd/conf/httpd.conf; \
    sed -i 's|CustomLog "logs/access_log" combined|CustomLog /dev/stdout combined|' \
           /etc/httpd/conf/httpd.conf; \
    chown 1001 /var/run/httpd

                   
# Copy web content
COPY index.html /var/www/html
# Define content volume
VOLUME /var/www/html
# Copy container entrypoint.sh script
COPY entrypoint.sh /entrypoint.sh
# Declare exposed ports
EXPOSE 8080
# Declare default user
USER 1001
ENTRYPOINT ["/entrypoint.sh"]
CMD ["httpd"]
```

This example was []{#Chapter_6.xhtml#idx_055195cb .index-entry
index-entry="builds:running, with Podman"}designed for the purpose of
this book to illustrate some []{#Chapter_6.xhtml#idx_6e01cc29
.index-entry index-entry="Podman:builds, running"}peculiar elements:

-   Packages installed with a package manager should be kept to a
    minimum.[]{.sentence-end} After installing the `httpd`{.inlineCode}
    package, necessary to run the web server, the cache is cleaned to
    reduce the size of the image.
-   Multiple commands can be grouped together in a single
    `RUN`{.inlineCode} instruction.[]{.sentence-end} However, we don\'t
    want to continue the build if a single command
    fails.[]{.sentence-end} To provide a failsafe shell execution, the
    `set -euo pipefail`{.inlineCode} command was
    prepended.[]{.sentence-end} Also, to improve readability, the single
    commands were split into more lines using the `\`{.inlineCode}
    character, which can work as a line break or escape character.
-   To avoid running the isolated processes as the root user, a series
    of workarounds were implemented in order to have the
    `httpd`{.inlineCode} process running as the generic
    `1001`{.inlineCode} user.[]{.sentence-end} Those workarounds
    included updating file permissions and group ownership on specific
    directories that are expected to be accessed by non-root
    users.[]{.sentence-end} This is a security best practice that
    reduces the attack surface of the container.[]{.sentence-end} Of
    course, this is unnecessary with rootless containers.
-   A common pattern in containers is the redirection of application
    logs to the container\'s `stdout`{.inlineCode} and
    `stderr`{.inlineCode}.[]{.sentence-end} The common
    `httpd`{.inlineCode} log streams have been modified for this purpose
    using regular expressions against the
    `/etc/httpd/conf/httpd.conf`{.inlineCode} file.
-   The web server ports are declared as exposed with the
    `EXPOSE`{.inlineCode} instruction.
-   The `CMD`{.inlineCode} instruction is a simple `httpd`{.inlineCode}
    command without any other arguments.[]{.sentence-end} This was done
    to illustrate how `ENTRYPOINT`{.inlineCode} can interact with the
    `CMD`{.inlineCode} arguments.

The container `ENTRYPOINT`{.inlineCode}
instruction[]{#Chapter_6.xhtml#idx_eda88e4b .index-entry
index-entry="builds:running, with Podman"} is modified with a custom
script that brings more []{#Chapter_6.xhtml#idx_1bf43fef .index-entry
index-entry="Podman:builds, running"}flexibility to the way the
`CMD`{.inlineCode} instruction is managed.[]{.sentence-end} The
`entrypoint.sh`{.inlineCode} file tests whether the container is
executed as root and checks the first `CMD`{.inlineCode} argument -- if
the argument is `httpd`{.inlineCode}, it executes the
`httpd -DFOREGROUND`{.inlineCode} command; otherwise, it lets you
execute any other command (a shell, for example).[]{.sentence-end} The
following code is the content of the `entrypoint.sh`{.inlineCode}
script:

``` {.programlisting .snippet-code}
#!/bin/sh
set -euo pipefail
if [ $UID != 0 ]; then
    echo "Running as user $UID"
fi
if [ "$1" == "httpd" ]; then
    echo "Starting custom httpd server"
    exec $1 -DFOREGROUND
else
    echo "Starting container with custom arguments"
    exec "$@"
fi
```

Let\'s now build the image with the `podman build`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ podman build -t myhttpd .
```

The newly built image will be available in the local host cache:

``` {.programlisting .snippet-con}
$ podman images | grep myhttpd
localhost/myhttpd latest 6dc90348520c 2 minutes ago   248 MB
```

After building, we can *tag* the image with the target registry
name.[]{.sentence-end} The following example tags the image applying the
`v1.0`{.inlineCode} tag and the `latest`{.inlineCode} tag:

``` {.programlisting .snippet-con}
$ podman tag localhost/myhttpd quay.io/<username>/myhttpd:v1.0
```

After tagging, the image []{#Chapter_6.xhtml#idx_7964972b .index-entry
index-entry="builds:running, with Podman"}will be ready to be pushed to
the remote registry.[]{.sentence-end} We will cover
[]{#Chapter_6.xhtml#idx_9da21100 .index-entry
index-entry="Podman:builds, running"}the interaction with registries in
greater detail in *[Chapter 9](#Chapter_9.xhtml#h1_219){.chapref}*,
*Pushing Images to a Container Registry*.

The example image will be composed of five layers, including the base
Fedora image layer.[]{.sentence-end} We can verify the number of layers
by running the `podman inspect`{.inlineCode} command against the new
image:

``` {.programlisting .snippet-con}
$ podman inspect myhttpd --format '{{ .RootFS.Layers }}'
[sha256:b6d0e02fe431db7d64d996f3dbf903153152a8f8b857cb4829ab3c4a3e484a72 sha256:f41274a78d9917b0412d99c8b698b0094aa0de74ec8995c88e5dbf1131494912 sha256:e57dde895085c50ea57db021bffce776ee33253b4b8cb0fe909bbbac45af0e8c sha256:9989ee85603f534e7648c74c75aaca5981186b787d26e0cae0bc7ee9eb54d40d sha256:ca402716d23bd39f52d040a39d3aee242bf235f626258958b889b40cdec88b43]
```

You can also have a look at the image layer with the dedicated
`podman image tree`{.inlineCode} subcommand:

``` {.programlisting .snippet-con}
$ podman image tree myhttpd                                                             main
Image ID: 4067fa24786a
Tags:     [localhost/myhttpd:latest]
Size:     317.6MB
Image Layers
├── ID: 38f7d1e80c77 Size: 186.2MB Top Layer of: [docker.io/library/fedora:latest]
├── ID: 3d6a999a163f Size: 131.3MB
├── ID: 6fc772abb506 Size: 15.87kB
├── ID: 778ad0afa90a Size: 3.584kB
└── ID: da726f9c0cf9 Size: 2.048kB Top Layer of: [localhost/myhttpd:latest]
```

It is possible to squash the current build layers into a single layer
using the `--layers=false`{.inlineCode} option.[]{.sentence-end} The
resulting image will have only two layers -- the base Fedora layer and
the squashed one.[]{.sentence-end} The following example rebuilds the
image without caching the intermediate layers:

``` {.programlisting .snippet-con}
$ podman build -t myhttpd --layers=false .
```

Let\'s inspect the output image again:

``` {.programlisting .snippet-con}
$ podman inspect myhttpd --format '{{ .RootFS.Layers }}'
[sha256:b6d0e02fe431db7d64d996f3dbf903153152a8f8b857cb4829ab3c4a3e484a72 sha256:6c279ab14837b30af9360bf337c7f9b967676a61831eee91012fa67083f5dcf1]
```

This time, the final []{#Chapter_6.xhtml#idx_b7cb460d .index-entry
index-entry="builds:running, with Podman"}image has the two expected
layers only.

Reducing the number of[]{#Chapter_6.xhtml#idx_ca5cece9 .index-entry
index-entry="Podman:builds, running"} layers can be useful to keep the
image minimal in terms of overlays.[]{.sentence-end} The downside of
this approach is that we will have to rebuild the whole image for every
configuration change without taking advantage of cached layers.

In terms of isolation, Podman can safely build images in rootless
mode.[]{.sentence-end} Indeed, this is considered a value since there
should be no need to run builds with a privileged user such as
root.[]{.sentence-end} If rootful builds are necessary, they are fully
functional and supported.[]{.sentence-end} The following example runs a
build as the root user (notice the hashtag symbol typical of the root
terminal session in Linux):

``` {.programlisting .snippet-con}
# podman build -t myhttpd .
```

The resulting image will[]{#Chapter_6.xhtml#idx_eeba7db8 .index-entry
index-entry="builds:running, with Podman"} be available only in the
system image cache and its layers stored
[]{#Chapter_6.xhtml#idx_643e13e4 .index-entry
index-entry="Podman:builds, running"}under
`/var/lib/containers/storage/`{.inlineCode}.

The flexible nature of Podman builds is strongly related to its
companion tool, **Buildah**, a specialized tool to build OCI images that
provides greater flexibility in builds.[]{.sentence-end} In the next
section, we will describe Buildah\'s features and how it manages image
builds.

# Introducing Buildah, Podman\'s companion {#Chapter_6.xhtml#h1_170 .heading-1}

Podman does an excellent job in plain builds with
Dockerfiles/Containerfiles and helps teams to preserve their previously
implemented build pipelines without the need for new investments.

However, when it comes to more specialized build tasks or when users
need more control over the build workflow, with the option of including
scripting logic, the Dockerfile/Containerfile approach shows its
limitations.[]{.sentence-end} Communities struggled to find alternative
building approaches that could overcome the rigid, workflow-based logic
of Dockerfiles/Containerfiles.

The same community that develops Podman brought to life the Buildah
(pronounced *build-ah*) project, a tool to manage OCI builds with
support for multiple building strategies.[]{.sentence-end} Images
created with Buildah are fully portable and compatible with Docker, and
all engines are compliant with the OCI image and runtime specs.

Buildah is an open source project released under the Apache 2.0
license.[]{.sentence-end} Sources are available on GitHub at the
following URL:
[[https://github.com/containers/buildah]{.url}](https://github.com/containers/buildah){style="text-decoration: none;"}.

Buildah is []{#Chapter_6.xhtml#idx_e2ba4d84 .index-entry
index-entry="Buildah"}complementary to Podman, which borrows its build
logic by vendoring its libraries to implement basic build
functionalities against Dockerfiles and Containerfiles.[]{.sentence-end}
The final Podman binary, which is compiled in Go as a statically linked
single file, embeds Buildah packages to manage the build steps.

Buildah uses the `containers/image`{.inlineCode} project
([[https://github.com/containers/image]{.url}](https://github.com/containers/image){style="text-decoration: none;"})
to manage an image\'s life cycle and its interaction with registries,
and the `containers/storage`{.inlineCode} project
([[https://github.com/containers/storage]{.url}](https://github.com/containers/storage){style="text-decoration: none;"})
to manage images and container filesystem layers.[]{.sentence-end} These
libraries are the same that Podman uses, and they have both now been
moved into the `containers/containers-libs`{.inlineCode} project
([[https://github.com/containers/container-libs]{.url}](https://github.com/containers/container-libs){style="text-decoration: none;"}).

The advanced build strategy of Buildah is based on the parallel support
for traditional Dockerfile/Containerfile-based builds, and for builds
driven by native Buildah commands that replicate the Dockerfile
instructions.

By replicating Dockerfile instructions in standard commands, Buildah
becomes a scriptable tool that can be interpolated with custom logic and
native shell constructs such as conditionals, loops, or environment
variables.[]{.sentence-end} For example, the `RUN`{.inlineCode}
instruction in a Dockerfile can be replaced with a
`buildah run`{.inlineCode} command.

If teams need to preserve the build logic implemented in previous
Dockerfiles, Buildah offers the `buildah build`{.inlineCode} (or its
alias, `buildah bud`{.inlineCode}) command, which builds the image
reading from the provided Dockerfile/Containerfile, as in a typical
`podman build`{.inlineCode} command.

Buildah can smoothly run in rootless mode to build images; this is a
valuable, highly demanded feature from a security point of
view.[]{.sentence-end} No Unix sockets are necessary to run a
build.[]{.sentence-end} At the beginning of this chapter, we explained
how builds are always based on containers; Buildah is not exempt from
this behavior, and all its builds are executed inside working
containers, starting on top of the base image.

The following list provides a non-exhaustive description of the most
frequently used commands in Buildah:

-   `buildah`{.inlineCode}` from`{.inlineCode}: Initializes a
    new[]{#Chapter_6.xhtml#idx_d143a421 .index-entry
    index-entry="Buildah:commands"} working container on top of a base
    image.[]{.sentence-end} It accepts the
    `buildah from [options] `{.inlineCode}`<`{.inlineCode}`image`{.inlineCode}`>`{.inlineCode}
    syntax.[]{.sentence-end} An example of this command is
    `$ buildah from fedora`{.inlineCode}.
-   `buildah`{.inlineCode}` run`{.inlineCode}: This is equivalent to the
    `RUN`{.inlineCode} instruction of a Dockerfile; it runs a command
    inside a working container.[]{.sentence-end} This command accepts
    the
    `buildah run [options] [--] `{.inlineCode}`<`{.inlineCode}`container`{.inlineCode}`>`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`command`{.inlineCode}`>`{.inlineCode}
    syntax.[]{.sentence-end} The `--`{.inlineCode} (double dash) option
    is necessary to separate potential options from the effective
    container command.[]{.sentence-end} An example of this command is
    `buildah run `{.inlineCode}`<`{.inlineCode}`containerID`{.inlineCode}`>`{.inlineCode}` -- dnf install -y nginx`{.inlineCode}.
-   `buildah`{.inlineCode}` config`{.inlineCode}: This command
    configures image metadata.[]{.sentence-end} It accepts the
    `buildah config [options] `{.inlineCode}`<`{.inlineCode}`container`{.inlineCode}`>`{.inlineCode}
    format.[]{.sentence-end} The options available for this command are
    associated with the various Dockerfile instructions that do not
    modify filesystem layers but set some container metadata -- for
    instance, the setup of the `entrypoint`{.inlineCode}
    container.[]{.sentence-end} An example of this command is
    `buildah config --entrypoint/entrypoint.sh `{.inlineCode}`<`{.inlineCode}`containerID`{.inlineCode}`>`{.inlineCode}.
-   `buildah`{.inlineCode}` add`{.inlineCode}: This is equivalent to the
    `ADD`{.inlineCode} instruction of the Dockerfile; it adds files,
    directories, and even URLs to the container.[]{.sentence-end} It
    supports the
    `buildah add [options] `{.inlineCode}`<`{.inlineCode}`container`{.inlineCode}`>`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`src`{.inlineCode}`>`{.inlineCode}` [[src …] `{.inlineCode}`<`{.inlineCode}`dst`{.inlineCode}`>`{.inlineCode}
    syntax and allows you to copy multiple files in one single
    command.[]{.sentence-end} An example of this command is
    `buildah add `{.inlineCode}`<`{.inlineCode}`containerID`{.inlineCode}`>`{.inlineCode}` index.php /var/www.html`{.inlineCode}.
-   `buildah`{.inlineCode}` copy`{.inlineCode}: This is the same as the
    Dockerfile `COPY`{.inlineCode} instruction; it adds files, URLs, and
    directories to the container.[]{.sentence-end} It supports the
    `buildah copy [options] `{.inlineCode}`<`{.inlineCode}`container`{.inlineCode}`>`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`src`{.inlineCode}`>`{.inlineCode}` [[src …] `{.inlineCode}`<`{.inlineCode}`dst`{.inlineCode}`>`{.inlineCode}
    syntax.[]{.sentence-end} An example of this command is
    `buildah copy `{.inlineCode}`<`{.inlineCode}`containerID`{.inlineCode}`>`{.inlineCode}` entrypoint.sh /`{.inlineCode}.
-   `buildah`{.inlineCode}` commit`{.inlineCode}: This
    []{#Chapter_6.xhtml#idx_b4681ee6 .index-entry
    index-entry="Buildah:commands"}commits a final image out of a
    working container.[]{.sentence-end} This command is usually the last
    executed one.[]{.sentence-end} It supports the
    `buildah copy [options] `{.inlineCode}`<`{.inlineCode}`container`{.inlineCode}`>`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`image_name`{.inlineCode}`>`{.inlineCode}
    syntax.[]{.sentence-end} The container image created from this
    command can be later tagged and pushed to a
    registry.[]{.sentence-end} An example of this command is
    `buildah commit `{.inlineCode}`<`{.inlineCode}`containerID`{.inlineCode}`>`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`myhttpd`{.inlineCode}`>`{.inlineCode}.
-   `buildah`{.inlineCode}` build`{.inlineCode}: The equivalent command
    of the classic `p`{.inlineCode}`odman build`{.inlineCode}
    command.[]{.sentence-end} This command takes Dockerfiles or
    Containerfiles as arguments, along with the build directory
    path.[]{.sentence-end} It accepts the
    `buildah build [options] [context]`{.inlineCode} syntax and the
    `buildah bud`{.inlineCode} command alias.[]{.sentence-end} An
    example of this command is
    `buildah build -`{.inlineCode}`t `{.inlineCode}`<`{.inlineCode}`imageName`{.inlineCode}`>`{.inlineCode}` .`{.inlineCode}.
-   `buildah`{.inlineCode}` containers`{.inlineCode}: This lists the
    active working container involved in Buildah builds, along with the
    base image used as a starting point.[]{.sentence-end} Equivalent
    commands are `buildah ls`{.inlineCode} and
    `buildah ps`{.inlineCode}.[]{.sentence-end} The supported syntax is
    `buildah containers [options]`{.inlineCode}.[]{.sentence-end} An
    example of this command is `buildah containers`{.inlineCode}.
-   `buildah`{.inlineCode}` rm`{.inlineCode}: This is used to remove
    working containers.[]{.sentence-end} The
    `buildah delete`{.inlineCode} command is
    equivalent.[]{.sentence-end} The supported syntax is
    `buildah rm `{.inlineCode}`<`{.inlineCode}`container`{.inlineCode}`>`{.inlineCode}.[]{.sentence-end}
    This command has only one option, the
    `-`{.inlineCode}`all, -`{.inlineCode}`a`{.inlineCode} option, to
    remove all the working containers.[]{.sentence-end} An example of
    this command is
    `buildah rm `{.inlineCode}`<`{.inlineCode}`containerID`{.inlineCode}`>`{.inlineCode}.
-   `buildah`{.inlineCode}` mount`{.inlineCode}: This command can be
    used to mount a working container root filesystem.[]{.sentence-end}
    The accepted syntax is
    `buildah mount [containerID `{.inlineCode}`… ]`{.inlineCode}.[]{.sentence-end}
    When no argument is passed, the command only shows the currently
    mounted containers.[]{.sentence-end} A practical example of this
    command used to mount a container filesystem is
    `buildah`{.inlineCode}` `{.inlineCode}`mount`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`containerID`{.inlineCode}`>`{.inlineCode}.[]{.sentence-end}
    This can be used to avoid `buildah add`{.inlineCode} and
    `buildah copy`{.inlineCode} and instead directly change the
    container filesystem yourself.
-   `buildah`{.inlineCode}` images`{.inlineCode}: This lists all the
    available images in the local host cache.[]{.sentence-end} The
    accepted syntax is
    `buildah images [options] [image]`{.inlineCode}.[]{.sentence-end}
    Custom output formats such as JSON are available.[]{.sentence-end}
    An example of this command is `buildah images --json`{.inlineCode}.
-   `buildah`{.inlineCode}` tag`{.inlineCode}: This applies a custom
    name and tags to an image in the local store.[]{.sentence-end} The
    syntax follows the
    `buildah tag `{.inlineCode}`<`{.inlineCode}`name`{.inlineCode}`>`{.inlineCode}` `{.inlineCode}`<`{.inlineCode}`new-name`{.inlineCode}`>`{.inlineCode}
    format.[]{.sentence-end} An example of this command is
    `buildah tag myapp quay.io/packt/myapp:latest`{.inlineCode}.
-   `buildah`{.inlineCode}` push`{.inlineCode}: This pushes a local
    image to a remote private or public register, or local directories
    in Docker or OCI format.[]{.sentence-end} The command syntax is
    `buildah push `{.inlineCode}`[options] `{.inlineCode}`<`{.inlineCode}`image`{.inlineCode}`>`{.inlineCode}` [destination]`{.inlineCode}.[]{.sentence-end}
    Examples of this command include
    `buildah push quay.io/packt/myapp:latest`{.inlineCode},` buildah push `{.inlineCode}`<`{.inlineCode}`imageID`{.inlineCode}`>`{.inlineCode}` docker://`{.inlineCode}`<`{.inlineCode}`URL`{.inlineCode}`>`{.inlineCode}`/repository:tag`{.inlineCode},
    and
    `buildah push `{.inlineCode}`<`{.inlineCode}`imageID`{.inlineCode}`>`{.inlineCode}` oci:`{.inlineCode}`<`{.inlineCode}`/path/to/dir`{.inlineCode}`>`{.inlineCode}`:image:tag`{.inlineCode}.
-   `buildah`{.inlineCode}` pull`{.inlineCode}:
    This[]{#Chapter_6.xhtml#idx_66dc4fea .index-entry
    index-entry="Buildah:commands"} pulls an image from a registry, an
    OCI archive, or a directory.[]{.sentence-end} The syntax includes
    `buildah pull [options] `{.inlineCode}`<`{.inlineCode}`image`{.inlineCode}`>`{.inlineCode}.[]{.sentence-end}
    Examples of this command include
    `buildah pull `{.inlineCode}`<`{.inlineCode}`imageName`{.inlineCode}`>`{.inlineCode},
    `buildah pull docker://`{.inlineCode}`<`{.inlineCode}`URL`{.inlineCode}`>`{.inlineCode}`/repository:tag`{.inlineCode},
    and
    `buildah pull dir:`{.inlineCode}`<`{.inlineCode}`/path/to/dir`{.inlineCode}`>`{.inlineCode}.

All the commands described previously have their corresponding
`man`{.inlineCode} page, with the
`man buildah-`{.inlineCode}`<`{.inlineCode}`command`{.inlineCode}`>`{.inlineCode}
pattern.[]{.sentence-end} For example, to read documentation details
about the `buildah run`{.inlineCode} command, just type
`man buildah-run`{.inlineCode} on the terminal.

The next example shows basic Buildah capabilities.[]{.sentence-end} A
Fedora base image is customized to run an `httpd`{.inlineCode} process:

``` {.programlisting .snippet-con}
$ container=$(buildah from fedora)
$ buildah run $container -- dnf install -y httpd; dnf clean all
$ buildah config --cmd "httpd -DFOREGROUND" $container
$ buildah config --port 80 $container
$ buildah commit $container myhttpd
$ buildah tag myhttpd registry.example.com/myhttpd:v0.0.1
```

The preceding commands will produce an OCI-compliant, portable image
with the same features as an image built from a Dockerfile, all in a few
lines that can be included in a simple script.

 note
**Good to** **know**

While the preceding example uses `buildah run`{.inlineCode} to execute
commands inside the container, the real *secret weapon* of Buildah is
the `buildah mount`{.inlineCode} command.

One of the primary reasons developers choose Buildah over standard
Dockerfiles is the ability to interact with the container\'s filesystem
directly from the host.[]{.sentence-end} When you run
`buildah mount $container`{.inlineCode}, the container\'s root
filesystem is mapped to a path on your host machine.[]{.sentence-end}
This unlocks several powerful workflows.


We will now focus on the first command:

``` {.programlisting .snippet-con}
$ container=$(buildah from fedora)
```

The `buildah from`{.inlineCode} command[]{#Chapter_6.xhtml#idx_03276c07
.index-entry index-entry="Buildah:commands"} pulls a Fedora image from
one of the allowed registries and spins up a working container from it,
returning the container name.[]{.sentence-end} Instead of simply having
it printed on standard output, we will capture the name with shell
expansion syntax.[]{.sentence-end} From now on, we can pass the
`$container`{.inlineCode} variable, which holds the name of the
generated container, to the subsequent commands.[]{.sentence-end}
Therefore, the build commands will be executed inside this working
container.[]{.sentence-end} This is quite a common pattern and is
especially useful to automate Buildah commands in scripts.

 note
**Important** **note**

There is a subtle difference between the concept of containers in
Buildah and Podman.[]{.sentence-end} Both adopt the same technology to
create containers, but Buildah containers are short-lived entities that
are created to be modified and committed, while Podman containers are
supposed to run long-living workloads.

Also, please consider that commands such as `podman ps`{.inlineCode} do
not show Buildah containers by default and require the
`--external`{.inlineCode} option.


The flexible and embeddable nature of this approach is remarkable --
Buildah commands can be included anywhere, and users can choose between
a fully automated build process and a more interactive one.

For example, Buildah can be easily
integrated[]{#Chapter_6.xhtml#idx_60293904 .index-entry
index-entry="Ansible"} with **Ansible**, the open source automation
engine, to provide automated builds using native connection plugins that
enable communication with working containers.

You can choose to include Buildah inside a
[]{#Chapter_6.xhtml#idx_242472e3 .index-entry index-entry="Jenkins"}CI
pipeline (such as **Jenkins**, **Tekton**, or **GitLab CI/CD**) to gain
[]{#Chapter_6.xhtml#idx_4ac60318 .index-entry
index-entry="GitLab CI/CD"}full control of the build and integration
tasks.[]{.sentence-end} Tekton, the cloud-native
CI/CD[]{#Chapter_6.xhtml#idx_eb0d1888 .index-entry
index-entry="Tekton:URL"} tool
([[https://tekton.dev/]{.url}](https://tekton.dev/){style="text-decoration: none;"}),
offers a collection of template tasks, the building block of Tekton
pipelines, on the Tekton Hub repository
([[https://hub.tekton.dev/]{.url}](https://hub.tekton.dev/){style="text-decoration: none;"});
Buildah tasks are also available on Tekton Hub
([[https://hub.tekton.dev/tekton/task/buildah]{.url}](https://hub.tekton.dev/tekton/task/buildah){style="text-decoration: none;"}).[]{.sentence-end}
In addition, Buildah can be run inside a container without issue,
allowing it to run in many other CI pipelines, even if they do not
normally support container builds.

Buildah is also included in larger []{#Chapter_6.xhtml#idx_621c150d
.index-entry index-entry="Shipwright project"}projects of the
cloud-native community, such as the **Shipwright** project
([[https://github.com/shipwright-io/build]{.url}](https://github.com/shipwright-io/build){style="text-decoration: none;"}).

Shipwright is an extensible build framework for Kubernetes that provides
the flexibility of customizing image builds using custom resource
definitions and different build tools.[]{.sentence-end} Buildah is one
of the available solutions that you can choose when designing your build
processes.

We will see more detailed and richer examples in the next
subsections.[]{.sentence-end} Now that we have seen an overview of
Buildah\'s capabilities and use cases, let\'s dive into the installation
and environment preparation steps.

# Preparing our environment {#Chapter_6.xhtml#h1_171 .heading-1}

Buildah[]{#Chapter_6.xhtml#idx_1912e562 .index-entry
index-entry="Buildah:environment, preparing"} is available on different
distributions and can be installed using the respective package
managers.[]{.sentence-end} This section provides a non-exhaustive list
of installation examples on the major distributions.[]{.sentence-end}
For the sake of clarity, it is important to reiterate that the book lab
environments were all based on Fedora 40:

-   **Fedora**: To install Buildah on Fedora and every other Linux
    distribution that uses the DNF package manager, run the following
    `dnf`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ sudo dnf -y install buildah
    ```

-   **Debian**: To install Buildah on Debian Bullseye or later, run the
    following `apt-get`{.inlineCode} commands:

    ``` {.programlisting .snippet-con-one}
    $ sudo apt-get update
    $ sudo apt-get -y install buildah
    ```

-   **CentOS Stream 9**: To install Buildah on CentOS Stream 9, run the
    following `yum`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ sudo dnf install -y buildah
    ```

-   **RHEL8**: To install Buildah on RHEL8, run the following
    `yum module`{.inlineCode} commands:

    ``` {.programlisting .snippet-con-one}
    $ sudo yum module enable -y container-tools:1.0
    $ sudo yum module install -y buildah
    ```

-   **RHEL9**: To install Buildah on RHEL9, install the
    `container-tools`{.inlineCode} meta-package:

    ``` {.programlisting .snippet-con-one}
    $ sudo dnf      -y install container-tools     
    ```

-   **Arch Linux**: To install Buildah on Arch Linux, run the following
    `pacman`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ sudo pacman -S buildah
    ```

-   **Ubuntu**: To install Buildah on Ubuntu 20.10 or later, run the
    following `apt-get`{.inlineCode} commands:

    ``` {.programlisting .snippet-con-one}
    $ sudo apt-get -y update
    $ sudo apt-get -y install buildah
    ```

-   **Gentoo**: To install Buildah on Gentoo, run the following
    `emerge`{.inlineCode} command:

    ``` {.programlisting .snippet-con-one}
    $ sudo emerge app-emulation/libpod
    ```

-   **Build from source**: Buildah[]{#Chapter_6.xhtml#idx_26aff88e
    .index-entry index-entry="Buildah:environment, preparing"} can also
    be built from the source.[]{.sentence-end} For the purpose of this
    book, we will keep the focus on simple deployment methods, but if
    you\'re curious, you will find the following guide useful to try out
    your own builds:
    [[https://github.com/containers/buildah/blob/main/install.md#building-from-scratch]{.url}](https://github.com/containers/buildah/blob/main/install.md#building-from-scratch){style="text-decoration: none;"}.

Finally, Buildah can be deployed as a container, and builds can be
executed inside it with a nested approach.[]{.sentence-end} This process
will be covered in greater detail in *[Chapter
7](#Chapter_7.xhtml#h1_183){.chapref}*, *Integrating* *with Existing
Application Build Processes*.

After installing Buildah on our host, we can move on to verifying our
installation.

## Verifying the installation {#Chapter_6.xhtml#h2_172 .heading-2}

After installing Buildah, we can now run
[]{#Chapter_6.xhtml#idx_ae54dca4 .index-entry
index-entry="Buildah:installation, verifying"}some basic test commands
to verify the installation.

To see all the available images in the host local store, use the
following commands:

``` {.programlisting .snippet-con}
$ buildah images
# buildah images
```

The image list will be the same as the one printed by the
`podman images`{.inlineCode} command, since they share the same local
store.

Also note that the two commands are executed as an unprivileged user and
as root, pointing respectively to the user rootless local store and the
system-wide local store.

We can run a simple test build to verify the
installation.[]{.sentence-end} This is a good chance to test a basic
build script whose only purpose is to verify whether Buildah is able to
fully run a complete build.

For the purpose of this[]{#Chapter_6.xhtml#idx_94d7c5f2 .index-entry
index-entry="Buildah:installation, verifying"} book (and for fun), we
have created the following simple test script that creates a minimal
Python 3 image:

``` {.programlisting .snippet-code}
#!/bin/bash

BASE_IMAGE=alpine
TARGET_IMAGE=python3-minimal

if [ $UID != 0 ]; then
    echo "### Running build test as unprivileged user"
else
    echo "### Running build test as root"
fi

echo "### Testing container creation"
container=$(buildah from $BASE_IMAGE)
if [ $? -ne 0 ]; then
    echo "Error initializing working container"
fi

echo "### Testing run command"
buildah run $container apk add --update python3 py3-pip
if [ $? -ne 0 ]; then
    echo "Error on run build action"
fi

echo "### Testing image commit"
buildah commit $container $TARGET_IMAGE
if [ $? -ne 0 ]; then
    echo "Error committing final image"
fi

echo "### Removing working container"
buildah rm $container
if [ $? -ne 0 ]; then
    echo "Error removing working container"
fi

echo "### Build test completed successfully!"
exit 0
```

The same test script can be []{#Chapter_6.xhtml#idx_fedfef4e
.index-entry index-entry="Buildah:installation, verifying"}executed by a
non-privileged user and by root.

We can verify the newly built image by running a simple container that
executes a Python shell:

``` {.programlisting .snippet-con}
$ podman run -it python3-minimal /usr/bin/python3
Python 3.12.7 (main, Oct  7 2024, 11:30:19)Python 3.9.5 (default, May 12 2021, 20:44:22)
[GCC 13.2.1 2024030910.3.1 20210424] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

After successfully testing our new Buildah installation, let\'s inspect
the main configuration files used by Buildah.

## Buildah configuration files {#Chapter_6.xhtml#h2_173 .heading-2}

The main Buildah configuration files[]{#Chapter_6.xhtml#idx_cfcd30c9
.index-entry index-entry="Buildah configuration files"} are the same
ones used by Podman.[]{.sentence-end} They can be leveraged to customize
the behavior of the working containers executed in builds.

On Fedora, these config files are installed by the
`containers-common`{.inlineCode} package, and we already covered them in
the *Prepare your environment* section in *[Chapter
3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the First Container*.

The main config files used by Buildah are as follows:

-   `/usr/share/containers/mounts.conf`{.inlineCode}: This config file
    defines the files and directories that are automatically mounted
    inside a Buildah working container
-   `/etc/containers/registries.conf`{.inlineCode}: This config file has
    the role of managing registries allowed to be accessed for image
    searches, pulls, and pushes
-   `/usr/share/containers/policy.json`{.inlineCode}: This JSON config
    file defines image signature verification behavior
-   `/usr/share/containers/seccomp.json`{.inlineCode}: This JSON config
    file defines the allowed and prohibited syscalls to a containerized
    process
-   `/usr/share/containers/containers.conf`{.inlineCode}: This config
    file contains many Podman-specific settings, but it\'s also used by
    Buildah

In this section, we have learned how to prepare the host environment to
run Buildah.[]{.sentence-end} In the next section, we are going to
identify the possible build strategies that can be implemented with
Buildah.

# Choosing our build strategy {#Chapter_6.xhtml#h1_174 .heading-1}

There are basically three types of build
strategies[]{#Chapter_6.xhtml#idx_7fa8346a .index-entry
index-entry="build strategies:selecting"} that we can use with Buildah:

-   Building a container image starting from an existing base image
-   Building a container image starting from scratch
-   Building a container image starting from a Dockerfile

We have already provided an example of the build strategy from an
existing base image in the *Introducing* *Buildah, Podman\'s companion*
section.[]{.sentence-end} Since this strategy is pretty similar from a
workflow point of view to building from scratch, we will focus our
practical examples on the last one, which provides great flexibility to
create a small footprint and secure images.

Before going through the various technical details in the next section,
let\'s start exploring all these strategies at a high level.

Even though we can find a lot of prebuilt container images available on
the most popular public container registries, sometimes we might not be
able to find a particular configuration, setup, or bundle of tools and
services for our containers; that is why container image creation
becomes a really important step that we need to
practice.[]{.sentence-end} In addition to that, the image creation also
allows bundling your own configuration files to customize an image for
your exact use.

Also, security constraints often []{#Chapter_6.xhtml#idx_8b2ad2af
.index-entry index-entry="build strategies:selecting"}require us to
implement images with reduced attack surfaces, and therefore, DevOps
teams must know how to customize every step of the build process to
achieve this result.[]{.sentence-end} In enterprise contexts, for
optimization and security reasons, images are almost always created from
scratch or using minimal base images ready to be customized.

With this awareness in mind, let\'s start with the first build strategy.

## Building a container image starting from an existing base image {#Chapter_6.xhtml#h2_175 .heading-2}

Let\'s imagine finding a []{#Chapter_6.xhtml#idx_847826aa .index-entry
index-entry="container images:building, from existing base image"}well-done
prebuilt container image for our favorite application server that our
company is widely using.[]{.sentence-end} All the configurations for
this container image are okay, and we can attach storage to the right
mount points to persist the data and so on, but sooner or later, we may
realize that some particular tools that we use for troubleshooting are
missing in the container image, or that some libraries are missing that
should be included!

In another scenario, we could be happy with the prebuilt image but still
need to add custom content to it -- for example, the customer
application.

What would be the solution in those cases?

In this first use case, we can extend the existing container image,
adding stuff and editing the existing files to suit our
purposes.[]{.sentence-end} In the previous basic examples, Fedora and
Alpine images were customized to serve different
purposes.[]{.sentence-end} Those images were generic OS filesystems with
no specific purpose, but the same concept can be applied to a more
complex image.

In the second use case, we can customize an image -- for example, the
default library, `Httpd`{.inlineCode}.[]{.sentence-end} We can install
PHP modules and then add our application\'s PHP files, producing a new
image with our custom content[]{#Chapter_6.xhtml#idx_a69c79e3
.index-entry
index-entry="container images:building, from existing base image"}
already built in.

We will see in the next sections that we can extend an existing
container image.

Let\'s move on to the second strategy.

## Building a container image starting from scratch {#Chapter_6.xhtml#h2_176 .heading-2}

The previous strategy would be []{#Chapter_6.xhtml#idx_9d3f76b7
.index-entry
index-entry="container images:building, from scratch"}enough for many
common situations, where we can find a prebuilt image to start working
with, but sometimes it may be that the particular use case, application,
or service that we want to containerize is not so common or widely used.

Imagine having a custom legacy application that requires some old
libraries and tools that are no longer included on the latest Linux
distribution, or that may have been replaced by more recent
ones.[]{.sentence-end} In this scenario, you might need to start from an
empty container image and add all the necessary stuff for your legacy
application piece by piece.

Finally, `FROM scratch`{.inlineCode} is also a way of distributing
super-minimal container images that contain only the files necessary to
run your container.[]{.sentence-end} You can even make single-file
images with only a single static binary inside them.

We have learned in this chapter that, actually, we will always start
from a sort of initial container image, so this strategy and the
previous one are pretty much the same.

Let\'s move on to the third and final strategy.

## Building a container image starting from a Dockerfile {#Chapter_6.xhtml#h2_177 .heading-2}

In *[Chapter 1](#Chapter_1.xhtml#h1_14){.chapref}*, *Introduction to
Container Technology*, we talked about[]{#Chapter_6.xhtml#idx_e2b9c160
.index-entry
index-entry="container images:building, starting from  Dockerfile"}
container technology history and how Docker gained momentum in that
context.[]{.sentence-end} Podman was born as an alternative evolution
project of the great concepts that Docker helped to develop until
now.[]{.sentence-end} One of the great innovations that Docker created
in its own project history is, for sure, the Dockerfile.

Looking into this strategy at a high level, we can affirm that even when
using a Dockerfile, we will arrive at one of the previous build
strategies.[]{.sentence-end} The reality is not far away from the latest
assumption we made, because Buildah, under the hood, will parse the
Dockerfile, and it will build the container that we briefly introduced
for previous build strategies.

So, in summary, are there any differences or advantages we need to
consider when choosing our default build strategy? Obviously, there is
no ultimate answer to this question.[]{.sentence-end} First of all, we
should always look into the container communities, searching for some
prebuilt image that could help our *build* process; on the other hand,
we can always fall back on the *build-from-scratch*
process.[]{.sentence-end} Last but []{#Chapter_6.xhtml#idx_740125bf
.index-entry
index-entry="container images:building, starting from  Dockerfile"}not
least, we can consider Dockerfile for easily distributing and sharing
our build steps with our development group or the whole container
community.

This ends our quick high-level introduction; we can now move on to the
practical examples!

# Building images from scratch {#Chapter_6.xhtml#h1_178 .heading-1}

Before going into the details of this section and learning how to build
a container image from scratch, let\'s make some tests
[]{#Chapter_6.xhtml#idx_acf829b7 .index-entry
index-entry="images:building, from scratch"}to verify that the installed
Buildah is working properly.

First of all, let\'s check whether our Buildah image cache is empty:

``` {.programlisting .snippet-con}
# buildah images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
# buildah containers -a
CONTAINER ID  BUILDER  IMAGE ID     IMAGE NAME                       CONTAINER NAME
```

 note
**Important** **note**

Podman and Buildah share the same container storage; for this reason, if
you previously ran any other example shown in this chapter or book, you
will find that your container storage cache is not that empty!


As we learned in the previous[]{#Chapter_6.xhtml#idx_d8a9ea10
.index-entry index-entry="images:building, from scratch"} section, we
can leverage the fact that Buildah will output the name of the
just-created working container to easily store it in an environment
variable and use it once needed.[]{.sentence-end} Let\'s create a
brand-new container from scratch:

``` {.programlisting .snippet-con}
# buildah from scratch
# buildah images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
# buildah containers
CONTAINER ID  BUILDER  IMAGE ID     IMAGE NAME
CONTAINER NAME
af69b9547db9     *                  scratch
working-container
```

As you can see, we used the special `from scratch`{.inlineCode} keywords
that tell Buildah to create an empty container with no data inside
it.[]{.sentence-end} If we run the `buildah images`{.inlineCode}
command, we will note that this special image is not listed.

Let\'s check whether the working container built from scratch is really
empty:

``` {.programlisting .snippet-con}
# buildah run working-container bash
2021-10-26T20:15:49.000397390Z: executable file `bash` not found in $PATH: No such file or directory
error running container: error from crun creating container for [bash]: : exit status 1
error while running runtime: exit status 1
```

No executable was found in our empty container -- what a surprise! The
reason is that the working container has been created on an empty
filesystem.

Let\'s see how we can easily fill []{#Chapter_6.xhtml#idx_842f08e4
.index-entry index-entry="images:building, from scratch"}this empty
container.[]{.sentence-end} In the following example, we will interact
directly with the underlying storage, using the package manager of our
host system to install the binaries and the libraries needed for running
a `bash`{.inlineCode} shell in our container image.

First of all, let\'s instruct Buildah to mount the container storage and
check where it resides:

``` {.programlisting .snippet-con}
# buildah mount working-container
/var/lib/containers/storage/overlay/b5034cc80252b6f4af2155f9e0a2a7e65b77dadec7217bd2442084b1f4449c1a/merged
```

 note
**Good to** **know**

If you start the build in rootless mode, Buildah will run the mount in a
different namespace, and for this reason, the mounted volume might not
be accessible from the host when using a driver different from
`vfs`{.inlineCode}.[]{.sentence-end} Alternatively, you can run the
commands in a `podman unshare`{.inlineCode} shell.


Great! Now that we have found it, we can leverage the host package
manager to install all the needed packages in this `root`{.inlineCode}
folder, which will be the `root`{.inlineCode} path of our container
image:

``` {.programlisting .snippet-con}
# scratchmount=$(buildah mount working-container)
# dnf install --installroot $scratchmount --releasever 40 bash coreutils --setopt install_weak_deps=false -y
```

 note
**Important** **note**

If you are running the previous command on a Fedora release different
from version 40, (for example, version 39), then you need to import the
GPG public keys of Fedora 40 or use the additional option --
`--nogpgcheck`{.inlineCode}.


First of all, we will save the very long directory path in an
environment variable and then execute the `dnf`{.inlineCode} package
manager, passing the just-obtained directory path as the install root
directory, setting the release version of our Fedora OS, specifying the
packages that we want to install (`bash`{.inlineCode} and
`coreutils`{.inlineCode}), and finally, disabling weak dependencies,
accepting all the changes to the system.

The command should end up[]{#Chapter_6.xhtml#idx_7739ffa8 .index-entry
index-entry="images:building, from scratch"} with a
`Complete!`{.inlineCode} statement; once done, let\'s try again with the
same command that earlier in this section we saw was failing:

``` {.programlisting .snippet-con}
# buildah run working-container bash
bash-5.2# cat /etc/fedora-release
 Fedora release  40 (Forty)
```

It worked! We just installed a Bash shell in our empty
container.[]{.sentence-end} Let\'s see now how to finish our image
creation with some other configuration steps.[]{.sentence-end} First of
all, to our final container image, we need to add a command to be run
once it is up and running.[]{.sentence-end} For this reason, we will
create a Bash script file with some basic commands:

``` {.programlisting .snippet-con}
# cat command.sh
#!/bin/bash
cat /etc/fedora-release
/usr/bin/date
```

We have created a Bash script file that prints the container Fedora
release and the system date.[]{.sentence-end} The file must have execute
permissions before being copied:

``` {.programlisting .snippet-con}
# chmod +x command.sh
```

Now that we have filled up our underlying container storage with all the
needed base packages, we can unmount the
`working-container`{.inlineCode} storage and use the
`buildah copy`{.inlineCode} command to inject files from the host to the
container:

``` {.programlisting .snippet-con}
# buildah unmount working-container
af69b9547db93a7dc09b96a39bf5f7bc614a7ebd29435205d358e09ac99857bc
# buildah copy working-container ./command.sh /usr/bin
659a229354bdef3f9104208d5812c51a77b2377afa5ac819e3c3a1a2887eb9f7
```

The `buildah copy`{.inlineCode} command gives us the ability to work
with the underlying storage without worrying about mounting it
or[]{#Chapter_6.xhtml#idx_562fde5a .index-entry
index-entry="images:building, from scratch"} handling it under the hood.

We are now ready to complete our container image by adding some metadata
to it:

``` {.programlisting .snippet-con}
# buildah config --cmd /usr/bin/command.sh working-container
# buildah config --created-by "Podman for DevOps example" working-container
# buildah config --label name=fedora-date working-container
```

We started with the `cmd`{.inlineCode} option, and after that, we added
some descriptive metadata.[]{.sentence-end} We can finally commit
`working-container`{.inlineCode} into an image!

``` {.programlisting .snippet-con}
# buildah commit working-container fedora-date
Getting image source signatures
Copying blob 939ac17066d4 done
Copying config e24a2fafde done
Writing manifest to image destination
Storing signatures
e24a2fafdeb5658992dcea9903f0640631ac444271ed716d7f749eea7a651487
```

Let\'s clean up the environment and check the available container images
on the host:

``` {.programlisting .snippet-con}
# buildah rm working-container
af69b9547db93a7dc09b96a39bf5f7bc614a7ebd29435205d358e09ac99857bc
```

We can now inspect the details of the just-created container image:

``` {.programlisting .snippet-con}
# podman images
REPOSITORY             TAG         IMAGE ID      CREATED             SIZE
localhost/fedora-date  latest      e24a2fafdeb5  About a minute ago  366 MB
# podman inspect localhost/fedora-date:latest
[...omitted output]        "Labels": {
            "io.buildah.version": "1.37.3",
            "name": "fedora-date"
        },
        "Annotations": {
            "org.opencontainers.image.base.digest": "",
            "org.opencontainers.image.base.name": ""
        },
        "ManifestType": "application/vnd.oci.image.manifest.v1+json",
        "User": "",
        "History": [
            {
                "created": "2024-11-01T13:51:50.170096562Z",
                "created_by": "Podman for DevOps example"
            }
        ],
        "NamesHistory": [
            "localhost/fedora-date:latest"
        ]
    }
]
```

As we can see from the previous output, the container image has a lot of
metadata that can tell us many details.[]{.sentence-end} Some of them we
set through the previous commands, such as the
`created_by`{.inlineCode}, `name`{.inlineCode}, and `Cmd`{.inlineCode}
tags; the other []{#Chapter_6.xhtml#idx_f4cfa65e .index-entry
index-entry="images:building, from scratch"}tags are populated
automatically by Buildah.

Finally, let\'s run our brand-new container image with Podman!

``` {.programlisting .snippet-con}
# podman run -ti localhost/fedora-date:latest
Fedora release 40 (Forty)
Tue Nov 01 21:18:29 UTC 2024
```

This ends our journey in creating a container image from
scratch.[]{.sentence-end} As we saw, this is not a typical method for
creating a container image; in many scenarios and for various use cases,
it can be enough to start with an OS base image, such as
`from fedora`{.inlineCode} or `from alpine`{.inlineCode}, and then add
the required packages, using the respective package managers available
in those images.

 note
**Good to** **know**

Some Linux distributions also provide base container images in a
**minimal** flavor (for example, `fedora-minimal`{.inlineCode}) that
reduces the number of packages installed, as well as the size of the
target container image.[]{.sentence-end} For more information, refer to
[[https://www.docker.com/]{.url}](https://www.docker.com/){style="text-decoration: none;"}
and
[[https://quay.io/]{.url}](https://quay.io/){style="text-decoration: none;"}!


Let\'s now inspect how to build images from Dockerfiles with Buildah.

# Building images from a Dockerfile {#Chapter_6.xhtml#h1_179 .heading-1}

As we described earlier in this[]{#Chapter_6.xhtml#idx_f0752b1b
.index-entry index-entry="images:building, from Dockerfile"} chapter,
the Dockerfile can be an easy option to create and
[]{#Chapter_6.xhtml#idx_ad4631e3 .index-entry
index-entry="Dockerfile:images, building"}share the build steps for
creating a container image, and for this reason, it is really easy to
find a lot of source Dockerfiles on the net.

The first step of this activity is to build a simple Dockerfile to work
with.[]{.sentence-end} Let\'s create a Dockerfile for creating a
containerized web server:

``` {.programlisting .snippet-code}
# mkdir webserver
# cd webserver/
[webserver]# vi Dockerfile
[webserver]# cat Dockerfile
# Start from latest fedora container base image
FROM fedora:latest

# Update the container base image
RUN echo "Updating all fedora packages"; dnf -y update; dnf -y clean all

# Install the httpd package
RUN echo "Installing httpd"; dnf -y install httpd

# Expose the http port 80
EXPOSE 80

# Set the default command to run once the container will be started
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

Looking at the previous output, we first created a new directory, and
inside, we created a text file named
`Dockerfile`{.inlineCode}.[]{.sentence-end} After
[]{#Chapter_6.xhtml#idx_fbb211e3 .index-entry
index-entry="images:building, from Dockerfile"}that, we inserted the
various keywords and steps commonly used in the definition of a
brand-new Dockerfile; every step and keyword has a
[]{#Chapter_6.xhtml#idx_4aba6920 .index-entry
index-entry="Dockerfile:images, building"}dedicated description comment
on top, so the file should be easy to read.

Just to recap, these are the steps contained in our brand-new
Dockerfile:

1.  Start from the latest Fedora container base image.
2.  Update all the packages for the container base image.
3.  Install the `httpd`{.inlineCode} package.
4.  Expose HTTP port `80`{.inlineCode}.
5.  Set the default command to run once the container is started.

As seen previously in this chapter, Buildah provides a dedicated
`buildah build`{.inlineCode} command to start a build from a Dockerfile.

Let\'s see how it works:

``` {.programlisting .snippet-con}
[webserver]# buildah build -f Dockerfile -t myhttpdservice .
STEP 1/6: FROM fedora:latest
Resolved "fedora" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull registry.fedoraproject.org/fedora:latest...
Getting image source signatures
Copying blob 944c4b241113 done
Copying config 191682d672 done
Writing manifest to image destination
Storing signatures
STEP 2/6: MAINTAINER podman-book  # this should be an email
STEP 3/6: RUN echo "Updating all fedora packages"; dnf -y update; dnf -y clean all
Updating all fedora packages
Fedora 40 - x86_64                               16 MB/s |  74 MB     00:04
...
STEP 4/6: RUN echo "Installing httpd"; dnf -y install httpd
Installing httpd
Fedora 40 - x86_64                               20 MB/s |  74 MB     00:03  
...
STEP 5/6: EXPOSE 80
STEP 6/6: CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
COMMIT myhttpdservice
Getting image source signatures
Copying blob 7500ce202ad6 skipped: already exists
Copying blob 51b52d291273 done
Copying config 14a2226710 done
Writing manifest to image destination
Storing signatures
--> 14a2226710e
Successfully tagged localhost/myhttpdservice:latest
14a2226710e7e18d2e4b6478e09a9f55e60e0666dd8243322402ecf6fd1eaa0d
```

As we can see from the previous output, we pass the following options to
the `buildah build`{.inlineCode} command:

-   `-f`{.inlineCode}: To define the name of the
    Dockerfile.[]{.sentence-end} The default filename is
    `Dockerfile`{.inlineCode}, so in our case, we can omit this option
    because we named the file as the default one.
-   `-t`{.inlineCode}: To define the name and the tag of the image we
    are building.[]{.sentence-end} In our case, we are only defining the
    name.[]{.sentence-end} The image will be tagged as
    `latest`{.inlineCode} by default.

Finally, as the last option, we[]{#Chapter_6.xhtml#idx_95514c4c
.index-entry index-entry="images:building, from Dockerfile"} need to
specify the directory where Buildah needs to work and search for the
Dockerfile.[]{.sentence-end} In our case, we are passing
the`.`{.inlineCode} Directory, which[]{#Chapter_6.xhtml#idx_4706e841
.index-entry index-entry="Dockerfile:images, building"} represents the
current working directory.

Of course, these are not the only options that Buildah gives us to
configure the build; we will see some of them later in this section.

Coming back to the command we just executed, as we can see from the
output, all the steps defined in the Dockerfile have been executed in
the exact order and printed with a given fractional number to show the
intermediate steps against the total number.[]{.sentence-end} In total,
six steps were executed.

We can check the result of our command by listing the images with the
`buildah images`{.inlineCode} command:

``` {.programlisting .snippet-con}
[webserver]# buildah images
REPOSITORY                                  TAG      IMAGE ID       CREATED          SIZE
localhost/myhttpdservice                    latest   14a2226710e7   2 minutes ago   497 MB
```

As we can see, our container[]{#Chapter_6.xhtml#idx_87562a0b
.index-entry index-entry="images:building, from Dockerfile"} image has
just been created with the `latest`{.inlineCode} tag;
let\'s[]{#Chapter_6.xhtml#idx_97299ef1 .index-entry
index-entry="Dockerfile:images, building"} try to run it:

``` {.programlisting .snippet-con}
# podman run -d localhost/myhttpdservice:latest
133584ab526faaf7af958da590e14dd533256b60c10f08acba6c1209ca05a885
# podman logs 133584ab526faaf7af958da590e14dd533256b60c10f08acba6c1209ca05a885
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.4. Set the 'ServerName' directive globally to suppress this message

# curl 10.88.0.4
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>Test Page for the HTTP Server on Fedora</title>
    <style type="text/css">
...
```

Looking at the output, we just ran our container in detached mode; after
that, we inspected the logs to find out the IP address that we need to
pass as an argument for the `curl`{.inlineCode} test command.

We just run the container as the root user on our workstation, and the
container just received an internal IP address on Podman\'s container
network interface.[]{.sentence-end} We can check that the IP address is
part of that network[]{#Chapter_6.xhtml#idx_bc92ca41 .index-entry
index-entry="images:building, from Dockerfile"} by running the following
[]{#Chapter_6.xhtml#idx_2f5c7852 .index-entry
index-entry="Dockerfile:images, building"}commands:

``` {.programlisting .snippet-con}
# ip a show dev cni-podman0
14: cni-podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether c6:bc:ba:7c:d3:0c brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global cni-podman0
       valid_lft forever preferred_lft forever
    inet6 fe80::c4bc:baff:fe7c:d30c/64 scope link
       valid_lft forever preferred_lft forever
```

As we can see, the container\'s IP address was taken from the network
reported in the previous `10.88.0.1/16`{.inlineCode} output.

As we anticipated, the `buildah build`{.inlineCode} command has a lot of
other options that can be useful while developing and creating brand-new
container images.[]{.sentence-end} Let\'s explore one of them that is
worth mentioning: `--layers`{.inlineCode}.

We already learned how to use this option with Podman earlier in this
chapter.[]{.sentence-end} Starting from version 1.2 of Buildah, the
development team added this great option that gives us the ability to
enable or disable the layers\' caching mechanism.[]{.sentence-end} The
default configuration sets the `--layers`{.inlineCode} option to
`false`{.inlineCode}, which means that Buildah will not keep
intermediate layers, resulting in a build that squashes all the changes
in a single layer.

It is also possible to set the management of the layers with an
environment variable -- for example, to enable layer caching, run
`export BUILDAH_LAYERS=true`{.inlineCode}.

While retaining intermediate layers consumes initial storage space, the
true impact on your system is more nuanced.[]{.sentence-end} Keeping
these layers allows for layer sharing across multiple different images,
which can actually save significant space and bandwidth; if 10 images
share the same base layers, those layers are stored and pulled only
once.

However, there are a couple of distinct trade-offs to consider:

-   **The** **downside of** **many** **layers**: Every additional layer
    adds complexity to the union filesystem.[]{.sentence-end} This can
    lead to slightly slower container startup times and increased
    overhead when the kernel merges the layers into a single view.
-   **The** **downside of** **squashing**: While *squashing* layers into
    a single one can improve startup speed and permanently remove files
    deleted in upper layers (files that would otherwise remain hidden
    but present in a multi-layered image), it destroys
    deduplication.[]{.sentence-end} A squashed image cannot share its
    layers with others, meaning the total
    storage[]{#Chapter_6.xhtml#idx_0c9bb551 .index-entry
    index-entry="images:building, from Dockerfile"} consumed on your
    host may actually increase[]{#Chapter_6.xhtml#idx_150c3e59
    .index-entry index-entry="Dockerfile:images, building"} if you
    maintain several similar images.

Ultimately, keeping intermediate layers is a balance: you trade a bit of
filesystem complexity for massive gains in build speed and storage
efficiency.

# Summary {#Chapter_6.xhtml#h1_180 .heading-1}

In this chapter, we explored a fundamental topic of container management
-- their creation.[]{.sentence-end} This step is mandatory if we want to
customize, keep updated, and manage our container infrastructure
correctly.[]{.sentence-end} We learned that Podman is often partnered
with another tool called Buildah that can help us in the process of
container image building.[]{.sentence-end} This tool has a lot of
options, like Podman, and shares a lot of them with it (storage
included!).[]{.sentence-end} Finally, we went through the different
strategies that Buildah offers us to build new container images, and one
of them is actually inherited by the Docker ecosystem -- the Dockerfile.

This chapter is only an introduction to the topic of container image
building; we will discover more advanced techniques in the next chapter!

# Further reading {#Chapter_6.xhtml#h1_181 .heading-1}

-   Buildah project tutorials:
    [[https://github.com/containers/buildah/tree/main/docs/tutorials]{.url}](https://github.com/containers/buildah/tree/main/docs/tutorials){style="text-decoration: none;"}
-   How to use Podman inside a container:
    [[https://www.redhat.com/sysadmin/podman-inside-container]{.url}](https://www.redhat.com/sysadmin/podman-inside-container){style="text-decoration: none;"}
-   How to build tiny container images:
    [[https://www.redhat.com/sysadmin/tiny-containers]{.url}](https://www.redhat.com/sysadmin/tiny-containers){style="text-decoration: none;"}

# Join us on Discord {#Chapter_6.xhtml#h1_182 .heading-1}

For discussions around the book and to connect with your peers, join us
on Discord at
[[packt.link/discordcloud]{.url}](https://packt.link/discordcloud){style="text-decoration: none;"}
or scan the QR code below:

![Image](images/B31467_6_1.png){style="width:25%;"}


[]{#Chapter_7.xhtml}

 {.section .chapter}