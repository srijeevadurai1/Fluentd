FROM debian:stretch
LABEL maintainer "Aricent <srithar.durairaj@aricent.com>"
LABEL Description="Fluentd docker image" Vendor="Fluent Organization" Version="1.3.3"
ENV TINI_VERSION=0.18.0

# Do not split this into multiple RUN!
# Docker creates a layer for every RUN-Statement
# therefore an 'apt-get purge' has no effect
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
            ca-certificates \
            ruby \
                        nginx \
 && buildDeps=" \
      make gcc g++ libc-dev \
      ruby-dev \
      wget bzip2 gnupg dirmngr \
    " \
 && apt-get install -y --no-install-recommends $buildDeps \
 && echo 'gem: --no-document' >> /etc/gemrc \
 && gem install oj -v 3.3.10 \
 && gem install json -v 2.1.0 \
 && gem install fluentd -v 1.3.3 \
 && apt-get install -y gnupg2 \
 && apt-get install -y postgresql-client && ln -s /usr/lib/libpq.so.* /usr/lib/libpq.so \
 && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
 && wget -O /usr/local/bin/tini "https://github.com/krallin/tini/releases/download/v$TINI_VERSION/tini-$dpkgArch" \
 && wget -O /usr/local/bin/tini.asc "https://github.com/krallin/tini/releases/download/v$TINI_VERSION/tini-$dpkgArch.asc" 
RUN export GNUPGHOME="$(mktemp -d)" \
 && gpg2 --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 6380DC428747F6C393FEACA59A84159D7001A4E5 \
 && gpg2 --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini \
 && apt-get install -y libpq-dev \ 
 && rm -r /usr/local/bin/tini.asc \
 && chmod +x /usr/local/bin/tini \
 && tini -h \
 && wget -O /tmp/jemalloc-4.5.0.tar.bz2 https://github.com/jemalloc/jemalloc/releases/download/4.5.0/jemalloc-4.5.0.tar.bz2 \
 && cd /tmp && tar -xjf jemalloc-4.5.0.tar.bz2 && cd jemalloc-4.5.0/ \
 && ./configure && make \
 && mv lib/libjemalloc.so.2 /usr/lib \
 && gem install fluent-plugin-cat-sweep \
 && gem install fluent-plugin-postgres-bulk \
 && buildDeps=" \
      make gcc g++ libc-dev \
      ruby-dev \
      wget bzip2 gnupg dirmngr \
    " \
 && apt-get purge -y --auto-remove \
                  -o APT::AutoRemove::RecommendsImportant=false \
                  $buildDeps \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

# for log storage (maybe shared with host)
RUN mkdir -p /fluentd/log
# configuration/plugins path (default: copied from .)
RUN mkdir -p /fluentd/etc /fluentd/plugins

RUN groupadd -r fluent && useradd -r -g fluent fluent
RUN chown -R fluent /fluentd && chgrp -R fluent /fluentd
#RUN /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-cat-sweep
#RUN /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-postgres
#RUN /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-postgres-bulk
RUN mkdir -p /home/fluent/ && chown fluent:fluent /home/fluent/

#COPY fluent.conf /fluentd/etc/
COPY entrypoint.sh /bin/
COPY docker_mgmt.key /fluentd/etc/
COPY docker_mgmt.cer /fluentd/etc/
COPY key-password.txt /fluentd/etc/
COPY nginx_fluentd.conf /etc/nginx/conf.d/


ENV LD_PRELOAD="/usr/lib/libjemalloc.so.2"
EXPOSE 30555 9292 5140

USER fluent
ENTRYPOINT ["tini",  "--", "/bin/entrypoint.sh"]
CMD ["fluentd"]

