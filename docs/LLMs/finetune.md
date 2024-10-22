# Fine Tuning LLMs
Unsloth is a tool designed to make fine-tuning large language models (LLMs) like Llama-3 and Mistral much faster and more resource-efficient. It allows fine-tuning up to 2.2 times faster and uses significantly less VRAM, often reducing memory requirements by up to 80%. It also supports continual pretraining, which is useful for adapting models to new languages or domain-specific tasks like law or medicine.

## Getting Started

First, we'll define and initialize an LLM and tokenizer to finetune. In our example, we'll use Llama-3.2-3B-Instruct through unsloth's model zoo. The tokenizer is responsible for converting text tokens/embeddings to be fed into the LLM.

```
import torch
from unsloth import FastLanguageModel

max_seq_length = 2048         # Choose any, Unsloth supports RoPE Scaling internally!
dtype = None                  # None for auto detection. Float16 for Tesla T4, V100, Bfloat16 for Ampere+
load_in_4bit = True           # Use 4bit quantization to reduce memory usage. Can be False.


model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/Llama-3.2-3B-Instruct-bnb-4bit",
    max_seq_length = max_seq_length,
    dtype = dtype,
    load_in_4bit = load_in_4bit,
)
```

Next, we'll get the PEFT model of the model we defined above. get_peft_model() transforms the original model by applying LoRA (Low-Rank Adaptation) or other parameter-efficient fine-tuning techniques. This PEFT version of the model is more memory-efficient, optimized for larger batch sizes, and supports features like gradient checkpointing.

After this cell, the model variable will now point to the modified model, which is ready for efficient fine-tuning.

Here's a breakdown of the arguments:

- r = 16: This is a key hyperparameter in LoRA that controls the rank of the low-rank decomposition. It's usually set between 8 and 128, with higher values capturing more information but using more memory.
- target_modules: Specifies which modules within the model should be fine-tuned. Here, common modules such as q_proj (query projection), k_proj (key projection), v_proj (value projection), and others are being fine-tuned.
- lora_alpha = 16: This scaling factor helps balance the LoRA updates during training. Higher values lead to larger updates but also increase the risk of overfitting.
- lora_dropout = 0: Dropout rate for LoRA. A value of 0 disables dropout, which can sometimes improve the performance of LoRA-based models.
- bias = "none": This indicates whether or not to include biases in the layers. The "none" option here suggests that no bias will be applied, which is an optimized choice in some setups.
- use_gradient_checkpointing = "unsloth": This feature saves memory by checkpointing activations, allowing for larger batch sizes and longer context lengths during training. "unsloth" is a new feature that boasts less VRAM usage and larger batch sizes.
- random_state = 3407: Sets a seed for randomization to ensure reproducibility of results.
- use_rslora = False: Rank-stabilized LoRA (rsLoRA) is an advanced technique that can stabilize training at higher ranks, but it's disabled here.
- loftq_config = None: This option enables LoftQ (Low-Rank Quantization), another advanced fine-tuning technique, but it’s not used here.

```
model = FastLanguageModel.get_peft_model(
    model,
    r = 16,                                        # Choose any number > 0 ! Suggested 8, 16, 32, 64, 128
    target_modules = ["q_proj", "k_proj", "v_proj",
                      "o_proj", "gate_proj", "up_proj", "down_proj",],
    lora_alpha = 16,
    lora_dropout = 0,                              # Supports any, but = 0 is optimized
    bias = "none",                                 # Supports any, but = "none" is optimized
    use_gradient_checkpointing = "unsloth",        # True, or [NEW] "unsloth" uses 30% less VRAM, fits 2x larger batch sizes!
    random_state = 3407,
    use_rslora = False,                            # We support rank stabilized LoRA
    loftq_config = None,                           # And LoftQ
)
```

## Step 2: Initialize Chat Template

Next, we'll setup a chat template and prepare our model for inference. We'll then set the model to inference mode (native 2x for faster inference), and send a prompt.


```
from unsloth.chat_templates import get_chat_template

tokenizer = get_chat_template(
    tokenizer,
    chat_template = "llama-3.1",
)
FastLanguageModel.for_inference(model)
```


Let's create a message and apply our chat template.

