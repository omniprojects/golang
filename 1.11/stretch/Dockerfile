FROM buildpack-deps:stretch-scm

RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash
RUN curl -Lo /usr/local/bin/jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64 \
	&& chmod +x /usr/local/bin/jq

# gcc for cgo
RUN apt-get update && apt-get install -y --no-install-recommends \
		g++ \
		gcc \
		libc6-dev \
		make \
		pkg-config \
		git-lfs \
        postgresql-client \
        zip \
        awscli \
        unzip \
	&& rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION 1.11

RUN set -eux; \
	\
# this "case" statement is generated via "update.sh"
	dpkgArch="$(dpkg --print-architecture)"; \
	case "${dpkgArch##*-}" in \
		amd64) goRelArch='linux-amd64'; goRelSha256='b3fcf280ff86558e0559e185b601c9eade0fd24c900b4c63cd14d1d38613e499' ;; \
		armhf) goRelArch='linux-armv6l'; goRelSha256='8ffeb3577d8ca5477064f1cb8739835973c866487f2bf81df1227eaa96826acd' ;; \
		arm64) goRelArch='linux-arm64'; goRelSha256='e4853168f41d0bea65e4d38f992a2d44b58552605f623640c5ead89d515c56c9' ;; \
		i386) goRelArch='linux-386'; goRelSha256='1a91932b65b4af2f84ef2dce10d790e6a0d3d22c9ea1bdf3d8c4d9279dfa680e' ;; \
		ppc64el) goRelArch='linux-ppc64le'; goRelSha256='e874d617f0e322f8c2dda8c23ea3a2ea21d5dfe7177abb1f8b6a0ac7cd653272' ;; \
		s390x) goRelArch='linux-s390x'; goRelSha256='c113495fbb175d6beb1b881750de1dd034c7ae8657c30b3de8808032c9af0a15' ;; \
		*) goRelArch='src'; goRelSha256='afc1e12f5fe49a471e3aae7d906c73e9d5b1fdd36d52d72652dde8f6250152fb'; \
			echo >&2; echo >&2 "warning: current architecture ($dpkgArch) does not have a corresponding Go binary release; will be building from source"; echo >&2 ;; \
	esac; \
	\
	url="https://golang.org/dl/go${GOLANG_VERSION}.${goRelArch}.tar.gz"; \
	wget -O go.tgz "$url"; \
	echo "${goRelSha256} *go.tgz" | sha256sum -c -; \
	tar -C /usr/local -xzf go.tgz; \
	rm go.tgz; \
	\
	if [ "$goRelArch" = 'src' ]; then \
		echo >&2; \
		echo >&2 'error: UNIMPLEMENTED'; \
		echo >&2 'TODO install golang-any from jessie-backports for GOROOT_BOOTSTRAP (and uninstall after build)'; \
		echo >&2; \
		exit 1; \
	fi; \
	\
	export PATH="/usr/local/go/bin:$PATH"; \
	go version

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"


ENV JUNIT_REPORT_LOCATION github.com/jstemmer/go-junit-report
RUN go get -u $JUNIT_REPORT_LOCATION && \
		cd "${GOPATH}/src/$JUNIT_REPORT_LOCATION" && \
		go build && \
		mv go-junit-report /usr/local/bin

#Install Protocol Buffer Compiler
ENV PROTOC_VERSION 3.5.1
RUN curl -OL https://github.com/google/protobuf/releases/download/v$PROTOC_VERSION/protoc-$PROTOC_VERSION-linux-x86_64.zip && \
    unzip protoc-$PROTOC_VERSION-linux-x86_64.zip -d protoc && \
    mv protoc/bin/* /usr/local/bin/ && \
    mv protoc/include/* /usr/local/include/ && \
    rm -rf ./protoc && \
    rm protoc-$PROTOC_VERSION-linux-x86_64.zip

#Install protobuf->Golang generator
ENV GEN_GO_LOCATION github.com/golang/protobuf/protoc-gen-go
RUN go get -u $GEN_GO_LOCATION && \
    cd "${GOPATH}/src/$GEN_GO_LOCATION" && \
	go build && \
	mv protoc-gen-go /usr/local/bin

#Install protobuf->Twirp  generator
ENV GEN_TWIRP_LOCATION github.com/twitchtv/twirp/protoc-gen-twirp
RUN go get -u $GEN_TWIRP_LOCATION && \
    cd "${GOPATH}/src/$GEN_TWIRP_LOCATION" && \
	go build && \
	mv protoc-gen-twirp /usr/local/bin

#Install protobuf->Swagger  generator
ENV GEN_SWAGGER_LOCATION github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
RUN go get -u $GEN_SWAGGER_LOCATION && \
    cd "${GOPATH}/src/$GEN_SWAGGER_LOCATION" && \
	go build && \
	mv protoc-gen-swagger /usr/local/bin

#Install protobuf->Validator generator
ENV GEN_VALIDATORS_LOCATION github.com/mwitkow/go-proto-validators/protoc-gen-govalidators
RUN go get -u $GEN_VALIDATORS_LOCATION && \
    cd "${GOPATH}/src/$GEN_VALIDATORS_LOCATION" && \
	go build && \
	mv protoc-gen-govalidators /usr/local/bin

WORKDIR $GOPATH
