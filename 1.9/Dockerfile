FROM buildpack-deps:xenial-scm

RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash
RUN curl -Lo /usr/local/bin/jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64 \
	&& chmod +x /usr/local/bin/jq

RUN apt-get update
RUN apt-get install -y --no-install-recommends \
		git-lfs \
		postgresql-client \
		g++ \
		gcc \
		libc6-dev \
		zip \
		make \
		awscli \
		unzip
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

COPY go-wrapper /usr/local/bin/
