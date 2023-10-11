ARG IMAGE=netsandbox/request-tracker-base
ARG TAG=latest
FROM ${IMAGE}:${TAG}

LABEL maintainer="Christian Loos <cloos@netsandbox.de>"
LABEL org.opencontainers.image.source="https://github.com/netsandbox/docker-rt"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV RT_SMTP_HOST smtp
ENV RT_SMTP_PORT 25

WORKDIR /usr/local/src
# hadolint ignore=DL3003,SC2174
RUN apt update \
  && apt -y install libpq-dev msmtp patch
  
COPY files/EditCustomFieldSelect /usr/local/src
  
RUN curl -fsSL "https://download.bestpractical.com/pub/rt/release/rt-5.0.4.tar.gz" -o rt.tar.gz \
  && echo "916d870d22d92027f843798be6f880aaf1517aebc3f6ab25f456f4e772f4834d rt.tar.gz" | sha256sum -c \
  && tar -xzf rt.tar.gz \
  && cd rt-5.0.4 \
  && ./configure \
    --enable-developer \
    --enable-externalauth \
    --enable-gd \
    --enable-gpg \
    --enable-graphviz \
    --enable-smime \
    --with-db-type=Pg \
    --with-web-handler=standalone
    
RUN PERL_MM_USE_DEFAULT=1 perl -MCPAN -e shell \
  && cd rt-5.0.4 \
  && make fixdeps \
  && make install

RUN cd rt-5.0.4 \
  && curl -o RT-Extension-ConditionalCustomFields-1.09.tar.gz https://cpan.metacpan.org/authors/id/G/GI/GIBUS/RT-Extension-ConditionalCustomFields-1.09.tar.gz \
  && tar -zxvf RT-Extension-ConditionalCustomFields-1.09.tar.gz \
  && cd RT-Extension-ConditionalCustomFields-1.09 \
  && perl Makefile.PL \
  && make && make install && cd .. \
  && patch -p1 < RT-Extension-ConditionalCustomFields-1.09/patches/5.0.4-add-callbacks-to-extend-customfields-capabilities.patch \
  && cp /usr/local/src/EditCustomFieldSelect /opt/rt5/share/html/Elements/EditCustomFieldSelect

RUN mkdir --mode=0600 --parents /opt/rt5/var/data/{gpg,smime} \
  && cd rt-5.0.4 \
  && make install \
  && cd .. && rm -rf /usr/local/src/* \
  && apt clean

WORKDIR /opt/rt5
COPY RT_SiteConfig.pm etc/
COPY msmtp-wrapper.sh usr/bin/

RUN chown -R www-data:www-data /opt/rt5/var/
RUN chown -R www-data:www-data /opt/rt5/etc/

VOLUME /opt/rt5

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 80

CMD ["/opt/rt5/sbin/rt-server", "--port", "80", "--webpath", "/rt"]