```
messages = [
    {"role": "user", "content": "What is the Fibonacci number after 89?"},
]
inputs = tokenizer.apply_chat_template(
    messages,
    tokenize = True,
    add_generation_prompt = True,     # Must add for generation
    return_tensors = "pt",
).to("cuda")
```

Now that our pipeline is set up, we can prompt our model as a quick test to see if it is working before fine-tuning. We'll prompt the LLM, decode the output, and print the result.

Notice how the LLM's output doesn't *correctly* provide you with the sequence of Fibonacci numbers.
```
from transformers import TextStreamer
text_streamer = TextStreamer(tokenizer, skip_prompt = True)
_ = model.generate(input_ids = inputs, streamer = text_streamer, max_new_tokens = 128,
                   use_cache = True, temperature = 1.5, min_p = 0.1)
```
Which results in the following output:

```
The Fibonacci sequence is a series of numbers in which each number is the sum of the two preceding ones, usually starting with 0 and 1.

89 is the 30th number in the Fibonacci sequence. To find the next number in the sequence after 89, we can look for the numbers that would precede it:

- The 28th number in the Fibonacci sequence would be 88, and
- The 29th number would be the sum of 89 and 88.

Calculating 89 + 88 gives 177.

Therefore, the Fibonacci number after 89 is 177.<|eot_id|>
```

This is wrong.

### Step 3: Finetune the Model

Since our model could not respond with the correct Fibonacci number, let's finetune our model to teach it how to calculate the sequence. We'll finetune our LLM with FineTome-100k.

First, we'll define a function to apply our chat template to a dataset. Then, we'll download the dataset we would like to use.

```
from datasets import load_dataset

def formatting_prompts_func(examples):
    convos = examples["conversations"]
    texts = [tokenizer.apply_chat_template(convo, tokenize = False, add_generation_prompt = False) for convo in convos]
    return {"text": texts}

dataset = load_dataset("mlabonne/FineTome-100k", split = "train")
```

Next, we'll use standardize_sharegpt to convert ShareGPT style datasets into HuggingFace's generic format. This changes the dataset from looking like:

```
{"from": "system", "value": "You are an assistant"}
{"from": "human", "value": "Hello!"}
{"from": "gpt", "value": "Hello! How can I help you?"}
```

to

```
{"role": "system", "content": "You are an assistant"}
{"role": "user", "content": "Hello!"}
{"role": "assistant", "content": "Hello! How can I help you?"}
```

Then, we'll use map() to apply formatting_prompts_func() to all of our conversations in our dataset, which will apply the chat template to every entry in our dataset. This will prepare our dataset for fine-tuning.

```
from unsloth.chat_templates import standardize_sharegpt

dataset = standardize_sharegpt(dataset)
dataset = dataset.map(formatting_prompts_func, batched=True,)
```

Finally, we can run the training loop using HuggingFace's Supervised Fine-Tuning Trainer (SFT). If you would like to enable or modify more parameters, or to learn more about the current parameters, click here.

```
from trl import SFTTrainer
from transformers import TrainingArguments, DataCollatorForSeq2Seq
from unsloth import is_bfloat16_supported

trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset,
    dataset_text_field = "text",
    max_seq_length = max_seq_length,
    data_collator = DataCollatorForSeq2Seq(tokenizer = tokenizer),
    dataset_num_proc = 2,
    packing = False, # Can make training 5x faster for short sequences.
    args = TrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 4,
        warmup_steps = 5,
        # num_train_epochs = 1, # Set this for 1 full training run.
        max_steps = 60,
        learning_rate = 2e-4,
        fp16 = not is_bfloat16_supported(),
        bf16 = is_bfloat16_supported(),
        logging_steps = 1,
        optim = "adamw_8bit",
        weight_decay = 0.01,
        lr_scheduler_type = "linear",
        seed = 3407,
        output_dir = "outputs",
    ),
)

# Set new trained model to inference mode.
FastLanguageModel.for_inference(model)
```

After fine tuning:

```
from transformers import TextStreamer
text_streamer = TextStreamer(tokenizer, skip_prompt = True)
_ = model.generate(input_ids = inputs, streamer = text_streamer, max_new_tokens = 128,
                   use_cache = True, temperature = 1.5, min_p = 0.1)
```

We get:

```
The Fibonacci sequence starts as follows: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89,...

The next Fibonacci number after 89 is 144.<|eot_id|>
```
