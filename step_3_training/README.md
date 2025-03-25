# Data Generation

## Verification

To start the verification process, you should prepare the required apks, emulator, snapshot, eval task path, output_dir and the document dir.
- Download the APKs for all **DroidTask applications** from [here](https://cloud.tsinghua.edu.cn/d/cb5817c334ee4eb08abc/). Extract the files in the **training/apks** folder
- The emulator we use to run AutoDroid-V2 is `Pixel_6_API_30`. Start the emulator first:

```bash
~/Android/Sdk/emulator/emulator -avd Pixel_6_API_30
```

- Run the following command to start AutoDroid-V2 with verification:

```bash
# These are the default parameters:
BEAM_WIDTH=2 # max width for tree search
MAX_DEPTH=2 # max depth for tree search
MAX_VISIT_TIME=4 # max visit times for tree search
DEVICE_SERIAL="emulator-5554" 
SNAPSHOT_NAME="snap_2024-10-31_22-54-15" # Set as the required snapshot name
APK_DIR="apks" # put all apks in apk_dir, 
EVAL_INPUT_PATH="tasks/tasks.json" # droidtask eval path
OUTPUT_DIR="output/tasks_eval" # result path
DOC_DIR="data/droidtask_docs" # document path

python -m bug_process_dfs_search \
--beam_width $BEAM_WIDTH \
--max_visit_time $MAX_VISIT_TIME \
--max_depth $MAX_DEPTH \
--device_serial $DEVICE_SERIAL \
--snapshot_name $SNAPSHOT_NAME \
--apk_dir $APK_DIR \
--eval_input_path $EVAL_INPUT_PATH \
--output_dir $OUTPUT_DIR \
--doc_dir $DOC_DIR
```

## Model Serving

First download model [here](https://huggingface.co/BlitherBoom/AutoDroid-V2).

Ensure you have vllm installed:

```bash
pip install vllm
```

Serve the model on port 8665:
```bash
MODEL_PATH=AutoDroid-V2 # Set to your model path
vllm serve $MODEL_PATH --dtype auto --host "localhost" --port 8665 --served-model-name AutoDroid-V2
```

## Model Finetuning

First download dataset [here](https://huggingface.co/datasets/BlitherBoom/AutoDroid-V2-Dataset).

First make sure to install [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory.git) following the instruction of the repository:

```bash
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e ".[torch,metrics]"
```

Then add dataset info in `data/dataset_info.json`:

```json
# data/dataset_info.json
{
  ...
  "autodroid": {
    "file_name": "AutoDroid-V2-Dataset.json",
    "formatting": "sharegpt",
    "columns": {
      "messages": "conversations"
    },
    "tags": {
      "role_tag": "from",
      "content_tag": "value",
      "user_tag": "user",
      "assistant_tag": "assistant",
      "system_tag": "system"
    }
  }
}
```

Then begin training with the following command:

```bash
#!/bin/bash
FORCE_TORCHRUN=1 \
CUDA_VISIBLE_DEVICES=1,2,3,4 \
llamafactory-cli train \
--stage sft \
--do_train true \
--finetuning_type full \
--model_name_or_path meta-llama/Llama-3.1-8B-Instruct \
--template llama3 \
--deepspeed examples/deepspeed/ds_z2_config.json \
--dataset autodroid \
--cutoff_len 8192 \
--max_samples 20000 \
--overwrite_cache true \
--preprocessing_num_workers 16 \
--output_dir output/AutoDroid-V2 \
--logging_steps 1 \
--logging_dir output/AutoDroid-V2/log \
--report_to tensorboard \
--save_total_limit 1 \
--save_strategy "no" \
--plot_loss true \
--overwrite_output_dir true \
--per_device_train_batch_size 1 \
--gradient_accumulation_steps 8 \
--learning_rate 1e-5 \
--weight_decay 0.1 \
--adam_beta2 0.95 \
--num_train_epochs {epoch_num} \
--lr_scheduler_type cosine \
--warmup_ratio 0.01 \
--bf16 true \
--ddp_timeout 180000000 \
--val_size 0.0 \
--per_device_eval_batch_size 1 \
--eval_strategy "no" \
```