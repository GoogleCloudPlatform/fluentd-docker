# Copyright 2017 The Kubernetes Authors.
# Copyright 2020 Google LLC
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

ENV C2D_RELEASE=1.2.4
ENV PATH="/opt/gtk/bin:${PATH}"

COPY Gemfile /Gemfile

RUN mkdir ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      dirmngr \
      libjemalloc1 \
      gnupg2 \
      procps

RUN gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB \
    && curl -sSL https://get.rvm.io | bash -s stable

RUN /bin/bash -l -c "rvm install 2.6.5" \
    && ln -s /usr/local/rvm/rubies/ruby-2.6.5/bin/ruby /usr/local/bin/ruby \
    && ln -s /usr/local/rvm/rubies/ruby-2.6.5/bin/gem  /usr/local/bin/gem

RUN /bin/bash -l -c "gem install --file Gemfile" \
    && ulimit -n 65536

COPY fluent.conf /etc/fluent/fluent.conf
COPY docker-entrypoint.sh /docker-entrypoint.sh

EXPOSE 80

ENV LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.1

CMD /docker-entrypoint.sh $FLUENTD_ARGS
