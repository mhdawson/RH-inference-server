# Set inference server using OpenShift

Based on the instuctions [here](https://docs.redhat.com/en/documentation/red_hat_ai_inference_server/3.0/html-single/getting_started/index).

### 1. Log in to your cluster oc login ...

### 2. create secrets
a. oc create secret generic hf-token-secret \
     --from-literal=HUGGING_FACE_HUB_TOKEN=<your_HF_token>
b. oc create secret docker-registry redhat-registry \
    --docker-server=registry.redhat.io \
    --docker-username=USERNAME \
    --docker-password=PASSWORD \
    --docker-email=USERNAME@redhat.com


### 3. Create PVC:
    oc apply -f deployment/storage.yaml
### 4. Deploy the inference server:
    oc apply -f deployment/inferece-server-deployment.yaml
### 5. Port forward:
    oc port-forward pod/rhai-inference-server-... 8000:8000
### 6. Testing:
    git clone https://github.com/vllm-project/vllm.git
    cd vllm
    python3 -m venv .venv
    source .venv/bin/activate
    pip install vllm pandas datasets
    python benchmarks/benchmark_serving.py --backend vllm \
    --model RedHatAI/Llama-3.2-1B-Instruct-FP8 --num-prompts 100 \
    --dataset-name random  --random-input 1024 \
    --random-output 512 --port 8000

    And the output should look like this:
    ============ Serving Benchmark Result ============
    Successful requests:                     100       
    Benchmark duration (s):                  9.75      
    Total input tokens:                      102300    
    Total generated tokens:                  40416     
    Request throughput (req/s):              10.26     
    Output token throughput (tok/s):         4146.89   
    Total Token throughput (tok/s):          14643.39  
    ---------------Time to First Token----------------
    Mean TTFT (ms):                          958.81    
    Median TTFT (ms):                        982.10    
    P99 TTFT (ms):                           1179.69   
    -----Time per Output Token (excl. 1st token)------
    Mean TPOT (ms):                          18.19     
    Median TPOT (ms):                        16.99     
    P99 TPOT (ms):                           31.95     
    ---------------Inter-token Latency----------------
    Mean ITL (ms):                           16.91     
    Median ITL (ms):                         16.85     
    P99 ITL (ms):                            41.46     
    ==================================================

