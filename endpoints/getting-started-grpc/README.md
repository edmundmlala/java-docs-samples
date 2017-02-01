# Endpoints Getting Started with gRPC & Java Quickstart

It is assumed that you have a working gRPC and Java environment.

1. Build the code:

    ```
    ./gradlew build
    ```

1. Test running the code, optional:

    ```
    # In the background or another terminal run the server:
    java -jar server/build/libs/server.jar

    # Check the client parameters:
    java -jar client/build/libs/client.jar --help

    # Run the client
    java -jar client/build/libs/client.jar --greetee 'Endpoints!'
    ```

1. Generate the `out.pb` from the proto file.

    ```
    protoc --include_imports --include_source_info api/src/main/proto/helloworld.proto --descriptor_set_out out.pb
    ```

1. Edit, `api_config.yaml`. Replace `MY_PROJECT_ID` with your project id.

1. Deploy your service config to Service Management:

    ```
    gcloud service-management deploy out.pb api_config.yaml
    # The Config ID should be printed out, looks like: 2017-02-01r0, remember this

    # set your project to make commands easier
    GCLOUD_PROJECT=<your project id>

    # Print out your Config ID again, in case you missed it
    gcloud service-management configs list --service hellogrpc.endpoints.${GCLOUD_PROJECT}.cloud.goog
    ```

1. Also get an API key from the Console's API Manager for use in the client later. (https://console.cloud.google.com/apis/credentials)

1. Build a docker image for your gRPC server, store in your Registry

    ```
    gcloud container builds submit --tag gcr.io/${GCLOUD_PROJECT}/java-grpc-hello .
    ```

1. Either deploy to GCE (below) or GKE (further down)

### GCE

1. Create your instance and ssh in.

    ```
    gcloud compute instances create grpc-host --image-family gci-stable --image-project google-containers --tags=http-server
    gcloud compute ssh grpc-host
    ```

1. Set some variables to make commands easier

    ```
    GCLOUD_PROJECT=$(curl -s "http://metadata.google.internal/computeMetadata/v1/project/project-id" -H "Metadata-Flavor: Google")
    SERVICE_NAME=hellogrpc.endpoints.${GCLOUD_PROJECT}.cloud.goog
    SERVICE_CONFIG_ID=<your config id>
    ```

1. Pull your credentials to access Container Registry, and run your gRPC server container

    ```
    /usr/share/google/dockercfg_update.sh
    docker run -d --name=grpc-hello gcr.io/${GCLOUD_PROJECT}/java-grpc-hello
    ```

1. Run the Endpoints proxy

    ```
    docker run --detach --name=esp \
        -p 80:9000 \
        --link=grpc-hello:grpc-hello \
        b.gcr.io/endpoints/endpoints-runtime:1 \
        -s ${SERVICE_NAME} \
        -v ${SERVICE_CONFIG_ID} \
        -P 9000 \
        -a grpc://grpc-hello:50051
    ```

1. Back on your local machine, get the IP of your GCE instance.

    ```
    gcloud compute instances list
    ```

1. Run the client

    ```
    java -jar client/build/libs/client.jar --host <ip of GCE instance>:80 --api_key <api key from console>
    ```

### GKE

1. Create a cluster
1. Edit GKE yaml, fill in service name / config
1. Deploy to GKE
1. Get IP of load balancer
1. Run the client

    ```
    java -jar client/build/libs/client.jar --host <ip of GKE LB>:80 --api_key <api key from console>
    ```
