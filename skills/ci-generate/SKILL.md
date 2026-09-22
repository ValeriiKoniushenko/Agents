---
name: ci-generate
description: use this skill when the user ask to create, modify, or improve the CI (gitea) infrastructure and/or some functionality. Write or update project's Gitea Actions CI, primarily .gitea/workflows/code-quality.yml. This skill mostly connected to PR validation on the CI level for self-hosted Gitea.
---

# Introduction

The CI is located on self hosted machine. It has 16 cores and 24 threads and 32 GB RAM.

The CI runner is Gitea 1.27.0. And it has it's own dockerfile for all CI jobs. Now, it has only CI for pull-requests (PR), nothing more or less. This docker file is:

```dockerfile
FROM debian:13

RUN apt-get update && \
 apt-get install -y --no-install-recommends \
 wget \
 gnupg \
 ca-certificates && \
 mkdir -p /etc/apt/keyrings && \
 wget -O- https://apt.llvm.org/llvm-snapshot.gpg.key | tee /etc/apt/keyrings/apt.llvm.org.asc > /dev/null && \
 echo "deb [signed-by=/etc/apt/keyrings/apt.llvm.org.asc] https://apt.llvm.org/trixie/ llvm-toolchain-trixie-22 main" \

> /etc/apt/sources.list.d/llvm-22.list && \
>  apt-get update

RUN apt-get install -y --no-install-recommends \
 build-essential \
 cmake \
 ninja-build \
 gcc \
 curl \
 llvm \
 git \
 ccache \
 libgtest-dev \
 gdb \
 nodejs \
 npm \
 libwayland-dev \
 wayland-protocols \
 libxkbcommon-dev \
 libxcursor-dev \
 libxi-dev \
 libxinerama-dev \
 libxrandr-dev \
 libgl1-mesa-dev \
 libvulkan-dev \
 libgl1-mesa-dri \
 libglx-mesa0 \
 rsync \
 valgrind \
 libc6-dbg \
 mold \
 pkg-config \
 xvfb \
 scrot \
 lcov \
 python3 \
 python3-pip \
 clang-22 \
 clang-tidy-22 \
 clang-format-22 \
 clang-tools-22 \
 && update-alternatives --install /usr/bin/clang clang /usr/bin/clang-22 100 \
 && update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-22 100 \
 && update-alternatives --install /usr/bin/clang-tidy clang-tidy /usr/bin/clang-tidy-22 100 \
 && update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-22 100 \
 && rm -rf /var/lib/apt/lists/\* \
 && pip install --no-cache-dir --break-system-packages "gcovr>=8.6"

RUN git config --system init.defaultBranch develop

ENV PATH="/usr/lib/ccache:$PATH"
ENV CCACHE_DIR=/ccache
ENV CCACHE_MAXSIZE=25G
ENV CCACHE_COMPRESS=true
ENV CCACHE_SLOPPINESS=include_file_mtime,include_file_ctime,time_macros,locale
```

And of course it has the Gitea runner file `docker/gitea-runner/runner/config.yaml`:
```yaml
log:
  level: info

runner:
  file: .runner
  capacity: 4
  envs: {}
  env_file: ""
  timeout: 3h
  insecure: false
  fetch_timeout: 5s
  fetch_interval: 1s
  labels: []

cache:
  enabled: true
  dir: ""
  host: ""
  port: 0

container:
  network: ""
  privileged: false
  options: ""
  workdir_parent: ""
  valid_volumes:
    - /srv/ci-cache/ccache
    - /srv/ci-cache/git-repo
    - /srv/ci-cache/build-cache
  docker_host: ""
  force_pull: false

host:
  workdir_parent: ""
```

## WORKDIR /workspace

So, try to align the original request with availability of the dockerfile. If something doesn't work, but expected that it should work, ask user for the last version of the Dockerfile. Sometimes, the dockerfile can be changed.

# Rules

- Use `.gitea/workflows/code-quality.yml` as the main workflow; follow Gitea Actions and the repository's build/check commands.
- After changing everything that the user asked, you should update the bottom **Platform status** section in `docs/<project_name>/modules/ROOT/pages/installation.adoc` to match actual platforms, compilers, configurations, and checks. BUT, you shouldn't make it every step. Because CI imrovement's it's an iterative process. So, when you feel that you finished all tasks that were originally outlined - just remind the user to ask you to update the `installation.adoc`
- Validate workflow syntax and apply the `docs-generation` skill to the documentation update, including its site build.
- Gitea is hosted on the self-hosted machine. It has hardware limitations with number of cores and RAM. So, the **maximum core per one task/job is only 4 cores**.
- When something unclear for you, or you are not sure in the some server/runner/gitea/etc configuration - you must ask the user about some clarifications.
