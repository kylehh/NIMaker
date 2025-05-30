# Kubernates Deployment

## Operator Deployment
1. Following steps in the hello world example to deploy `dynamo cloud`
2. Build the dynamo image
    ```
    ./container/buid.sh
    # Set runtime image name, make sure it's the GPU supported image
    docker push <dynamo_base_image>
    export DYNAMO_IMAGE=<dynamo_base_image>
    ```
3. Build the application image
    ```
    # Build and containerize the Frontend service
    DYNAMO_TAG=$(dynamo build graphs.agg:Frontend | grep "Successfully built" |  awk '{ print $NF }' | sed 's/\.$//')
    ```
4. Managing the deployment
    - Create the deployment
    ```
    # Set your Helm release name
    export DEPLOYMENT_NAME=llm-agg

    # Create the deployment
    dynamo deployment create $DYNAMO_TAG --no-wait -n $DEPLOYMENT_NAME -f ./configs/agg.yaml
    ```
    It will create a container under `$DOCKER_SERVER/dynamo-pipelines:dynamo.front.xxxx`
    in `dynamo-image-builder` pod. 
    And this container will be used to create frontend/processor/vllmworker pods.

6. Test the deployment
    - Port forward and run test script
    ```
    export FRONTEND_POD=$(kubectl get pods -n ${NAMESPACE} | grep "${DEPLOYMENT_NAME}-frontend" | sort -k1 | tail -n1 | awk '{print $1}')

    # Forward the service port to localhost
    kubectl -n ${NAMESPACE} port-forward pod/${FRONTEND_POD} 8000:8000

    # Test the API endpoint
    curl localhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "deepseek-ai/DeepSeek-R1-Distill-Llama-8B",
        "messages": [
        {
            "role": "user",
            "content": "In the heart of Eldoria, an ancient land of boundless magic and mysterious creatures, lies the long-forgotten city of Aeloria. Once a beacon of knowledge and power, Aeloria was buried beneath the shifting sands of time, lost to the world for centuries. You are an intrepid explorer, known for your unparalleled curiosity and courage, who has stumbled upon an ancient map hinting at ests that Aeloria holds a secret so profound that it has the potential to reshape the very fabric of reality. Your journey will take you through treacherous deserts, enchanted forests, and across perilous mountain ranges. Your Task: Character Background: Develop a detailed background for your character. Describe their motivations for seeking out Aeloria, their skills and weaknesses, and any personal connections to the ancient city or its legends. Are they driven by a quest for knowledge, a search for lost familt clue is hidden."
        }
        ],
        "stream":false,
        "max_tokens": 30
    }'
    ```