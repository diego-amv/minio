> [!NOTE]
> **THIS REPOSITORY IS NO LONGER MAINTAINED.**
>
> **Alternatives:**
> - **[AIStor Free](https://min.io/download)** — Full-featured, standalone edition for community use (free license)
> - **[AIStor Enterprise](https://min.io/pricing)** — Distributed edition with commercial support

> [!TIP]
> **OpenShift:** this fork adds a source build and Helm values for OpenShift. See [Deploy on OpenShift](#deploy-on-openshift).

---

# MinIO Quickstart Guide

[![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io) [![Docker Pulls](https://img.shields.io/docker/pulls/minio/minio.svg?maxAge=604800)](https://hub.docker.com/r/minio/minio/) [![license](https://img.shields.io/badge/license-AGPL%20V3-blue)](https://github.com/minio/minio/blob/master/LICENSE)

[![MinIO](https://raw.githubusercontent.com/minio/minio/master/.github/logo.svg?sanitize=true)](https://min.io)

MinIO is a high-performance, S3-compatible object storage solution released under the GNU AGPL v3.0 license.
Designed for speed and scalability, it powers AI/ML, analytics, and data-intensive workloads with industry-leading performance.

- S3 API Compatible – Seamless integration with existing S3 tools
- Built for AI & Analytics – Optimized for large-scale data pipelines
- High Performance – Ideal for demanding storage workloads.

This README provides instructions for building MinIO from source and deploying onto baremetal hardware.
Use the [MinIO Documentation](https://github.com/minio/docs) project to build and host a local copy of the documentation.

## MinIO is Open Source Software

We designed MinIO as Open Source software for the Open Source software community. We encourage the community to remix, redesign, and reshare MinIO under the terms of the AGPLv3 license.

All usage of MinIO in your application stack requires validation against AGPLv3 obligations, which include but are not limited to the release of modified code to the community from which you have benefited. Any commercial/proprietary usage of the AGPLv3 software, including repackaging or reselling services/features, is done at your own risk.

The AGPLv3 provides no obligation by any party to support, maintain, or warranty the original or any modified work.
All support is provided on a best-effort basis through Github and our [Slack](https://slack.min.io) channel, and any member of the community is welcome to contribute and assist others in their usage of the software.

MinIO [AIStor](https://www.min.io/product/aistor) includes enterprise-grade support and licensing for workloads which require commercial or proprietary usage and production-level SLA/SLO-backed support. For more information, [reach out for a quote](https://min.io/pricing).

## Source-Only Distribution

**Important:** The MinIO community edition is now distributed as source code only. We will no longer provide pre-compiled binary releases for the community version.

### Installing Latest MinIO Community Edition

To use MinIO community edition, you have two options:

1. **Install from source** using `go install github.com/minio/minio@latest` (recommended)
2. **Build a Docker image** from the provided Dockerfile

See the sections below for detailed instructions on each method.

### Legacy Binary Releases

Historical pre-compiled binary releases remain available for reference but are no longer maintained:

- GitHub Releases: https://github.com/minio/minio/releases
- Direct downloads: https://dl.min.io/server/minio/release/

**These legacy binaries will not receive updates.** We strongly recommend using source builds for access to the latest features, bug fixes, and security updates.

## Install from Source

Use the following commands to compile and run a standalone MinIO server from source.
If you do not have a working Golang environment, please follow [How to install Golang](https://golang.org/doc/install). Minimum version required is [go1.24](https://golang.org/dl/#stable)

```sh
go install github.com/minio/minio@latest
```

You can alternatively run `go build` and use the `GOOS` and `GOARCH` environment variables to control the OS and architecture target.
For example:

```
env GOOS=linux GOARCH=arm64 go build
```

Start MinIO by running `minio server PATH` where `PATH` is any empty folder on your local filesystem.

The MinIO deployment starts using default root credentials `minioadmin:minioadmin`.
You can test the deployment using the MinIO Console, an embedded web-based object browser built into MinIO Server.
Point a web browser running on the host machine to <http://127.0.0.1:9000> and log in with the root credentials.
You can use the Browser to create buckets, upload objects, and browse the contents of the MinIO server.

You can also connect using any S3-compatible tool, such as the MinIO Client `mc` commandline tool:

```sh
mc alias set local http://localhost:9000 minioadmin minioadmin
mc admin info local
```

See [Test using MinIO Client `mc`](#test-using-minio-client-mc) for more information on using the `mc` commandline tool.
For application developers, see <https://docs.min.io/enterprise/aistor-object-store/developers/sdk/> to view MinIO SDKs for supported languages.

> [!NOTE]
> Production environments using compiled-from-source MinIO binaries do so at their own risk.
> The AGPLv3 license provides no warranties nor liabilities for any such usage.

## Build Docker Image

You can use the `docker build .` command to build a Docker image on your local host machine.
You must first [build MinIO](#install-from-source) and ensure the `minio` binary exists in the project root.

The following command builds the Docker image using the default `Dockerfile` in the root project directory with the repository and image tag `myminio:minio`

```sh
docker build -t myminio:minio .
```

Use `docker image ls` to confirm the image exists in your local repository.
You can run the server using standard Docker invocation:

```sh
docker run -p 9000:9000 -p 9001:9001 myminio:minio server /tmp/minio --console-address :9001
```

Complete documentation for building Docker containers, managing custom images, or loading images into orchestration platforms is out of scope for this documentation.
You can modify the `Dockerfile` and `dockerscripts/docker-entrypoint.sh` as-needed to reflect your specific image requirements.

See the [MinIO Container](https://docs.min.io/community/minio-object-store/operations/deployments/baremetal-deploy-minio-as-a-container.html#deploy-minio-container) documentation for more guidance on running MinIO within a Container image.

## Install using Helm Charts

There are two paths for installing MinIO onto Kubernetes infrastructure:

- Use the [MinIO Operator](https://github.com/minio/operator)
- Use the community-maintained [Helm charts](https://github.com/minio/minio/tree/master/helm/minio)

See the [MinIO Documentation](https://docs.min.io/community/minio-object-store/operations/deployments/kubernetes.html) for guidance on deploying using the Operator.
The Community Helm chart has instructions in the folder-level README.

## Deploy on OpenShift

This fork builds MinIO and the MinIO Client (`mc`) **from source inside OpenShift** and deploys them with the Helm chart in [`helm/minio`](helm/minio).
Upstream no longer publishes community binaries or images, so the other Dockerfiles in this repository (which download prebuilt binaries or extend `minio/minio` images) cannot produce new images.

| File | Purpose |
|------|---------|
| [`Dockerfile.ocp`](Dockerfile.ocp) | Multi-stage build: compiles `minio` and `mc` on `ubi9/go-toolset:1.25`, ships them on `ubi9/ubi-micro`. Runs with any UID (GID 0), as required by the `restricted-v2` SCC. |
| [`ocp/minio-build.yaml`](ocp/minio-build.yaml) | `ImageStream` + `BuildConfig` (Docker strategy) that builds `Dockerfile.ocp` from this fork. |
| [`helm/minio/values-openshift.yaml`](helm/minio/values-openshift.yaml) | Chart overrides: internal registry image for MinIO and the post-install Job, no fixed UID/GID, standalone sizing. |

### Prerequisites

- OpenShift 4.x project where you can create builds, and the `oc` and `helm` (v3+) CLIs.
- Build pod resources: the `BuildConfig` requests 4 GiB of memory (6 GiB limit). Compiling MinIO with less memory may get the build `OOMKilled`; lower `GO_BUILD_JOBS` or raise the limits.
- Egress from build pods to `github.com` (source and `mc` module), `proxy.golang.org` (Go modules) and `registry.access.redhat.com` (base images).
  Disconnected clusters need mirrors for these; `Dockerfile.ocp` does not configure a custom `GOPROXY` yet.

### 1. Build the image

```sh
oc new-project minio            # or: oc project <existing-namespace>
oc apply -f ocp/minio-build.yaml
oc start-build minio --follow
```

The build compiles `mc` first, then `minio`, and pushes `minio:latest` to the `minio` ImageStream.
To build from another fork or branch, edit `spec.source.git` in `ocp/minio-build.yaml`.
To build your local working copy instead of the Git source:

```sh
oc start-build minio --from-dir=. --follow
```

### 2. Install the Helm chart

Create the root credentials as a Secret instead of passing them on the command line:

```sh
oc create secret generic minio-root \
  --from-literal=rootUser=<root-user> \
  --from-literal=rootPassword=<root-password>
```

Install the chart, pointing both images at the ImageStream in the current namespace:

```sh
NS=$(oc project -q)
IMAGE=image-registry.openshift-image-registry.svc:5000/$NS/minio

helm install minio ./helm/minio -n "$NS" \
  -f helm/minio/values-openshift.yaml \
  --set image.repository="$IMAGE" \
  --set mcImage.repository="$IMAGE" \
  --set existingSecret=minio-root
```

`values-openshift.yaml` deploys a single standalone instance requesting 2 GiB of memory.
For a distributed deployment, add for example `--set mode=distributed --set replicas=4` and size `resources` and `persistence` for your workload.
Buckets, users and policies listed in `buckets`, `users` and `policies` are created by the chart's post-install Job, which uses the `mc` binary shipped in the same image.

If the chart is installed in a different namespace than the ImageStream, allow its service account (`minio-sa` by default) to pull the image:

```sh
oc policy add-role-to-user system:image-puller \
  system:serviceaccount:<app-namespace>:minio-sa -n <build-namespace>
```

> [!IMPORTANT]
> Keep `securityContext.enabled: false` on OpenShift. Enabling it sets fixed UID/GID `1000`, which the `restricted-v2` SCC rejects, and renders a cluster-scoped `SecurityContextConstraints` object that only cluster administrators can create.

### 3. Expose and verify

```sh
oc create route edge minio-api --service=minio
oc create route edge minio-console --service=minio-console

oc get pods -l app=minio
oc exec deploy/minio -- sh -c \
  'mc alias set local http://localhost:9000 "$MINIO_ROOT_USER" "$MINIO_ROOT_PASSWORD" && mc admin info local'
```

### Versioning

The `BuildConfig` pins `MINIO_VERSION` to the UTC commit time of the last upstream commit included in this fork, so `minio --version` matches the chart's `appVersion` (`DEVELOPMENT.<timestamp>`).
When merging upstream changes, update both values together:

```sh
TZ=UTC git log -1 --format=%cd --date=iso-strict-local origin/master
# e.g. 2026-02-12T20:18:48Z
#   ocp/minio-build.yaml    -> MINIO_VERSION: "2026-02-12T20:18:48Z"
#   helm/minio/Chart.yaml   -> appVersion: DEVELOPMENT.2026-02-12T20-18-48Z
```

If `MINIO_VERSION` is not set, `Dockerfile.ocp` uses the build time. `COMMIT_ID` defaults to `OPENSHIFT_BUILD_COMMIT` when OpenShift provides it, otherwise `unknown`.

### Build locally with Podman

```sh
podman build -f Dockerfile.ocp \
  --build-arg MINIO_VERSION=2026-02-12T20:18:48Z \
  --build-arg COMMIT_ID="$(git rev-parse HEAD)" \
  -t minio:ocp .

podman run --rm --user 1000680000:0 -p 9000:9000 \
  -e MINIO_ROOT_USER=<root-user> -e MINIO_ROOT_PASSWORD=<root-password> \
  minio:ocp server /data
```

On Git Bash for Windows, prefix `podman run` with `MSYS_NO_PATHCONV=1`; otherwise `/data` is rewritten to a Windows path and MinIO fails with `file access denied`.

### Troubleshooting

| Symptom | Cause / fix |
|---------|-------------|
| Build pod `OOMKilled` or killed during `go build` | Not enough memory for the compiler. Raise `resources.limits.memory` in the `BuildConfig` or lower `GO_BUILD_JOBS`. |
| `ImagePullBackOff` | Wrong namespace in `image.repository` / `mcImage.repository`, or missing `system:image-puller` role across namespaces. |
| Pod rejected with `unable to validate against any security context constraint` | `securityContext.enabled` is `true`; set it to `false`. |
| Post-install Job fails with `/usr/bin/mc: not found` | `mcImage` points to an image without `mc`; use the image built from `Dockerfile.ocp`. |
| Container exits with `exec ... docker-entrypoint.sh: No such file or directory` | Entrypoint with CRLF line endings, from an image built with another Dockerfile on a Windows checkout. `Dockerfile.ocp` converts it automatically. |

### Limitations

- **Maintenance:** upstream MinIO is no longer maintained and `minio/mc` is archived on GitHub. `mc` is pinned to its last release (`RELEASE.2025-08-13T08-35-41Z`). Security fixes for both, and for the Helm chart, are the responsibility of this fork.
- **Go toolchain:** Red Hat does not ship `go-toolset` 1.24.8 or later (the `go.mod` toolchain), so the build uses Go 1.25.
- **License:** MinIO and `mc` are AGPLv3. Review your obligations before distributing modified images or offering them as a service.
- **Validation status:** the image and the chart's `mc` scripts (bucket/user creation) were validated locally with Podman and `helm template` using arbitrary UIDs; validate on your target OpenShift version before production use.

## Test MinIO Connectivity

### Test using MinIO Console

MinIO Server comes with an embedded web based object browser.
Point your web browser to <http://127.0.0.1:9000> to ensure your server has started successfully.

> [!NOTE]
> MinIO runs console on random port by default, if you wish to choose a specific port use `--console-address` to pick a specific interface and port.

### Test using MinIO Client `mc`

`mc` provides a modern alternative to UNIX commands like ls, cat, cp, mirror, diff etc. It supports filesystems and Amazon S3 compatible cloud storage services.

The following commands set a local alias, validate the server information, create a bucket, copy data to that bucket, and list the contents of the bucket.

```sh
mc alias set local http://localhost:9000 minioadmin minioadmin
mc admin info
mc mb data
mc cp ~/Downloads/mydata data/
mc ls data/
```

Follow the MinIO Client [Quickstart Guide](https://docs.min.io/community/minio-object-store/reference/minio-mc.html#quickstart) for further instructions.

## Explore Further

- [The MinIO documentation website](https://docs.min.io/community/minio-object-store/index.html)
- [MinIO Erasure Code Overview](https://docs.min.io/community/minio-object-store/operations/concepts/erasure-coding.html)
- [Use `mc` with MinIO Server](https://docs.min.io/community/minio-object-store/reference/minio-mc.html)
- [Use `minio-go` SDK with MinIO Server](https://docs.min.io/enterprise/aistor-object-store/developers/sdk/go/)

## Contribute to MinIO Project

Please follow MinIO [Contributor's Guide](https://github.com/minio/minio/blob/master/CONTRIBUTING.md) for guidance on making new contributions to the repository.

## License

- MinIO source is licensed under the [GNU AGPLv3](https://github.com/minio/minio/blob/master/LICENSE).
- MinIO [documentation](https://github.com/minio/minio/tree/master/docs) is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- [License Compliance](https://github.com/minio/minio/blob/master/COMPLIANCE.md)
