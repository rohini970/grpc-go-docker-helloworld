# Use the official Golang image as base
FROM golang:latest

# Set the Current Working Directory inside the container

WORKDIR /root/hmp-api-embedded-go/server

# Copy go mod and sum files
COPY go.mod go.sum ./

# Download all dependencies. Dependencies will be cached if the go.mod and go.sum files are not changed
RUN go mod download

# Copy the rest of the application source code
COPY . .

# Install protoc and zip system library
RUN apt-get update && apt-get install -y zip protobuf-compiler && \
    mkdir /opt/protoc && cd /opt/protoc && wget https://github.com/protocolbuffers/protobuf/releases/download/v3.7.0/protoc-3.7.0-linux-x86_64.zip && \
    unzip protoc-3.7.0-linux-x86_64.zip

ENV PATH=$PATH:$GOPATH/bin:/opt/protoc/bin

# Compile and install protoc-gen-go plugin
RUN go get -d github.com/golang/protobuf/protoc-gen-go && \
    go install github.com/golang/protobuf/protoc-gen-go
# Install protoc-gen-go plugin
RUN go get -u github.com/golang/protobuf/protoc-gen-go
# Generate Go code from proto file
RUN protoc -I . homegateway.proto --go_out=plugins=grpc:.


# Copy the grpc proto file and generate the go module
RUN mkdir -p /go/src/hmp-api-embedded-go
COPY homegateway.proto /go/src/hmp-api-embedded-go
RUN cd /go/src && \
    protoc -I hmp-api-embedded-go/ hmp-api-embedded-go/homegateway.proto --go_out=plugins=grpc:hmp-api-embedded-go

# Copy the project to be executed
RUN mkdir -p /go/src/grpc-client
COPY server.go /go/src/grpc-client

ENTRYPOINT cd /go/src/grpc-client && go run server.go

# Expose port 50051 to the outside world
EXPOSE 50051

# Command to run the executable
CMD ["./server"]
