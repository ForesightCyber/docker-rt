ARG IMAGE=netsandbox/request-tracker-base
ARG TAG=latest
FROM ${IMAGE}:${TAG}

LABEL maintainer="Christian Loos <cloos@netsandbox.de>"
LABEL org.opencontainers.image.source="https://github.com/netsandbox/docker-rt"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /usr/local/src
# hadolint ignore=DL3003,SC2174
RUN apt update \
  && apt -y install libpq-dev
  
RUN curl -fsSL "https://download.bestpractical.com/pub/rt/release/rt-5.0.3.tar.gz" -o rt.tar.gz \
  && echo "e23aee3cb291ccad5e521aeabe0fcd2f076bcfa8b7f801af498a7505e53d8441 rt.tar.gz" | sha256sum -c \
  && tar -xzf rt.tar.gz \
  && cd rt-5.0.3 \
  && ./configure \
    --enable-developer \
    --enable-externalauth \
    --enable-gd \
    --enable-gpg \
    --enable-graphviz \
    --enable-smime \
    --with-db-type=Pg \
    --with-web-handler=standalone \
  && PERL_MM_USE_DEFAULT=1 perl -MCPAN -e shell \
  && make fixdeps \
  && make install \
  && mkdir --mode=0600 --parents /opt/rt5/var/data/{gpg,smime} \
  && cd .. && rm -rf /usr/local/src/* \
  && apt clean

WORKDIR /opt/rt5
COPY RT_SiteConfig.pm etc/

RUN chown -R www-data:www-data /opt/rt5/var/
RUN chown -R www-data:www-data /opt/rt5/etc/

VOLUME /opt/rt5

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 80

CMD ["/opt/rt5/sbin/rt-server", "--port", "80", "--webpath", "/rt"]
