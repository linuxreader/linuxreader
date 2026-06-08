# Integrating with Existing Application Build Processes 

After learning how to create custom container images using Podman and
Buildah, we can now focus on special use cases that can make our build
workflows more efficient and portable.[]{.sentence-end} For instance,
small images are a very common requirement in an enterprise environment,
for performance and security reasons.[]{.sentence-end} We will explore
how to achieve this goal by breaking down the build process into
different stages.

This chapter will also try to uncover scenarios where Buildah is not
expected to run directly on a developer machine but is driven instead by
a container orchestrator or embedded inside custom applications that are
expected to call its libraries []{#Chapter_7.xhtml#idx_2de67b1b
.index-entry index-entry="command-line interface (CLI)"}or
**command-line interface** (**CLI**).

In this chapter, we\'re going to cover the following main topics:

-   Multistage container builds
-   Running Buildah inside a container
-   Integrating Buildah with custom builders

# Technical requirements {#Chapter_7.xhtml#h1_185 .heading-1}

Before proceeding with this chapter, a machine with a working Podman and
Buildah installation is required.[]{.sentence-end} As stated in
*[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the* *First
Container*, all the examples in the book are executed on a Fedora 40
system or later versions but can be reproduced on the reader\'s OS of
choice.

A good understanding of the topics covered in *[Chapter
6](#Chapter_6.xhtml#h1_163){.chapref}*, *Meet Buildah -- Building
Containers from Scratch*, will be useful to easily grasp concepts
regarding builds, both with native Buildah commands and from
Dockerfiles.

# Multistage container builds {#Chapter_7.xhtml#h1_186 .heading-1}

So far, we have learned how to create[]{#Chapter_7.xhtml#idx_6382ed7d
.index-entry index-entry="multistage container builds"} builds with
Podman and Buildah using Dockerfiles or native Buildah commands that
unleash potential advanced building techniques.[]{.sentence-end} There
is still an important point that we haven\'t yet discussed -- the size
of the images.

When creating a new image, we should always take care of its final size,
which is the result of the total number of layers and the number of
changed files inside them.

Minimal images with a small size have the great advantage of being able
to be pulled faster from registries.[]{.sentence-end} Nevertheless, a
large image will eat a lot of precious disk space in the host\'s local
store.

We already showed examples of some best practices to keep images compact
in size, such as building from scratch, cleaning up package manager
caches, and reducing the amount of `RUN`{.inlineCode},
`COPY`{.inlineCode}, and `ADD`{.inlineCode} instructions to the minimum
necessary.[]{.sentence-end} However, what happens when we need to build
an application from its source and create a final image with the final
artifacts?

Let\'s say we need to build a containerized Go application -- we should
start from a base image that includes Go runtimes, copy the source code,
and compile to produce the final binary with a series of intermediate
steps, most notably downloading all the necessary Go packages inside the
image cache.[]{.sentence-end} At the end of the build, we should clean
up all the source code and the downloaded dependencies and put the final
binary (which is statically linked in Go) in a working
directory.[]{.sentence-end} Everything will work, but the final image
will still include the Go runtimes included in the base image, which are
no longer necessary at the end of the compilation process.

When Docker was introduced and Dockerfiles gained momentum, this problem
was circumvented in different ways by DevOps teams who struggled to keep
images minimal.[]{.sentence-end} For example, **binary builds** were
a[]{#Chapter_7.xhtml#idx_8c75c973 .index-entry
index-entry="binary builds"} way to inject the final artifact compiled
externally inside the built image.[]{.sentence-end} This approach solves
the image size problem but removes the advantage of a standardized
environment for builds provided by runtime/compiler images.

A better approach is to share volumes between containers and have the
final container image grab the compiled artifacts from a first build
image.

To provide a standardized approach, Docker, and then the OCI
specifications, introduced the concept of **multistage
builds**.[]{.sentence-end} Multistage[]{#Chapter_7.xhtml#idx_3d94aebd
.index-entry index-entry="multistage builds"} builds, as the name
suggests, allow users to create builds with multiple stages using
different `FROM`{.inlineCode} instructions and have subsequent images
grab contents from the previous ones.

In the next subsections, we will explore how to achieve this result with
Dockerfiles/Containerfiles and with Buildah\'s native commands.

## Multistage builds with Dockerfiles {#Chapter_7.xhtml#h2_187 .heading-2}

The first approach[]{#Chapter_7.xhtml#idx_dde71335 .index-entry
index-entry="Dockerfiles:multistage builds"} to multistage
[]{#Chapter_7.xhtml#idx_64e2286e .index-entry
index-entry="multistage builds:with Dockerfiles"}builds is creating
multiple stages in a single Dockerfile/Containerfile, with each block
beginning with a `FROM`{.inlineCode} instruction.

Build stages can copy files and folders from previous ones using the
`--from`{.inlineCode} option to specify the source stage.

The next examples show how to create a minimal multistage build for the
Go application, with the first stage acting as a pure build context and
the second stage copying the final artifact inside a minimal image.

This example is defined in the folloing file:
`Chapter07/http_hello_world/Dockerfile`{.inlineCode}:

``` {.programlisting .snippet-code}
# Builder image
FROM docker.io/library/golang

# Copy files for build
COPY go.mod /go/src/hello-world/
COPY main.go /go/src/hello-world/

# Set the working directory
WORKDIR /go/src/hello-world

# Download dependencies
RUN go get -d -v ./...

# Install the package
RUN go build -v

# Runtime image
FROM registry.access.redhat.com/ubi9/ubi-micro:latest
COPY --from=0 /go/src/hello-world/hello-world /
EXPOSE 8080

CMD ["/hello-world"]
```

The first stage copies the source `main.go`{.inlineCode} file and the
`go.mod`{.inlineCode} file to manage the Go module
dependencies.[]{.sentence-end} After downloading the dependency packages
(`go get -d -`{.inlineCode}`v ./`{.inlineCode}`...`{.inlineCode}), the
final application is built
(`go build -`{.inlineCode}`v ./...`{.inlineCode}).

The second stage[]{#Chapter_7.xhtml#idx_fa2d69c9 .index-entry
index-entry="Dockerfiles:multistage builds"} grabs the final artifact
(`/go/src/hello-world/hello-world`{.inlineCode}) and copies it under the
new image root.[]{.sentence-end} To specify that the source file
should[]{#Chapter_7.xhtml#idx_7f9a70d9 .index-entry
index-entry="multistage builds:with Dockerfiles"} be copied from the
first stage, the `--from=0`{.inlineCode} syntax is used.

In the first stage, we used the official
`docker.io`{.inlineCode}/`library`{.inlineCode}/`golang`{.inlineCode}
image, which includes the latest version of the Go programming
language.[]{.sentence-end} In the second stage, we used the
`ubi-micro`{.inlineCode} image, a minimal image from Red Hat with a
reduced footprint, optimized for microservices and statically linked
binaries.[]{.sentence-end} Universal Base Image will be covered in
greater detail in *[Chapter 8](#Chapter_8.xhtml#h1_199){.chapref}*,
*Choosing* *the* *Container Base Image*.

The Go []{#Chapter_7.xhtml#idx_1da3ce56 .index-entry
index-entry="multistage builds:with Dockerfiles"}application listed as
follows is a basic web server that
listens[]{#Chapter_7.xhtml#idx_33bc7543 .index-entry
index-entry="Dockerfiles:multistage builds"} on port
`8080/tcp`{.inlineCode} and prints a crafted HTML page with the
**\"Hello World!\"** message when it receives a `GET /`{.inlineCode}
request:

 note
**Important** **note**

For the purpose of this book, it is not necessary to be able to write or
understand the Go programming language.[]{.sentence-end} However, a
basic understanding of the language syntax and logic will prove to be
very useful, since the greater part of container-related software (such
as Podman, Docker, Buildah, Skopeo, Kubernetes, and OpenShift) is
written in Go.


`Chapter07/http_hello_world/main.go`{.inlineCode}

``` {.programlisting .snippet-code}
package main

import (
       "log"
   "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
     log.Printf("%s %s %s\n", r.RemoteAddr, r.Method, r.URL)
     w.Header().Set("Content-Type", "text/html")
     w.Write([]byte("<html>\n<body>\n"))
     w.Write([]byte("<p>Hello World!</p>\n"))
     w.Write([]byte("</body>\n</html>\n"))
}

func main() {
     http.HandleFunc("/", handler)
     log.Println("Starting http server")
     log.Fatal(http.ListenAndServe(":8080", nil))
}
```

The application can be built using either Podman or
Buildah.[]{.sentence-end} In this example, we chose to build the
application with Buildah:

``` {.programlisting .snippet-con}
$ cd http_hello_world
$ buildah build -t hello-world .
```

Finally, we can check the resulting image size:

``` {.programlisting .snippet-con}
$ buildah images --format '{{.Name}} {{.Size}}' \     localhost/hello-world
localhost/hello-world   36.1 MB
```

The final[]{#Chapter_7.xhtml#idx_ee9f1e91 .index-entry
index-entry="multistage builds:with Dockerfiles"} image has a size of
only 36 MB!

We can improve []{#Chapter_7.xhtml#idx_91e5e581 .index-entry
index-entry="Dockerfiles:multistage builds"}our Dockerfile by adding
custom names to the base images using the `AS`{.inlineCode}
keyword.[]{.sentence-end} The following example is a rework of the
previous Dockerfile following this approach, with the key elements
highlighted in bold:

`Chapter07/http_hello_world/Dockerfile-builder`{.inlineCode}

``` {.programlisting .snippet-code}
# Builder image
FROM docker.io/library/golang AS builder

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
FROM registry.access.redhat.com/ubi9/ubi-micro:latest AS srv
COPY --from=builder /go/src/hello-world/hello-world /
EXPOSE 8080

CMD ["/hello-world"]
```

In the preceding example, the name of the builder image is set as
`builder`{.inlineCode}, while the final image is named
`srv`{.inlineCode}.[]{.sentence-end} Interestingly, the
`COPY`{.inlineCode} instruction can now specify the builder as using the
custom name with the `--from=builder`{.inlineCode} option.

We can again build the container image with the following command:

``` {.programlisting .snippet-con}
$ cd http_hello_world
$ buildah build -f ./Dockerfile-builder -t hello-world-v2 .
```

Dockerfile/Containerfile[]{#Chapter_7.xhtml#idx_68932a76 .index-entry
index-entry="multistage builds:with Dockerfiles"} builds are the most
common approach but still[]{#Chapter_7.xhtml#idx_fccba0d6 .index-entry
index-entry="Dockerfiles:multistage builds"} lack some flexibility when
it comes to implementing a custom build workflow.[]{.sentence-end} For
those special use cases, Buildah native commands come to our rescue.

## Multistage builds with Buildah native commands {#Chapter_7.xhtml#h2_188 .heading-2}

As mentioned before, the multistage []{#Chapter_7.xhtml#idx_b35f60df
.index-entry
index-entry="multistage builds:with Buildah native commands"}build
feature is a great approach to []{#Chapter_7.xhtml#idx_e343449b
.index-entry
index-entry="Buildah native commands:multistage builds"}produce images
with a small footprint and a reduced attack surface.[]{.sentence-end} To
provide greater flexibility during the build process, the Buildah native
commands come to our rescue.[]{.sentence-end} As we mentioned earlier in
*[Chapter 6](#Chapter_6.xhtml#h1_163){.chapref}*, *Meet Buildah --
Building Containers from Scratch*, Buildah offers a series of commands
that replicate the behavior of the Dockerfile instructions, thus
offering greater control over the build process when those commands are
included in scripts or automations.

The same concept applies when working with multistage builds, where we
can also apply extra steps between the stages.[]{.sentence-end} For
instance, we can mount the build container overlay filesystem and
extract the built artifact to release alternate packages, all before
building the final runtime image.

The following[]{#Chapter_7.xhtml#idx_b065e1b1 .index-entry
index-entry="multistage builds:with Buildah native commands"} example
builds the same `hello-world`{.inlineCode} Go
[]{#Chapter_7.xhtml#idx_a3cbf824 .index-entry
index-entry="Buildah native commands:multistage builds"}application by
translating the previous Dockerfile instructions into native Buildah
commands, with everything inside a simple shell script:

`Chapter07/http_hello_world/buildah-commands.sh`{.inlineCode}

``` {.programlisting .snippet-code}
#!/bin/bash
# Define builder and runtime images
BUILDER=docker.io/library/golang
RUNTIME=registry.access.redhat.com/ubi9/ubi-micro:latest

# Create builder container
container1=$(buildah from $BUILDER)

# Copy files from host
if [ -f go.mod ]; then
    buildah copy $container1 'go.mod' '/go/src/hello-world/'
else
    exit 1
fi
if [ -f main.go ]; then
    buildah copy $container1 'main.go' '/go/src/hello-world/'
else
    exit 1
fi

# Configure and start build
buildah config --workingdir /go/src/hello-world $container1
buildah run $container1 go get -d -v ./...
buildah run $container1 go build -v ./...


# Create runtime container
container2=$(buildah from $RUNTIME)

# Copy files from the builder container
buildah copy --chown=1001:1001 \
    --from=$container1 $container2 \
    '/go/src/hello-world/hello-world' '/'

# Configure exposed ports
buildah config --port 8080 $container2

# Configure default CMD
buildah config --cmd /hello-world $container2

# Configure default user
buildah config --user=1001 $container2

# Commit final image
buildah commit $container2 helloworld

# Remove build containers
buildah rm $container1 $container2
```

In the preceding[]{#Chapter_7.xhtml#idx_6c3ec17e .index-entry
index-entry="multistage builds:with Buildah native commands"} example,
we highlighted the two []{#Chapter_7.xhtml#idx_8470cabd .index-entry
index-entry="Buildah native commands:multistage builds"}working
containers\' creation commands and the related `container1`{.inlineCode}
and `container2`{.inlineCode} variables that store the container ID.

Also, note the `buildah copy`{.inlineCode} command, where we have
defined the source container with the `--from`{.inlineCode} option, and
used the `--chown`{.inlineCode} option to define user and group owners
of the copied resource.[]{.sentence-end} This approach proves to be more
flexible than the Dockerfile-based workflow, since we can enrich our
script with variables, conditionals, and loops.

For instance, we have tested with the `if`{.inlineCode} condition in the
Bash script to check the existence of the `go.mod`{.inlineCode} and
`main.go`{.inlineCode} files before copying them inside the working
container dedicated to the build.

Let\'s now add []{#Chapter_7.xhtml#idx_6dc7bf1e .index-entry
index-entry="multistage builds:with Buildah native commands"}an extra
feature to the script.[]{.sentence-end} In the
following[]{#Chapter_7.xhtml#idx_96322af3 .index-entry
index-entry="Buildah native commands:multistage builds"} example, we
evolved the previous one by adding semantic versioning for the build and
creating a version archive before starting the build of the final
runtime image:

 note
**Important** **note**

The concept of semantic versioning[]{#Chapter_7.xhtml#idx_7ec170f0
.index-entry index-entry="semantic versioning:reference link"} is aimed
at providing a clear and standardized way to manage software versioning
and dependency management.[]{.sentence-end} It is a set of standard
rules whose purpose is to define how software release versions are
applied, and follows the *X.Y.Z* versioning pattern, where *X* is the
major version, *Y* is the minor version, and *Z* is the patch
version.[]{.sentence-end} For more information, check out the official
specifications:
[[https://semver.org/]{.url}](https://semver.org/){style="text-decoration: none;"}.


`Chapter07/http_hello_world/buildah-commands-release.sh`{.inlineCode}

``` {.programlisting .snippet-code}
#!/bin/bash
# Define builder and runtime images
BUILDER=docker.io/library/golang
RUNTIME=registry.access.redhat.com/ubi9/ubi-micro:latest
RELEASE=1.0.0

# Create builder container
container1=$(buildah from $BUILDER)

# Copy files from host
if [ -f go.mod ]; then
    buildah copy $container1 'go.mod' '/go/src/hello-world/'
else
    exit 1
fi
if [ -f main.go ]; then
    buildah copy $container1 'main.go' '/go/src/hello-world/'
else
    exit 1
fi

# Configure and start build
buildah config --workingdir /go/src/hello-world $container1
buildah run $container1 go get -d -v ./...
buildah run $container1 go build -v ./...
# Extract build artifact and create a version archive
buildah unshare --mount mnt=$container1 \
    sh -c 'cp $mnt/go/src/hello-world/hello-world .'
cat > README << EOF
Version $RELEASE release notes:
- Implement basic features
EOF
tar zcf hello-world-${RELEASE}.tar.gz hello-world README
rm -f hello-world README

# Create runtime container
container2=$(buildah from $RUNTIME)

# Copy files from the builder container
buildah copy --chown=1001:1001 \
    --from=$container1 $container2 \
    '/go/src/hello-world/hello-world' '/'

# Configure exposed ports
buildah config --port 8080 $container2

# Configure default CMD
buildah config --cmd /hello-world $container2

# Configure default user
buildah  config --user=1001 $container2

# Commit final image
buildah commit $container2 hello world:$RELEASE

# Remove build containers
buildah rm $container1 $container2
```

The key []{#Chapter_7.xhtml#idx_9bb248a7 .index-entry
index-entry="multistage builds:with Buildah native commands"}changes in
the script are again highlighted in bold.[]{.sentence-end} First, we
added a `RELEASE`{.inlineCode} variable that tracks the release version
of the application.[]{.sentence-end} Then, we extracted the build
artifact using the `buildah unshare`{.inlineCode} command, followed by
the `--mount`{.inlineCode} option to pass the container mount
point.[]{.sentence-end} The `unshare`{.inlineCode} user namespace was
necessary to make the script capable of running rootless.

After extracting the []{#Chapter_7.xhtml#idx_87ef9896 .index-entry
index-entry="Buildah native commands:multistage builds"}artifact, we
created a gzipped archive using the `$RELEASE`{.inlineCode} variable
inside the archive name and removed the temporary files.

Finally, we started the build of the runtime image and committed using
the `$RELEASE`{.inlineCode} variable again as the image tag.

In this section, we have learned how to run multistage builds with
Buildah using both Dockerfiles/Containerfiles and native
commands.[]{.sentence-end} In the next section, we will learn how to
isolate Buildah builds inside a container.

# Running Buildah inside a container {#Chapter_7.xhtml#h1_189 .heading-1}

Podman and []{#Chapter_7.xhtml#idx_b57c9c0d .index-entry
index-entry="Buildah:running, inside container"}Buildah follow a
fork/exec approach that makes them []{#Chapter_7.xhtml#idx_fbef9d6f
.index-entry index-entry="container:Buildah, running"}very easy to run
inside a container, including rootless container scenarios.

There are many use cases that imply the need for containerized
builds.[]{.sentence-end} Nowadays, one of the most common
adoption[]{#Chapter_7.xhtml#idx_ae890e30 .index-entry
index-entry="Kubernetes cluster"} scenarios is the application build
workflow running on top of a **Kubernetes** cluster.

Kubernetes is basically a container orchestrator that manages the
scheduling of containers from a control plane over a set of
worker[]{#Chapter_7.xhtml#idx_b70a4ec9 .index-entry
index-entry="Container Runtime Interface (CRI)"} nodes that run a
container engine compatible with the **Container Runtime Interface**
(**CRI**).[]{.sentence-end} Its design allows great flexibility in
customizing networking, storage, and runtimes, and leads to the great
flourishing of side projects that []{#Chapter_7.xhtml#idx_a78a923b
.index-entry index-entry="Cloud Native Computing Foundation (CNCF)"}are
now incubating or matured inside the **Cloud Native Computing
Foundation** (**CNCF**).

**Vanilla** Kubernetes (which is the basic community release without any
customization or add-ons) doesn\'t have a
native[]{#Chapter_7.xhtml#idx_89f729b5 .index-entry
index-entry="Vanilla Kubernetes"} build feature but offers the proper
framework to implement one.[]{.sentence-end} Over time, many solutions
appeared trying to address this need.

For example, Red []{#Chapter_7.xhtml#idx_105d3c99 .index-entry
index-entry="OpenShift"}Hat **OpenShift** introduced, way back when
Kubernetes 1.0 was released, its own build APIs and the
**Source-to-Image** toolkit[]{#Chapter_7.xhtml#idx_b0d01468 .index-entry
index-entry="Source-to-Image toolkit"} to create container images from
source code directly on top of the OpenShift cluster.

Other interesting solutions are Google\'s **kaniko**,
which[]{#Chapter_7.xhtml#idx_269c1334 .index-entry index-entry="kaniko"}
is a build tool to create container images inside a Kubernetes cluster
that runs every build step inside user space, and **Cloud Native
Buildpacks** (**CNBs**), which[]{#Chapter_7.xhtml#idx_ab1b0ce1
.index-entry index-entry="Cloud Native Buildpacks (CNBs)"} offer an
approach similar to Source-to-Image with advanced multi-process and
single-bill-of-materials management features.

Besides using already-implemented solutions, we can design our own
running Buildah inside containers that are orchestrated by
Kubernetes.[]{.sentence-end} We can also leverage the rootless-ready
design to implement secure build workflows.

It is possible to run CI/CD pipelines on top of a Kubernetes cluster and
embed containerized builds within a pipeline.[]{.sentence-end} One of
the most interesting CNCF projects, **Tekton Pipelines**,
offers[]{#Chapter_7.xhtml#idx_dfa52f3c .index-entry
index-entry="Tekton Pipelines"} a cloud-native approach to accomplish
this goal.[]{.sentence-end} Tekton allows running pipelines that are
driven by Kubernetes\' custom resources -- special APIs that extend the
basic API set.

Tekton Pipelines[]{#Chapter_7.xhtml#idx_d1ba1efb .index-entry
index-entry="Tekton Pipelines"} are made up of many different tasks, and
users can either create their own or grab them from **Tekton Hub**
([[https://hub.tekton.dev/]{.url}](https://hub.tekton.dev/){style="text-decoration: none;"}),
a []{#Chapter_7.xhtml#idx_4f6d8d09 .index-entry
index-entry="Tekton Hub:URL"}free repository where many pre-baked tasks
are available to be consumed immediately, including examples from
Buildah
([[https://hub.tekton.dev/tekton/task/buildah]{.url}](https://hub.tekton.dev/tekton/task/buildah){style="text-decoration: none;"}).

The preceding examples are useful to understand why containerized builds
are important.[]{.sentence-end} In this book, we want to focus on the
details of running builds within containers, with special attention paid
to security-related constraints.

## Running rootless Buildah containers with volume stores {#Chapter_7.xhtml#h2_190 .heading-2}

For the examples[]{#Chapter_7.xhtml#idx_994ad4e4 .index-entry
index-entry="rootless Buildah containers:running, with volume stores"}
in this subsection, the stable[]{#Chapter_7.xhtml#idx_8e18b321
.index-entry
index-entry="volume stores:rootless Buildah containers, running"}
upstream `quay.io/buildah/stable`{.inlineCode} Buildah image will be
used.[]{.sentence-end} This image already embeds the latest stable
Buildah binary.

Let\'s run our first example with a rootless container that builds the
contents of the `~/build`{.inlineCode} directory in the host and stores
the output in a local volume named `storevol`{.inlineCode}:

``` {.programlisting .snippet-con}
$ podman run --device /dev/fuse \
    -v ./http_hello_world:/build:z \
    -v storevol:/var/lib/containers quay.io/buildah/stable \
    buildah build -t build_test1 /build
```

This example contains some peculiar options that deserve attention:

-   The `--device /dev/fuse`{.inlineCode} option loads the fuse kernel
    module in the container, which is necessary to run fuse-overlay
    commands and mount the container filesystem
-   The `-v ~/build:/build:z`{.inlineCode} option bind-mounts the
    `/build`{.inlineCode} directory inside the container, assigning
    proper SELinux labeling with the `:z`{.inlineCode} suffix
-   The `-v storevol:/var/lib/containers`{.inlineCode} option creates a
    fresh volume mounted on the default container store, where all the
    layers are created

When the build[]{#Chapter_7.xhtml#idx_6d4134a5 .index-entry
index-entry="rootless Buildah containers:running, with volume stores"}
is complete, we can run a new []{#Chapter_7.xhtml#idx_38398bbb
.index-entry
index-entry="volume stores:rootless Buildah containers, running"}container
using the same volume and inspect or manipulate the built image:

``` {.programlisting .snippet-con}
$ podman run --rm -v storevol:/var/lib/containers quay.io/buildah/stable buildah images
REPOSITORY                                  TAG      IMAGE ID       CREATED          SIZE
localhost/build_test1                       latest   3605829966b5   41 seconds ago   33.9 MB
registry.access.redhat.com/ubi9/ubi-micro   latest   e279e18c7ef8   3 days ago       26.4 MB
docker.io/library/golang                    latest   0457bb691895   9 days ago       862 MB
```

We have successfully built an image whose layers have been stored inside
the `storevol`{.inlineCode} volume.[]{.sentence-end} To recursively list
the content of the store, we can extract the volume mount point with the
`podman volume inspect`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ ls -alR \
$(podman volume inspect storevol --format '{{.Mountpoint}}')
```

From now on, it is possible to launch a new Buildah container to
authenticate with the remote registry, tag the image, and push
it.[]{.sentence-end} In the following example, Buildah tags the
resulting image, authenticates with the remote registry, and then pushes
the image:

``` {.programlisting .snippet-con}
$ podman run --rm -v storevol:/var/lib/containers \
  quay.io/buildah/stable \
  sh -c 'buildah tag build_test1 \
    registry.example.com/build_test1 \
&&buildahlogin-u=<USERNAME> -p=<PASSWORD> \
    registry.example.com && \
    buildah push registry.example.com/build_test1'
```

When the image is successfully pushed, it is finally safe to remove the
volume:

``` {.programlisting .snippet-con}
# podman volume rm storevol
```

Despite []{#Chapter_7.xhtml#idx_7b3f12e1 .index-entry
index-entry="rootless Buildah containers:running, with volume stores"}working
perfectly, this approach[]{#Chapter_7.xhtml#idx_f4af9df5 .index-entry
index-entry="volume stores:rootless Buildah containers, running"} has
some limits that are worth discussing.

The first limit we can notice is that the store volume is not isolated,
and thus any other container can access its contents.[]{.sentence-end}
To overcome this issue, we can use SELinux\'s **Multi-Category
Security** (**MCS**) with[]{#Chapter_7.xhtml#idx_4163a59a .index-entry
index-entry="Multi-Category Security (MCS)"} the `:Z`{.inlineCode}
suffix in order to apply categories to the volume and make it accessible
exclusively to the running container.

However, since a second container would run by default with different
category labels, we should grab the volume categories and run the second
tag/push container with the
`--security-opt label=level`{.inlineCode}`:s0`{.inlineCode}`:`{.inlineCode}`<`{.inlineCode}`CAT1`{.inlineCode}`>`{.inlineCode}`,`{.inlineCode}`<`{.inlineCode}`CAT2`{.inlineCode}`>`{.inlineCode}
option.

Alternatively, we can just run `build`{.inlineCode}, `tag`{.inlineCode},
and `push`{.inlineCode} commands in one single container, as shown in
the following example:

``` {.programlisting .snippet-con}
$ podman run --device /dev/fuse \
    -v ~/build:/build \
    -v secure_storevol:/var/lib/containers:Z \
    quay.io/buildah/stable \
    sh -c 'buildah build -t test2 /build && \
      buildah tag test2 registry.example.com/build_test2 && \
      buildah login -u=<USERNAME> \
      -p=<PASSWORD> \
      registry.example.com && \
      buildah push registry.example.com/build_test2'
```

 packt_tip
**Important** **note**

In the preceding examples, we used the Buildah login by directly passing
the username and password in the command.[]{.sentence-end} Needless to
say, this is far from being an acceptable security practice.


Instead of []{#Chapter_7.xhtml#idx_cf4c6545 .index-entry
index-entry="rootless Buildah containers:running, with volume stores"}passing
sensitive data in the command[]{#Chapter_7.xhtml#idx_65897675
.index-entry
index-entry="volume stores:rootless Buildah containers, running"} line,
we can mount the authentication file that contains a valid session token
as a volume inside the container.

The next example mounts a valid `auth.json`{.inlineCode} file, stored
under the
`/run/user/`{.inlineCode}`<`{.inlineCode}`UID`{.inlineCode}`>`{.inlineCode}
`tmpfs`{.inlineCode}, inside the build container, and the
`--authfile /auth.json`{.inlineCode} option is then passed to the
`buildah push`{.inlineCode} command:

``` {.programlisting .snippet-con}
$ podman run --device /dev/fuse \
    -v ~/build:/build \
    -v /run/user/<UID>/containers/auth.json:/auth.json:z \
    -v secure_storevol:/var/lib/containers:Z \
    quay.io/buildah/stable \
    sh -c 'buildah build -t test3 /build && \
      buildah tag test3 registry.example.com/build_test3 && \
      buildah push --authfile /auth.json \
      registry.example.com/build_test3'
```

Finally, we have a working example that avoids exposing clear
credentials in the commands passed to the container.

To provide a working authentication file, we need to authenticate from
the host that will run the containerized build or copy a valid
authentication file.[]{.sentence-end} To authenticate with Podman,
we\'ll use the following command:

``` {.programlisting .snippet-con}
$ podman login -u <USERNAME> -p <PASSWORD> <REGISTRY>
```

If the authentication process succeeds, the obtained token is stored in
the
`/run/user/`{.inlineCode}`<`{.inlineCode}`UID`{.inlineCode}`>`{.inlineCode}`/containers/auth.json`{.inlineCode}
file, which stores a JSON-encoded object with a structure similar to the
following example:

``` {.programlisting .snippet-code}
{
      "auths": {
           "registry.example.com": {
                "auth": "<base64_encoded_token>"
               }
    }
}
```

 packt_tip
**Security** **alert!**

If the authentication file mounted inside the container has multiple
authentication records for different registries, they will be exposed
inside the build container.[]{.sentence-end} This can lead to potential
security issues, since the container will be able to authenticate on
those registries using the tokens specified in the file.


The []{#Chapter_7.xhtml#idx_347dca1c .index-entry
index-entry="rootless Buildah containers:running, with volume stores"}volume-based
approach we just described[]{#Chapter_7.xhtml#idx_5e87b407 .index-entry
index-entry="volume stores:rootless Buildah containers, running"} has a
small impact on the performance when compared to a native host build but
provides better isolation of the build process, a reduced attack surface
(thanks to the rootless execution), and standardization of the build
environment across different hosts.

Let\'s now inspect how to run containerized builds using bind-mounted
stores.

## Running Buildah containers with bind-mounted stores {#Chapter_7.xhtml#h2_191 .heading-2}

In the highest []{#Chapter_7.xhtml#idx_02ffd63f .index-entry
index-entry="Buildah containers:running, with bind-mounted stores"}isolation
scenario, where DevOps []{#Chapter_7.xhtml#idx_c2736ab1 .index-entry
index-entry="bind-mounted stores:Buildah containers, running"}teams
follow a zero-trust approach, every build container should have its own
isolated store populated at the beginning of the build and destroyed
upon completion.[]{.sentence-end} Isolation can be easily achieved with
SELinux MCS security.

To test this approach, let\'s start by creating a temporary directory
that will host the build layers.[]{.sentence-end} We also want to
generate a random suffix for a name in order to host multiple builds
without conflicts:

``` {.programlisting .snippet-con}
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)
# mkdir $BUILD_STORE
```

 note
**Important** **note**

The preceding example and the next builds are executed as root.


We can[]{#Chapter_7.xhtml#idx_858c118c .index-entry
index-entry="Buildah containers:running, with bind-mounted stores"} now
run the build and bind-mount []{#Chapter_7.xhtml#idx_7334fc4b
.index-entry
index-entry="bind-mounted stores:Buildah containers, running"}the new
directory to the `/var/lib/containers`{.inlineCode} folder inside the
container, as well as adding the `:Z`{.inlineCode} suffix to ensure
multi-category security isolation:

``` {.programlisting .snippet-con}
# podman run --device /dev/fuse \
    -v ./build:/build:z \
    -v $BUILD_STORE:/var/lib/containers:Z \
    -v /run/containers/0/auth.json:/auth.json \
    quay.io/buildah/stable \
    sh -c 'set -euo pipefail; \
      buildah build -t registry.example.com/test4 /build; \
      buildah push --authfile /auth.json \
      registry.example.com/test4'
```

The MCS isolation guarantees isolation from other
containers.[]{.sentence-end} Every build container will have its own
custom store, and this implies the need to re-pull the base image layers
on every execution, since they are never cached.

Despite being the most secure in terms of isolation, this approach also
offers the slowest performance because of the continuous pulls of the
base image during the build run.

On the other []{#Chapter_7.xhtml#idx_c4a16f14 .index-entry
index-entry="Buildah containers:running, with bind-mounted stores"}hand,
the less secure approach []{#Chapter_7.xhtml#idx_66586929 .index-entry
index-entry="bind-mounted stores:Buildah containers, running"}does not
expect any store isolation, and all the build containers mount the
default host store under
`/var/lib/containers`{.inlineCode}.[]{.sentence-end} This approach
provides better performance, since it allows the reuse of cached layers
from the host store.

SELinux will not allow a containerized process to access the host store;
therefore, we need to relax SELinux security restrictions to run the
following example using the `--security-opt label=disable`{.inlineCode}
option.

The following example runs another build using the default host store:

``` {.programlisting .snippet-con}
# podman run --device /dev/fuse \
  -v ./build:/build:z \
  -v /var/lib/containers:/var/lib/containers \
  --security-opt label=disable \
  -v /run/containers/0/auth.json:/auth.json \
  quay.io/buildah/stable \
  sh -c 'set -euo pipefail; \
    buildah build -t registry.example.com/test5 /build; \
    buildah push --authfile /auth.json \
    registry.example.com/test5'
```

The approach described in this example is the opposite of the previous
one -- better performances but worse security isolation.

A good compromise between the two implies the usage of a secondary,
read-only image store to provide access to the cached
layers.[]{.sentence-end} Buildah supports the usage of multiple image
stores, and the `/etc/containers/storage.conf`{.inlineCode} file *inside
the Buildah stable image* already configures the
`/var/lib/shared`{.inlineCode} folder for this purpose.

To prove this, we can inspect the content of the
`/etc/containers/storage.conf`{.inlineCode} file, where the following
section is defined:

``` {.programlisting .snippet-con}
# AdditionalImageStores is used to pass paths to additional Read/Only image stores
# Must be comma separated list.
additionalimagestores = [
"/var/lib/shared",
]
```

This way, we []{#Chapter_7.xhtml#idx_30cb364f .index-entry
index-entry="Buildah containers:running, with bind-mounted stores"}can
get good isolation and better[]{#Chapter_7.xhtml#idx_ef48093d
.index-entry
index-entry="bind-mounted stores:Buildah containers, running"}
performance, since cached images from the host will already be available
in the read-only store.[]{.sentence-end} The read-only store can be
pre-populated with the most used images to speed up builds, or can be
mounted from a network share.

The following example shows this approach, by bind-mounting the
read-only store to the container and executing the build with the
advantage of reusing pre-pulled images:

``` {.programlisting .snippet-con}
# podman run --device /dev/fuse \
  -v ./build:/build:z \
  -v $BUILD_STORE:/var/lib/containers:Z \
  -v /var/lib/containers/storage:/var/lib/shared:ro \
  -v /run/containers/0/auth.json:/auth.json:z \
  quay.io/buildah/stable \
  bash -c 'set -euo pipefail; \
  buildah build -t registry.example.com/test6 /build; \
  buildah push --authfile /auth.json \
  registry.example.com/test6'
```

The examples shown in this subsection are also inspired by a great
technical article written by *Dan Walsh* (one of the leads of the
Buildah and Podman projects) on the *Red Hat Developer* blog; refer to
the *Further reading* section for the original article
link.[]{.sentence-end} Let\'s close this section with an example of
native Buildah commands.

## Running native Buildah commands inside containers {#Chapter_7.xhtml#h2_192 .heading-2}

We have so far []{#Chapter_7.xhtml#idx_4b912fef .index-entry
index-entry="native Buildah commands:running, inside containers"}illustrated
examples using []{#Chapter_7.xhtml#idx_0ab12c46 .index-entry
index-entry="container:native Buildah commands, running"}Dockerfiles/Containerfiles,
but nothing prevents us from running containerized native Buildah
commands.[]{.sentence-end} The following example creates a custom Python
image built from a Fedora base image:

``` {.programlisting .snippet-con}
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)# mkdir $BUILD_STORE

# podman run --device /dev/fuse \
  -e REGISTRY=<USER_DEFINED_REGISTRY:PORT> \
  --security-opt label=disable \
  -v $BUILD_STORE:/var/lib/containers:Z \
  -v /var/lib/containers/storage:/var/lib/shared:ro \
  -v /run/containers/0:/run/containers/0 \
  quay.io/buildah/stable \
  sh -c 'set -euo pipefail; \
    container=$(buildah from fedora); \
    buildah run $container dnf install -y python3 python3; \
    buildah commit $container $REGISTRY/python_demo; \
    buildah push -authfile \
    /run/containers/0/auth.json $REGISTRY/python_demo'
```

From a performance standpoint, as well as the build process, nothing
changes from the previous examples.[]{.sentence-end} As already stated,
this approach provides more flexibility in the build operations.

If the []{#Chapter_7.xhtml#idx_225c1362 .index-entry
index-entry="native Buildah commands:running, inside containers"}commands
to be passed are too many, a good []{#Chapter_7.xhtml#idx_671d258a
.index-entry
index-entry="container:native Buildah commands, running"}workaround can
be to create a shell script and inject it into the Buildah image using a
dedicated volume:

``` {.programlisting .snippet-con}
# BUILD_STORE=/var/lib/containers-$(echo $RANDOM | md5sum | head -c 8)
# PATH_TO_SCRIPT=/path/to/script
# REGISTRY=<USER_DEFINED_REGISTRY:PORT>
# mkdir $BUILD_STORE
# podman run --device /dev/fuse \
  -v $BUILD_STORE:/var/lib/containers:Z \
  -v /var/lib/containers/storage:/var/lib/shared:ro \
  -v /run/containers/0:/run/containers/0 \
  -v $PATH_TO_SCRIPT:/root:z \
  quay.io/buildah/stable /root/build.sh
```

`build.sh`{.inlineCode} is the[]{#Chapter_7.xhtml#idx_f8e36e7e
.index-entry
index-entry="native Buildah commands:running, inside containers"} name
of the shell script file containing []{#Chapter_7.xhtml#idx_7d55e4ea
.index-entry
index-entry="container:native Buildah commands, running"}all the build
custom commands.

In this section, we have learned how to run Buildah in containers
covering both volume mounts and bind mounts.[]{.sentence-end} We have
learned how to run rootless build containers that can be easily
integrated into pipelines or Kubernetes clusters to provide an
end-to-end application life cycle workflow.[]{.sentence-end} This is due
to the flexible nature of Buildah, and for the same reason, it is very
easy to embed Buildah inside custom builders, as we will see in the next
section.

# Integrating Buildah with custom builders {#Chapter_7.xhtml#h1_193 .heading-1}

As we saw in the []{#Chapter_7.xhtml#idx_a85ea92e .index-entry
index-entry="Buildah:integrating, with custom builders"}previous section
of this chapter, Buildah is a key []{#Chapter_7.xhtml#idx_a87fa307
.index-entry
index-entry="custom builders:Buildah, integrating with"}component of
Podman\'s container ecosystem.[]{.sentence-end} Buildah is a dynamic and
flexible tool that can be adapted to different scenarios to build
brand-new containers.[]{.sentence-end} It has several options and
configurations available, but our exploration is not yet finished.

Podman and all the projects developed around it have been built with
extensibility in mind, making every programmable interface available to
be reused from the outside world.

Podman, for example, inherits Buildah capabilities for building
brand-new containers through the `podman build`{.inlineCode} command;
with the same principle, we can embed Buildah interfaces and its engine
in our custom builder.

Let\'s see how to build a custom builder in the Go language; we will see
that the process is pretty straightforward, because Podman, Buildah, and
many other projects in this ecosystem are actually written in the Go
language.

## Including Buildah in our Go build tool {#Chapter_7.xhtml#h2_194 .heading-2}

As a first step, we []{#Chapter_7.xhtml#idx_1eaf2ba2 .index-entry
index-entry="Buildah:including, in Go build tool"}need to prepare our
development environment, downloading []{#Chapter_7.xhtml#idx_db91c436
.index-entry index-entry="Go build tool:Buildah, including"}and
installing all the required tools and libraries for creating our custom
build tool.

 packt_tip
**Important** **note**

Please be aware that this is an unsupported scenario and the stability
in terms of deprecations and changes of the Buildah Go APIs is not
guaranteed.[]{.sentence-end} For supported and stable use cases, refer
to the Podman REST APIs.


In *[Chapter 3](#Chapter_3.xhtml#h1_83){.chapref}*, *Running* *the First
Container*, we saw various Podman installation methods.[]{.sentence-end}
In the following section, we will use a similar procedure while going
through the preliminary steps for building a Buildah project from
scratch, downloading its source file to include in our custom builder.

First of all, let\'s ensure we have all the needed packages installed on
our development host system:

``` {.programlisting .snippet-con}
# dnf install -y golang git go-md2man btrfs-progs-devel gpgme-devel device-mapper-devel
Fedora 40 - x86_64                                        253 kB/s |  28 kB     00:00  
Fedora 40 openh264 (From Cisco) - x86_64                  9.5 kB/s | 989  B     00:00  
Fedora 40 - x86_64 - Updates                              141 kB/s |  25 kB     00:00  
Fedora 40 - x86_64 - Updates                              3.9 MB/s | 6.2 MB     00:01  
Dependencies resolved.
==============================================================================================================================================================================================
 Package                                                Architecture                        Version              Repository                            Size
==============================================================================================================================================================================================
Installing:
 btrfs-progs-devel                                       x86_64        6.11-1.fc40          updates                               49 k
 device-mapper-devel                                      x86_64       1.02.199-1.fc40       updates                               41 k
 git                                                      x86_64       2.47.0-1.fc40         updates                               52 k
 golang                                                   x86_64        1.22.7-1.fc40         updates                              666 k
 golang-github-cpuguy83-md2man                            x86_64        2.0.3-3.fc40          fedora                               748 k
 gpgme-devel                                              x86_64        1.23.2-3.fc40         fedora                               167 k
U
                                                        
[... omitted output]
```

After installing[]{#Chapter_7.xhtml#idx_91069220 .index-entry
index-entry="Buildah:including, in Go build tool"} the Go language core
libraries and some other development []{#Chapter_7.xhtml#idx_9a13cdbf
.index-entry index-entry="Go build tool:Buildah, including"}tools, we
are ready to create the directory structure for our project and
initialize it:

``` {.programlisting .snippet-con}
$ mkdir ~/custombuilder
$ cd ~/custombuilder
[custombuilder]$ export GOPATH=`pwd`
```

As shown in the []{#Chapter_7.xhtml#idx_3c2fa2ff .index-entry
index-entry="Buildah:including, in Go build tool"}previous example, we
followed these steps:

1.  Created []{#Chapter_7.xhtml#idx_cd1ee337 .index-entry
    index-entry="Go build tool:Buildah, including"}the project root
    directory
2.  Defined the Go language root path that we are going to use

We are now ready to create our Go module that will create our customized
container image with a few easy steps.

To speed up the example and avoid any writing errors, we can download
the Go language code that we are going to use for this test from the
official GitHub repository of this book:

1.  Go to
    [[https://github.com/PacktPublishing/Podman-for-DevOps]{.url}](https://github.com/PacktPublishing/Podman-for-DevOps){style="text-decoration: none;"}
    or run the following command:

    ``` {.programlisting .snippet-con-one}
    $ git clone https://github.com/PacktPublishing/Podman-for-DevOps
    ```

2.  After that, copy the files provided in the
    `Chapter07/*`{.inlineCode} directory into the newly created
    `~/custombuilder/`{.inlineCode} directory.[]{.sentence-end}

    You should have the following files in your directory at this point:

    ``` {.programlisting .snippet-con-one}
    $ cd ~/custombuilder/src/builder
    $ ls -la   
    total 148
    drwxrwxr-x. 1 alex alex     74 9 nov 15.22 .
    drwxrwxr-x. 1 alex alex     14 9 nov 14.10 ..
    -rw-rw-r--. 1 alex alex   1466 9 nov 14.10 custombuilder.go
    -rw-rw-r--. 1 alex alex    161 9 nov 15.22 go.mod
    -rw-rw-r--. 1 alex alex 135471 9 nov 15.22 go.sum
    -rw-rw-r--. 1 alex alex    337 9 nov 14.17 script.js
    ```

    At this point, we can run the following command to let the Go tools
    acquire all the needed dependencies to ready the module for
    execution:

    ``` {.programlisting .snippet-con-one}
    $ go mod tidy -v
    go: finding module for package github.com/containers/storage/pkg/unshare
    go: finding module for package github.com/containers/image/v5/storage[...omitted output...]
    ```

    The tool analyzed the provided `custombuilder.go`{.inlineCode} file,
    and it found all the required libraries, populating the
    `go.mod`{.inlineCode} file.

    ::: packt_tip-one
    **Important** **note**

    Please be aware that the previous command will verify whether a
    module is available, and if it is not, the tool will start
    downloading it from the internet.[]{.sentence-end} So, be patient
    during this step!
    :::

    We can check []{#Chapter_7.xhtml#idx_52544037 .index-entry
    index-entry="Buildah:including, in Go build tool"}that the previous
    commands downloaded all the[]{#Chapter_7.xhtml#idx_c4953ae8
    .index-entry index-entry="Go build tool:Buildah, including"}
    required packages by inspecting the directory structure we created
    earlier:

    ``` {.programlisting .snippet-con-one}
    $ cd ~/custombuilder
    [custombuilder]$ ls
    pkg  src
    [custombuilder]$ ls -la pkg/
    total 0
    drwxrwxr-x. 1 alex alex  28  9 nov 18.27 .
    drwxrwxr-x. 1 alex alex  12  9 nov 18.18 ..
    drwxrwxr-x. 1 alex alex  20  9 nov 18.27 linux_amd64
    drwxrwxr-x. 1 alex alex 196  9 nov 18.27 mod
    [custombuilder]$ ls -la pkg/mod/
    total 0
    drwxrwxr-x. 1 alex alex 196  9 nov 18.27 .
    drwxrwxr-x. 1 alex alex  28  9 nov 18.27 ..
    drwxrwxr-x. 1 alex alex  22  9 nov 18.18 cache
    drwxrwxr-x. 1 alex alex 918  9 nov 18.27 github.com
    drwxrwxr-x. 1 alex alex  24  9 nov 18.27 go.etcd.io
    drwxrwxr-x. 1 alex alex   2  9 nov 18.27 golang.org
    [... omitted output]
    [custombuilder]$ ls -la pkg/mod/github.com/
    [... omitted output]
    drwxrwxr-x. 1 alex alex  98  9 nov 18.27  containerd
    drwxrwxr-x. 1 alex alex  20  9 nov 18.27  containernetworking
    drwxrwxr-x. 1 alex alex 184  9 nov 18.27  containers
    drwxrwxr-x. 1 alex alex 110  9 nov 18.27  coreos
    [... omitted output]
    ```

We are now[]{#Chapter_7.xhtml#idx_d63aac82 .index-entry
index-entry="Buildah:including, in Go build tool"} ready to run our
custom builder module, but before[]{#Chapter_7.xhtml#idx_9ecc76c0
.index-entry index-entry="Go build tool:Buildah, including"} moving
forward, let\'s take a look at the key elements contained in the Go
source file.

 packt_tip
**Important** **note**

Please consider that on Fedora 40, as well as for other Linux
distributions, you may also need additional development packages from
your distribution\'s repositories.[]{.sentence-end} For Fedora, for
example, in order to successfully run the Go program, you may also need
to install the `btrfs-progs-devel`{.inlineCode} and
`libgpgme-devel`{.inlineCode} packages.[]{.sentence-end} Please refer to
Podman\'s documentation for more info:

[[https://podman.io/docs/installation#build-and-run-dependencies]{.url}](https://podman.io/docs/installation#build-and-run-dependencies){style="text-decoration: none;"}


If we start looking at the `custombuilder.go`{.inlineCode} file, just
after defining the package and the libraries to use, we define the main
function of our module.

In the main function, at the beginning of the definition, we inserted a
fundamental code block:

``` {.programlisting .snippet-code}
  if buildah.InitReexec() {
    return
  }
  unshare.MaybeReexecUsingUserNamespace(false)
```

This piece []{#Chapter_7.xhtml#idx_d7507867 .index-entry
index-entry="Buildah:including, in Go build tool"}of code enables the
usage of **rootless** mode[]{#Chapter_7.xhtml#idx_c15a1f25 .index-entry
index-entry="rootless mode"} by []{#Chapter_7.xhtml#idx_44f82dba
.index-entry index-entry="Go build tool:Buildah, including"}leveraging
the Go `unshare`{.inlineCode} package, available through
`github.com/containers/storage/pkg/unshare`{.inlineCode}.

To leverage the build features of Buildah, we have to instantiate
`buildah.Builder`{.inlineCode}.[]{.sentence-end} This object has all the
methods to define the build steps, configure the build, and finally run
it.

To create `Builder`{.inlineCode}, we need an object called
`storage.Store`{.inlineCode} from the
`github.com/containers/storage`{.inlineCode} package.[]{.sentence-end}
This element is responsible for storing the intermediate and result
container images.[]{.sentence-end} Let\'s see the code block we are
discussing:

``` {.programlisting .snippet-code}
buildStoreOptions, err := storage.DefaultStoreOptions(     )
buildStore, err := storage.GetStore(buildStoreOptions)
```

As you can see from the previous example, we are getting the default
options and passing them to the `storage`{.inlineCode} module to request
a `Store`{.inlineCode} object.

Another element we need for creating `Builder`{.inlineCode} is the
`BuilderOptions`{.inlineCode} object.[]{.sentence-end} This element
contains all the default and custom options we might assign to
Buildah\'s `Builder`{.inlineCode}.[]{.sentence-end} Let\'s see how to
define it:

``` {.programlisting .snippet-code}
builderOpts := buildah.BuilderOptions{
  FromImage:        "node:23-alpine", // Starting image
  Isolation:        define.IsolationChroot, // Isolation environment
  CommonBuildOpts:  &define.CommonBuildOptions{},
  ConfigureNetwork: define.NetworkDefault,
  SystemContext:    &types.SystemContext {},
}
```

In the previous code block, we defined a `BuilderOptions`{.inlineCode}
object that contains the following:

-   An initial image that we are going to use to build our target
    container image:
    -   In this case, we chose the Node.js image based on the Alpine
        Linux distribution.[]{.sentence-end} This is because, in our
        example, we are simulating the build process of a Node.js
        application.
-   Isolation mode to adopt once the build starts.[]{.sentence-end} In
    this case, we are going to use chroot isolation, which fits a lot of
    build scenarios well -- less isolation but fewer requirements.
-   Some default options for the build, network, and system contexts:
    -   `SystemContext`{.inlineCode} objects define the information
        contained in configuration files as parameters

Now that we have all the necessary data for instantiating
`Builder`{.inlineCode}, let\'s do it:

``` {.programlisting .snippet-code}
builder, err := buildah.NewBuilder(context.TODO(), buildStore, builderOpts)
```

As you can see, we []{#Chapter_7.xhtml#idx_62e843e2 .index-entry
index-entry="Buildah:including, in Go build tool"}are calling the
`NewBuilder`{.inlineCode} function, with all the
[]{#Chapter_7.xhtml#idx_75e86ef8 .index-entry
index-entry="Go build tool:Buildah, including"}required options that we
created in code earlier in this section, to get `Builder`{.inlineCode}
ready to create our custom container image.

We are now ready to instruct `Builder`{.inlineCode} with the required
options to create the custom image.[]{.sentence-end} Let\'s first add to
the container image the **JavaScript**
file[]{#Chapter_7.xhtml#idx_689a427f .index-entry
index-entry="JavaScript file"} containing our application, for which we
are creating this container image:

``` {.programlisting .snippet-code}
err = builder.Add("/home/node/", false, buildah.AddAndCopyOptions{}, "script.js")
```

We are assuming that the JavaScript main file is stored next to the Go
module that we are writing and using in this example, and we are copying
this file into the `/home/node`{.inlineCode} directory, which is the
default path where the base container image expects to find this kind of
data.

The JavaScript program that we are going to copy into the container
image and use for this test is really simple -- let\'s inspect it:

``` {.programlisting .snippet-code}
var http = require("http");
http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello Podman and Buildah friends. This page is provided to you through a container running Node.js version: ");
  response.write(process.version);
  response.end();
}).listen(8080);
```

Without going deep[]{#Chapter_7.xhtml#idx_3d87ba91 .index-entry
index-entry="Buildah:including, in Go build tool"} into the JavaScript
language syntax and its concepts, we can note, by looking at the
JavaScript file, that we are using the HTTP library for listening on
port `8080`{.inlineCode} for incoming requests, responding to these
requests with a default welcome message:
`Hello Podman and Buildah friends.`{.inlineCode}[]{.sentence-end}` This page is `{.inlineCode}`provided to you through a container running Node.js`{.inlineCode}.[]{.sentence-end}
We also append the Node.js version to the response string.

 note
**Important** **note**

Please consider []{#Chapter_7.xhtml#idx_9fdac81e .index-entry
index-entry="JavaScript"}that **JavaScript**, also known as **JS**, is a
high-level programming language that is compiled just in
time.[]{.sentence-end} As we stated earlier, we are not going to go deep
into the definition of the JavaScript language or its most famous
runtime environment, Node.js.


After that, we configure the default command to run for our custom
container image:

``` {.programlisting .snippet-code}
builder.SetCmd([]string{"node", "/home/node/script.js"})
```

We just set the []{#Chapter_7.xhtml#idx_ef92c173 .index-entry
index-entry="Go build tool:Buildah, including"}command to execute the
Node.js execution runtime, referring to the JavaScript program that we
just added to the container image.

To commit the changes we made, we need to get the image reference that
we are working on.[]{.sentence-end} At the same time, we will also
define the name of the container image that `Builder`{.inlineCode} will
create:

``` {.programlisting .snippet-code}
imageRef, err := is.Transport.ParseStoreReference(buildStore, "podmanbook/nodejs-welcome")
```

Now, we are ready to commit the changes and call the
`commit`{.inlineCode} function of `Builder`{.inlineCode}:

``` {.programlisting .snippet-code}
imageId, _, _, err := builder.Commit(context.TODO(), imageRef, define.CommitOptions{})
fmt.Printf("Image built! %s\n", imageId)
```

As we can see, we just requested `Builder`{.inlineCode} to commit the
changes, passing the image reference we obtained earlier, and then we
finally print it as a reference.

We are now ready to []{#Chapter_7.xhtml#idx_dd943bff .index-entry
index-entry="Buildah:including, in Go build tool"}run our program!
Let\'s execute it:

``` {.programlisting .snippet-con}
[builder]$ go run custombuilder.go
Image built! e60fa98051522a51f4585e46829ad6a18df704dde774634dbc010baae4404849
```

We can now test the custom container image we just built:

``` {.programlisting .snippet-con}
[builder]$ podman run -dt -p 8080:8080/tcp podmanbook/nodejs-welcome:latest
747805c1b59558a70c4a2f1a1d258913cae5ffc08cc026c74ad3ac21aab18974
[builder]$ curl localhost:8080
Hello Podman and Buildah friends. This page is provided to you through a container running Node.js version: v23.1.0
```

As we can see in the previous []{#Chapter_7.xhtml#idx_415c8f5f
.index-entry index-entry="Go build tool:Buildah, including"}code block,
we are running the container image we just created with the following
options:

-   `-d`{.inlineCode}: Detached mode, which runs the container in the
    background
-   `-t`{.inlineCode}: Allocates a new pseudo-TTY
-   `-p`{.inlineCode}: Publishes the container port to the host system
-   `podmanbook/nodejs-welcome:latest`{.inlineCode}: The name of our
    custom container image

Finally, we use the `curl`{.inlineCode} command-line tool for requesting
and printing the HTTP response provided by our JavaScript program, which
is containerized in the custom container image that we created!

 note
**Important** **note**

The example described in this section is just a simple overview of all
the great features that the Buildah Go module can enable for our custom
image builders.[]{.sentence-end} To learn more about the various
functions, variables, and code documentation, you can refer to the docs
at
[[https://pkg.go.dev/github.com/containers/buildah]{.url}](https://pkg.go.dev/github.com/containers/buildah){style="text-decoration: none;"}.


As we saw in this section, Buildah is a really flexible tool, and with
its libraries, it can support custom builders in many different
scenarios.

If we try to search on the internet, we can find many examples of
Buildah supporting the creation of custom container
images.[]{.sentence-end} Let\'s see some of them.

## Running Quarkus-native executables in containers {#Chapter_7.xhtml#h2_195 .heading-2}

**Quarkus** is []{#Chapter_7.xhtml#idx_1ab23d45 .index-entry
index-entry="Quarkus"}defined as the Kubernetes-native Java stack
leveraging the OpenJDK (the open Java development kit) project and the
GraalVM project.[]{.sentence-end} GraalVM is a Java virtual machine that
has many special features, such as the compilation of Java applications
for fast startup and a low memory footprint.

 note
**Important note**

We will not go into the details of Quarkus, GraalVM, and any other
companion projects.[]{.sentence-end} The example that we will deep-dive
into is only for your reference.[]{.sentence-end} We encourage you to
learn more about these projects by going through their web pages and
reading the related documentation.


If we take a []{#Chapter_7.xhtml#idx_a3c2929f .index-entry
index-entry="Quarkus-native executables:running, in container"}look at
the Quarkus documentation web []{#Chapter_7.xhtml#idx_d3ed306a
.index-entry
index-entry="container:Quarkus-native executables, running"}page, we can
easily find that, after a long tutorial in which we can learn how to
build a Quarkus-native executable, we can then pack and execute this
executable in a container image.

The steps provided in the Quarkus documentation leverage a Maven wrapper
with a special option.[]{.sentence-end} Maven was created as a Java
build automation tool, but then it was also extended to other
programming languages.[]{.sentence-end} If we take a quick look at this
command, we will notice that `p`{.inlineCode}`odman`{.inlineCode} is
mentioned:

``` {.programlisting .snippet-con}
$ ./mvnw package -Pnative -Dquarkus.native.container-build=true -Dquarkus.native.container-runtime=podman
```

This means[]{#Chapter_7.xhtml#idx_fd661ade .index-entry
index-entry="Quarkus-native executables:running, in container"} that the
Maven wrapper program will []{#Chapter_7.xhtml#idx_e03c28c9 .index-entry
index-entry="container:Quarkus-native executables, running"}invoke a
Podman build to create a container image with the preconfigured
environment shipped by the Quarkus project and the binary application
that we are developing.

Podman is mentioned because, as we saw in *[Chapter
6](#Chapter_6.xhtml#h1_163){.chapref}*, *Meet Buildah -- Building
Containers from Scratch*, it borrows Buildah\'s build logic by vendoring
its libraries.

To explore this example further, take a look at
[[https://quarkus.io/guides/building-native-image]{.url}](https://quarkus.io/guides/building-native-image){style="text-decoration: none;"}.

# Summary {#Chapter_7.xhtml#h1_196 .heading-1}

In this chapter, we have learned how to leverage Podman\'s companion,
Buildah, in some advanced scenarios to support our development projects.

We saw how to use Buildah for multistage container image creation, which
allows us to create builds with multiple stages using different
`FROM`{.inlineCode} instructions and, subsequently, to have images that
grab contents from the previous ones.

Then, we discovered that there are many use cases that imply the need
for containerized builds.[]{.sentence-end} Nowadays, one of the most
common adoption scenarios is the application build workflow running on
top of a Kubernetes cluster.[]{.sentence-end} For this reason, we went
into the details of containerizing Buildah.

Finally, we learned, through a lot of interesting examples, how to
integrate Buildah to create custom builders for container
images.[]{.sentence-end} As we saw in this chapter, there are several
options and methods to actually build a container image with the Podman
ecosystem tools.[]{.sentence-end} Most of the time, we start from a base
image to customize and extend a previous OS layer to fit our use cases.

In the next chapter, we will learn more about container base images, how
to choose them, and what to look out for when we are making our choice.

# Further reading

-   A list of CNCF projects:
    https://landscape.cncf.io/
-   Best practices for running Buildah in a container:
    https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container
-   The Buildah Go module documentation:
    https://pkg.go.dev/github.com/containers/buildah
-   Quarkus-native executables:
    https://quarkus.io/guides/building-native-image
