# Deploying LLMs with vLLM

vLLM is a high-performance system designed to accelerate the serving of large language models (LLMs), making them more efficient and scalable for real-world applications. Developed by researchers at UC Berkeley, vLLM aims to overcome the limitations that existing inference systems face, particularly when serving modern LLMs like GPT-3 and GPT-4. The core innovation in vLLM is its novel memory management system, which is tailored for optimizing the use of GPU memory during the inference process.

1. Create a Deployment
2. For the command put something like `vllm serve lmsys/vicuna-7b-v1.5 --dtype half --quantization bitsandbytes --load-format bitsandbytes`
3. Choose a GPU instance type
4. Choose the `saturn-python-llm` image, version `2024.08.01`
5. click save.

Click "start" to deploy your LLM. Please see the [section on deployments](/docs) to understand how to authenticate with this deployment, as well as restrict access to it.

## VLLm serve options

As long as your [model architecture is supported](https://docs.vllm.ai/en/latest/models/supported_models.html) you should be able to serve your model with vLLM. The parameters `--dtype half --quantization bitsandbytes --load-format bitsandbytes` are recommended in order to reduce the GPU memory foot print of some of the larger models.
