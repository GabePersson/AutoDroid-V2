# Generate Documents

## Warning: 
Generate all the apps' document may cost up to $100, if you do not want to generate the document, you can skip this `Genrate Documents` section and move to the next section. 

## Preparation: 
Go to gen_doc.py, set your `OPENAI_API_KEY` and `OPENAI_API_URL` to your openai api key and base url. 

## Run: 
`python gen_doc.py -d <dir_path of the explore data> -m gpt-4o -t <tag of the current experiment> -o <output_dir>`

-d: the directory path of the explore data, in the following format:

```
<dir_path>/
├── log.yaml
├── states
│   ├── screen_<tag1>.png
│   ├── screen_<tag2>.png
│   ├── ...
│   ├── state_<tag1>.json
│   ├── state_<tag2>.json
│   ├── ...
```
## Setup Environment
### Prerequisite
1. Python

2. Java

3. Android SDK

4. Add platform_tools directory in Android SDK to PATH

### Install


```
download the apks from https://cloud.tsinghua.edu.cn/f/eeea64534064438abbc4/, unzip the apks.zip and put the apks under apks folder
git clone git@github.com:wenh18/droidbot-llm.git
cd droidbot-llm
pip install -e .
```

## Run on DroidTask Dataset
```
Specify the app you want to test by setting app_name = <app_name> in run.py

python run.py
```
It loads tasks in tasks.json, and run all the tasks in <app_name>. 

# Debug

You can get the results in test/<app_name>/<task_index>: 

- `task.json` records the task that is to be finished. 
- `code.txt` is the raw LLM-generated code for this task. 
- `compiled_code.txt` is the revised LLM-generated code that is executable in python. 
- `log.yaml` is the execution trace, where you can get the UI state, actual action information, and the executing code line, etc. 
- `error.json` records the error encountered when executing the code (if any). 
- Folders `1, 2, 3, ...` are the retry information, also including `code.txt`, `log.yaml`, ...

You can see the document for each app at `data/docs_droidtask_v3`

# TODO
1. Debug: I noticed the error of missing some text in the UI in Settings page of `voicerecorder` app (you can try by running the task1), we may need some fuzzy matching methods for the `match` api. 

2. Change the foundation model from gpt-4o to llama-3.1

3. I currently fail to launch the former android virtual machine as well as the snapshot (containing the initial state of each app for testing). So we are now running on the new machine, where I didn't create the snapshot. When we finish the former steps, I need to create a new snapshot for better evaluation. 

## Acknowledgement

1. The DroidBot Project: [DroidBot](https://github.com/honey/droidbot)

-o: the output directory to store the generated documentation.