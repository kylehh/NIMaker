# Kubernates Deployment
There are two ways of Kubernates deployment

## Helm Chart Path
- Document [Link](https://github.com/ai-dynamo/dynamo/blob/main/docs/guides/dynamo_deploy/manual_helm_deployment.md)
- High level steps
1. Build the Dynamo base image
    - CPU support only image 
    ```
    export DOCKER_SERVER=my-registry
    export IMAGE_tag=hello-world

    earthly +dynamo-base-docker --DOCKER_SERVER=$DOCKER_SERVER --IMAGE_tag=$IMAGE_tag
    # Image should succesfully be built and tagged as my-registry/dynamo-base-docker:hello-world
    ```
    - GPU support image
    ```
    # Build the base image with CUDA and vLLM support
    ./container/build.sh
    # This will create dynamo:latest-vllm image
    ```
    - See more details at `dynamo build` [doc](https://github.com/ai-dynamo/dynamo/blob/main/docs/guides/dynamo_build.md)
2. Build the application image
    ```
    # Set runtime image name
    export DYNAMO_IMAGE=<dynamo_base_image>

    # Build and containerize the Frontend service
    dynamo build --containerize hello_world:Frontend
    ```
    Try add `DOCKER_BUILDKIT=0` if the `dynamo build` fails. 
3. Extra Helm values file 
    - Previous step will generate an images "frontend:tag-values"
    - Tag the image as 'APP_IMAGE' and push to the docker repo
    - Extra Helm values from the images
    ```
    dynamo get frontend > pipeline-values.yaml
    ```
4. Helm Deployment
    - NATs and etcd deployment
    ```
    # Add and update NATS Helm repository
    helm repo add nats https://nats-io.github.io/k8s/helm/charts/
    helm repo update
    # Install NATS with custom values
    helm install --namespace ${NAMESPACE} ${RELEASE_NAME}-nats nats/nats \
        --values nats-values.yaml
    Install etcd key-value store:
    # Install etcd using Bitnami chart
    helm install --namespace ${NAMESPACE} ${RELEASE_NAME}-etcd \
        oci://registry-1.docker.io/bitnamicharts/etcd \
        --values etcd-values.yaml
    ```
    - Update `values.yaml` in the chart [folder](https://github.com/ai-dynamo/dynamo/blob/main/deploy/Kubernetes/pipeline/chart/values.yaml). Create `imagePullSecrets` if needed
    ```
    kubectl create secret docker-registry docker-imagepullsecret \
    --docker-server=<registry-server> \
    --docker-username=<username> \
    --docker-password=<password> \
    -n <namespace>
    ```
    - Application deployment
    ```
    # Set release name for Helm
    export HELM_RELEASE=hello-world-manual

    # Install/upgrade Helm release
    helm upgrade -i "$HELM_RELEASE" ./chart \
        -f pipeline-values.yaml \
        --set image="$APP_IMAGE" \
        --set dynamoIdentifier="hello_world:Frontend" \
        -n "$NAMESPACE"
    ```
5. Test the deployment
    - Port forward and run test script
    ```
    # Forward the service port to localhost
    kubectl -n ${NAMESPACE} port-forward svc/${HELM_RELEASE}-frontend 8000:80

    # Test the API endpoint
    curl -X 'POST' 'http://localhost:8000/generate' \
        -H 'accept: text/event-stream' \
        -H 'Content-Type: application/json' \
        -d '{"text": "test"}'
    ```
## Operator Path
- Document [link](https://github.com/ai-dynamo/dynamo/blob/main/docs/guides/dynamo_deploy/operator_deployment.md)
- High level steps
1. Build the Dynamo base image 
    - see detais from Helm Charm Path
2. Build `dynamo cloud` components
    - see details in the [doc](https://github.com/ai-dynamo/dynamo/blob/main/docs/guides/dynamo_deploy/dynamo_cloud.md)
    - Build `dynamo-operator` and `dynamo-api-store` and push to docker repo
    ```
    earthly --push +all-docker --DOCKER_SERVER=$DOCKER_SERVER --IMAGE_TAG=$IMAGE_TAG
    docker push $DOCKER_SERVER/dynamo-operator:$IMAGE_TAG
    docker push $DOCKER_SERVER/dynamo-api-store:$IMAGE_TAG
    ```
3. Create `dynamo cloud` environment
    - Deploy dynamo cloud
    ```
    export DOCKER_SECRET_NAME='docker-imagepullsecret'
    export DOCKER_USERNAME='$oauthtoken'
    export DOCKER_PASSWORD='xxxxxxxxxxx'
    export DOCKER_SERVER='nvcr.io/org/team' # Pull path is NOT needed for secret but needed for docker path

    cd dynamo/deploy/dynamo/helm
    ./deploy.sh
    ```
    The script will create a secret $DOCKER_SECRET_NAME and deploy the cloud
    - Expose cloud for external access
    ```
    # For local development, simply forward the 80 port and set `DYNAMO_CLOUD` 
    kubectl port-forward svc/dynamo-store <local-port>:80 -n $NAMESPACE
    export DYNAMO_CLOUD=http://localhost:<local-port>
    ```
4. Build the application image
    ```
    # Set runtime image name
    export DYNAMO_IMAGE=<dynamo_base_image>

    # Build and containerize the Frontend service
    DYNAMO_TAG=$(dynamo build hello_world:Frontend | grep "Successfully built" | awk '{ print $3 }' | sed 's/\.$//')
    ```
5. Managing the deployment
    - Create the deployment
    ```
    # Set your Helm release name
    export DEPLOYMENT_NAME=hello-world-operator

    # Create the deployment
    dynamo deployment create $DYNAMO_TAG --no-wait -n $DEPLOYMENT_NAME
    ```
    - List and delete deployment
    ```
    dynamo deployment list
    dynamo deployment get $DEPLOYMENT
    dynamo deployment delete $DEPLOYMENT_NAME
    ```
6. Test the deployment
    - Port forward and run test script
    ```
    kubectl -n ${NAMESPACE} port-forward svc/${DEPLOYMENT_NAME}-frontend 8000:8000
    # Test the API endpoint
    curl -X 'POST' 'http://localhost:8000/generate' \
        -H 'accept: text/event-stream' \
        -H 'Content-Type: application/json' \
        -d '{"text": "test"}'
    ```