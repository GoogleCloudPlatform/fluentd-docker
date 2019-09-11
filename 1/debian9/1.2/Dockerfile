# Copyright 2017 The Kubernetes Authors.
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# This Dockerfile will build an image that is configured
# to run Fluentd with an Elasticsearch plug-in and the
# provided configuration file.
# The image acts as an executable for the binary /usr/sbin/td-agent.
# Note that fluentd is run with root permssion to allow access to
# log files with root only access under /var/log/containers/*

FROM gcr.io/google-appengine/debian9

ARG DEBIAN_FRONTEND=noninteractive

# This field is hardcoded to correspond with value in Gemfile,
# which is not generated from versions.yaml
ENV C2D_RELEASE=1.2.4

COPY Gemfile /Gemfile

RUN BUILD_DEPS="make gcc g++ libc6-dev ruby-dev libffi-dev" && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
      $BUILD_DEPS ca-certificates libjemalloc1 ruby && \
    echo 'gem: --no-document' >> /etc/gemrc && \
    gem install --file Gemfile && \
    apt-get purge -y --auto-remove \
      -o APT::AutoRemove::RecommendsImportant=false \
      $BUILD_DEPS && \
    rm -rf /var/lib/apt/lists/* && \
    # Ensure fluent has enough file descriptors
    ulimit -n 65536

COPY fluent.conf /etc/fluent/fluent.conf
COPY docker-entrypoint.sh /docker-entrypoint.sh

EXPOSE 80

ENV LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.1

RUN test ${C2D_RELEASE} = $(fluentd -v --dry-run | grep -o -E 'starting fluentd-([0-9]+\.)([0-9]+\.)([0-9]+)' | cut -d- -f2) \
         || { echo "Version check failed !! WARNING: version is hardcoded !!"; exit 1; }

CMD /docker-entrypoint.sh $FLUENTD_ARGS
