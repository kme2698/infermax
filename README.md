# InferMax

## How to install?

#### vLLM installation

We use vLLM as a real inference system and modified it to take the schedules generated outside vLLM.
We recommend running InferMax with Docker (see [here](https://docs.vllm.ai/en/v0.6.3/getting_started/installation.html))

```
cd vllm-scheduler
pip install -e .
```

#### InferMax installation

We extend a simulation-based framework, Vidur, to implement our schedulers and embed our cost models.

```
git clone ...
cd vidur
pip install -r requirements.txt
```

## How to run?

First, run Vidur to simulate the scheduler for a given workload.
Read the scripts, `vidur/run_splitwise*.sh` to see how to run Vidur for different workloads and schedulers.

```
export MODEL_SIZE=7B # If you want to run the Llama 3 7B model, set this to 70B
cd vidur
bash run_exp_splitwise_test_sortI_histogram.sh # run our new scheduler
```

As a result, you will get a directory that contains the schedule and the results of the simulation under ./simulator_output
You need to turn on the option that generates the schedule and the results of the simulation (see ./run_exp_splitwise_test.sh)

After that, run vLLM with the schedule generated by Vidur.
If you want to enable the chunked prefill, add `--enable-chunked-prefill` to the command.

```
cd vllm-scheduler/benchmarks/scheduler
python3 run_vllm.py --vidur \
    --tensor-parallel-size 4 \
    --gpu-memory-utilization 0.85 \
    --model meta-llama/Meta-Llama-3-70B \
    --max-num-batched-tokens 16384 \
    --max-num-seqs 1024 \
    --disable-async-output-proc \
    --schedule ${path/to/schedule.pkl}
```

As a result, you will get a file ` path/to/schedule.pkl.vllm.json` that contains the execution results (e.g., batch time, model forward time per batch, etc.)
