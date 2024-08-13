# Speech to Speech: An Open-Sourced and Modular GPT-4 Pipeline

## Approach

### Structure
This repository implements a speech-to-speech cascaded pipeline with consecutive parts:
1. **Voice Activity Detection (VAD)**: [silero VAD v5](https://github.com/snakers4/silero-vad)
2. **Speech to Text (STT)**: Whisper models (including distilled versions)
3. **Language Model (LM)**: Any instruct model available on the [Hugging Face Hub](https://huggingface.co/models)! 🤗
4. **Text to Speech (TTS)**: [Parler-TTS](https://github.com/huggingface/parler-tts)

### Modularity
The pipeline aims to provide a fully open and modular approach, leveraging models available on the Transformers library via the Hugging Face hub. The level of modularity intended for each part is as follows:
- **VAD**: Uses the implementation from [Silero's repo](https://github.com/snakers4/silero-vad).
- **STT**: Uses Whisper models exclusively; however, any Whisper checkpoint can be used, enabling options like [Distil-Whisper](https://huggingface.co/distil-whisper/distil-large-v3) and [French Distil-Whisper](https://huggingface.co/eustlb/distil-large-v3-fr).
- **LM**: This part is fully modular and can be changed by simply modifying the Hugging Face hub model ID. Users need to select an instruct model since the usage here involves interacting with it.
- **TTS**: The mini architecture of Parler-TTS is standard, but different checkpoints, including fine-tuned multilingual checkpoints, are supported.

The code is designed to facilitate easy modification. Each component is implemented as a class and can be re-implemented to match specific needs.

## Setup

Clone the repository:
```bash
git clone https://github.com/eustlb/speech-to-speech.git
cd speech-to-speech
```

Install the required dependencies:
```bash
pip install -r requirements.txt
```

## Usage

The pipeline can be run in two ways:
- **Server/Client approach**: Models run on a server, and audio input/output are streamed from a client.
- **Local approach**: Uses the same client/server method but with the loopback address.

### Server/Client Approach

To run the pipeline on the server:
```bash
python s2s_pipeline.py --recv_host "0.0.0.0" --send_host "0.0.0.0"
```

Then run the client locally to handle sending microphone input and receiving generated audio:
```bash
python listen_and_play.py --host <IP address of your server>
```

### Local Approach
Simply use the loopback address:
```bash
python s2s_pipeline.py --recv_host localhost --send_host localhost
python listen_and_play.py --host localhost
```

## Command-line Usage

### Model Parameters

`model_name`, `torch_dtype`, and `device` are exposed for each part leveraging the Transformers' implementations: Speech to Text, Language Model, and Text to Speech. Specify the targeted pipeline part with the corresponding prefix:
- `stt` (Speech to Text)
- `lm` (Language Model)
- `tts` (Text to Speech)

For example:
```bash
--lm_model_name google/gemma-2b-it
```

### Generation Parameters

Other generation parameters of the model's generate method can be set using the part's prefix + `_gen_`, e.g., `--stt_gen_max_new_tokens 128`. These parameters can be added to the pipeline part's arguments class if not already exposed (see `LanguageModelHandlerArguments` for example).

### Notable Parameters

#### VAD Parameters
- `--thresh`: Threshold value to trigger voice activity detection.
- `--min_speech_ms`: Minimum duration of detected voice activity to be considered speech.
- `--min_silence_ms`: Minimum length of silence intervals for segmenting speech, balancing sentence cutting and latency reduction.

#### Language Model
- `--init_chat_role`: Defaults to `None`. Sets the initial role in the chat template, if applicable. Refer to the model's card to set this value (e.g. for [Phi-3-mini-4k-instruct](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct) you have to set `--init_chat_role system`)
- `--init_chat_prompt`: Defaults to `"You are a helpful AI assistant."` Required when setting `--init_chat_role`.

#### Speech to Text
- `--description`: Sets the description for Parler-TTS generated voice, with defaults describing the vocal qualities and environment of the speaker.

- `--play_steps_s`: Specifies the duration of the first chunk sent during streaming output from Parler-TTS, impacting readiness and decoding steps.

## Citations

### Distil-Whisper
```
@misc{gandhi2023distilwhisper,
      title={Distil-Whisper: Robust Knowledge Distillation via Large-Scale Pseudo Labelling},
      author={Sanchit Gandhi and Patrick von Platen and Alexander M. Rush},
      year={2023},
      eprint={2311.00430},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```

### Silero VAD
```
@misc{Silero VAD,
  author = {Silero Team},
  title = {Silero VAD: pre-trained enterprise-grade Voice Activity Detector (VAD), Number Detector and Language Classifier},
  year = {2021},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/snakers4/silero-vad}},
  commit = {insert_some_commit_here},
  email = {hello@silero.ai}
}
```