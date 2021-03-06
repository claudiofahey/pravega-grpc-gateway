# Pravega GRPC Gateway

This is a sample GRPC server that provides a gateway to Pravega.
It provides **limited** Pravega functionality to any environment that support GRPC, including Python.

Using a GRPC gateway is better than a REST gateway for the following reasons:

- GRPC streaming is used for reading and writing events. This allows the Pravega connection to remain open for the life
  of the streaming request. Within a streaming request, any number of read or write operations can be performed.
  In the case of writing, these can also be wrapped in transactions. 
  Events can be marked to commit the current transaction and open a new one.
  
- GRPC uses Protobuf for efficient serialization.
  REST/JSON requires base64 encoding of binary data which is less efficient.
  
- GRPC stubs (client code) can be easily created for nearly any language and environment.

# Run Gateway Locally

```
export PRAVEGA_CONTROLLER=tcp://localhost:9090
../gradlew run
```

# Run Gateway in Docker

```
export DOCKER_REPOSITORY=claudiofahey
export IMAGE_TAG=0.6.0
export PRAVEGA_CONTROLLER=tcp://localhost:9090
scripts/build-k8s-components.sh
docker run -d \
  --restart always \
  -e PRAVEGA_CONTROLLER \
  -p 54672:80 \
  --name pravega-grpc-gateway \
  ${DOCKER_REPOSITORY}/pravega-grpc-gateway:${IMAGE_TAG}
```

# Run Gateway in Dell EMC Streaming Data Platform (SDP)

If your network requires non-standard TLS certificates to be trusted during the build process, 
place them in the ca-certificates directory.

```
export DOCKER_REPOSITORY=<hostname>:<port>/<namespace>
export IMAGE_TAG=0.6.0
scripts/build-k8s-components.sh
scripts/deploy-k8s-components.sh
```

# Using the Pravega GRPC Gateway from Python

```
git clone https://github.com/pravega/pravega-grpc-gateway /tmp/pravega-grpc-gateway && \
cd /tmp/pravega-grpc-gateway && \
pip install grpcio pravega-grpc-gateway/src/main/python
```

See [integration_test1.py](pravega-grpc-gateway/src/test/python/integration_test1.py).

# Create Python Environment using Conda

1. Install [Miniconda Python 3.7](https://docs.conda.io/en/latest/miniconda.html).

2. Create Conda environment.
    ```
    cd pravega-grpc-gateway
    ./create_conda_env.sh
    ```

# Run Test and Sample Applications

1. Create Python environment as shown above.

2. Run the following commands:
    ```
    cd pravega-grpc-gateway
    conda activate ./env
    pip install -e src/main/python
    src/test/python/integration_test1.py
    ```

# Rebuild Python GRPC Stub for Pravega Gateway

This section is only needed if you make changes to the pravega.proto file.

This will build the Python files necessary to allow a Python application to call this gateway.

1. Create Python environment as shown above.

2. Run Protobuf compiler.
    ```
    ./build_python.sh
    ```
