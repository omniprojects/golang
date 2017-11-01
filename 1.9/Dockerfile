FROM buildpack-deps:xenial-scm

RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash

RUN apt-get update
RUN apt-get install -y --no-install-recommends \
		git-lfs \
		postgresql-client \
		g++ \
		gcc \
		libc6-dev \
		zip \
		make \
		awscli
RUN rm -rf /var/lib/apt/lists/*

ENV DOCKERIZE_VERSION v0.5.0
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

ENV GOLANG_VERSION 1.9.2
ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz

RUN curl -fsSL "$GOLANG_DOWNLOAD_URL" -o golang.tar.gz \
	&& echo "$GOLANG_DOWNLOAD_SHA256  golang.tar.gz" \
	&& tar -C /usr/local -xzf golang.tar.gz \
	&& rm golang.tar.gz

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH

COPY go-wrapper /usr/local/bin/
