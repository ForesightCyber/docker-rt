ARG IMAGE=netsandbox/request-tracker-base
ARG TAG=latest
FROM ${IMAGE}:${TAG}

EXPOSE 80

LABEL maintainer="Christian Loos <cloos@netsandbox.de>"
LABEL org.opencontainers.image.source="https://github.com/netsandbox/docker-rt"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /usr/local/src
RUN curl -sSL "https://download.bestpractical.com/pub/rt/release/rt-5.0.1.tar.gz" -o rt.tar.gz \
  && echo "6c181cc592c48a2cba8b8df1d45fda0938d70f84ceeba1afc436f16a6090f556  rt.tar.gz" | sha256sum -c \
  && tar -xzf rt.tar.gz

WORKDIR /usr/local/src/rt-5.0.1
RUN perl -MCPAN -e shell </dev/null
RUN sed -i "s/buster /testing /g" /etc/apt/sources.list
RUN apt update
RUN apt install -y libdbd-pgsql libdbd-pg-perl libdbix-searchbuilder-perl libpath-dispatcher-perl 
RUN ./configure \
    --disable-gpg \
    --disable-smime \
    --enable-developer \
    --enable-externalauth \
    --enable-gd \
    --enable-graphviz \
    --with-db-type=Pg \
  && make fixdeps \
  && make install \
  && rm -rf /usr/local/src/* \
  && apt clean

COPY apache.rt.conf /etc/apache2/sites-available/000-default.conf
COPY RT_SiteConfig.pm /opt/rt5/etc/RT_SiteConfig.pm
COPY docker-entrypoint.sh /usr/local/bin/

RUN chown -R www-data:www-data /opt/rt5/var/
RUN chown -R www-data:www-data /opt/rt5/etc/

VOLUME /opt/rt5

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["apache2-foreground"]
