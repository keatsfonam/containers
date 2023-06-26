
FROM ghcr.io/onedr0p/alpine:3.18.2 as builder
ARG TARGETPLATFORM
ARG VERSION
ARG CHANNEL

ENV \
    NZBGET__BRANCH="${CHANNEL}" \
    NZBGET__PORT="6789" \
    NZBGET__APP_DIR="/app/nzbget" \
    NZBGET__MAIN_DIR="/downloads"

RUN	echo "**** install build packages ****" \
    && case "${TARGETPLATFORM}" in \
        'linux/amd64') export ARCH='x64' ;; \
        'linux/arm64') export ARCH='arm64' ;; \
    esac \
	&& apk add --no-cache \
	g++ \
	gcc \
	git \
	libxml2-dev \
	libxslt-dev \
	libcap-dev \
	autoconf \
	automake \
	make \
	ncurses-dev \
	openssl-dev \
  curl \
	&& echo "**** build nzbget ****" \
	&& mkdir -p ${NZBGET__APP_DIR} \
	&& git clone --branch ${NZBGET__BRANCH} https://github.com/nzbget-ng/nzbget.git /nzbget \
	&& cd /nzbget \
	&& make -j \
	&& make prefix=${NZBGET__APP_DIR} install \
	&& sed -i \
    -e "s#^MainDir=.*#MainDir=${NZBGET__MAIN_DIR}#g" \
    -e "s#^ScriptDir=.*#ScriptDir=$\{MainDir\}/scripts#g" \
    -e "s#^WebDir=.*#WebDir=${NZBGET__APP_DIR}/webui#g" \
    -e "s#^ConfigTemplate=.*#ConfigTemplate=${NZBGET__APP_DIR}/webui/nzbget.conf.template#g" \
    -e "s#^UnrarCmd=.*#UnrarCmd=${NZBGET__APP_DIR}/unrar#g" \
    -e "s#^SevenZipCmd=.*#SevenZipCmd=${NZBGET__APP_DIR}/7za#g" \
    -e "s#^CertStore=.*#CertStore=${NZBGET__APP_DIR}/cacert.pem#g" \
    -e "s#^CertCheck=.*#CertCheck=yes#g" \
    -e "s#^DestDir=.*#DestDir=$\{MainDir\}/completed#g" \
    -e "s#^InterDir=.*#InterDir=$\{MainDir\}/intermediate#g" \
    -e "s#^LogFile=.*#LogFile=/dev/stdout#g" \
    -e "s#^AuthorizedIP=.*#AuthorizedIP=127.0.0.1,10.*,192.168.*#g" \
    -e "s#^ControlPort=.*#ControlPort=${NZBGET__PORT}#g" \
${NZBGET__APP_DIR}/share/nzbget/nzbget.conf \
	&& mv ${NZBGET__APP_DIR}/share/nzbget/webui ${NZBGET__APP_DIR}/ \
	&& cp ${NZBGET__APP_DIR}/share/nzbget/nzbget.conf ${NZBGET__APP_DIR}/webui/nzbget.conf.template \
	&& ln -s /usr/bin/7za ${NZBGET__APP_DIR}/7za \
	&& ln -s /usr/bin/unrar ${NZBGET__APP_DIR}/unrar \
	&& cp /nzbget/pubkey.pem ${NZBGET__APP_DIR}/pubkey.pem \
	&& curl -o ${NZBGET__APP_DIR}/cacert.pem -L "https://curl.se/ca/cacert.pem" \
	&& chown -R kah:kah ${NZBGET__APP_DIR} \
	&& chmod -R 755 ${NZBGET__APP_DIR}

FROM ghcr.io/onedr0p/alpine:3.18.2
ARG	BUILD_DATE
ARG	VERSION

ENV \
    NZBGET__APP_DIR="/app/nzbget" \
    NZBGET__MAIN_DIR="/downloads"

LABEL build_version="Linuxserver.io version:- ${VERSION} Build-date:- ${BUILD_DATE}"
LABEL maintainer="thelamer"
RUN	echo "**** install packages ****" \
	&& mkdir -p ${NZBGET__MAIN_DIR} \
  && chown kah:kah ${NZBGET__MAIN_DIR} \
  && chmod 755 ${NZBGET__MAIN_DIR} \
	&& apk add --no-cache \
	libxml2 \
	libxslt \
	openssl \
	p7zip \
	py3-pip \
	python3

USER kah
COPY --from=builder ${NZBGET__APP_DIR} ${NZBGET__APP_DIR}
COPY ./apps/nzbget-ng/entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh"]

EXPOSE 6789
