# Text-to-Music Personalization

#### Code for [_Investigating Personalization Methods in Text to Music Generation Generation_](https://arxiv.org/abs/2309.11140)

  

Recently, text-to-music generation models have achieved unprecedented results in synthesizing high-quality and diverse music samples from a given text prompt. Despite these advances, it remains unclear how one can generate personalized, user-specific musical concepts, manipulate them, and combine them with existing ones. For example, can one generate a rock song using their personal guitar playing style or a specific ethnic instrument? Motivated by the computer vision literature, we investigate text-to-music \textit{personalization} by exploring two established methods, namely Textual Inversion and Dreambooth. Using quantitative metrics and a user study, we evaluate their ability to reconstruct and modify new musical concepts, given only a few samples. Finally, we provide a new dataset and propose an evaluation protocol for this new task.

- [x] Release code!

- [x] Example code for training and evaluation

- [ ] Gradio app!

- [ ] Release code for Personalized Style Transfer
  
### Install the dependencies and download AudioLDM:
  ```
pip install -r requirements.txt
git clone https://huggingface.co/cvssp/audioldm-m-full
  ```
  
  ### Training Examples

  #### DreamBooth:

  To train the personalization methods for e.g. a short collection of guitar recordings, you can chose "guitar" or "string instrument" as a class, and a not commonly used word like "sks" as an instance word.
  
  ```bash
export MODEL_NAME="audioldm-m-full"
export DATA_DIR="path/to/concept/audios"
export OUTPUT_DIR="path/to/output/dir"
accelerate launch dreambooth_audioldm.py \
  --pretrained_model_name_or_path=$MODEL_NAME \
  --train_data_dir=$DATA_DIR \
  --instance_word="sks" \
  --object_class="guitar" \
  --train_batch_size=1 \
  --gradient_accumulation_steps=4 \
  --max_train_steps=300 \
  --learning_rate=1.0e-06 \
  --output_dir=$OUTPUT_DIR \
  --num_vectors=1 \
  --save_as_full_pipeline 
  ```

#### Textual Inversion:

In textual inversion you just need to specify a placeholder token, that can be any rearely used string. Here we use "<guitar>" as a placeholder token.

```bash
export MODEL_NAME="audioldm-m-full"
export DATA_DIR="path/to/concept/audios"
export OUTPUT_DIR="path/to/output/dir"
accelerate launch textual_inversion_audioldm.py \
  --pretrained_model_name_or_path=$MODEL_NAME \
  --train_data_dir=$DATA_DIR \
  --learnable_property="object" \
  --placeholder_token="<guitar>" \
  --validation_prompt="a recording of a <guitar>" \
  --initializer="mean" \
  --initializer_token="" \
  --train_batch_size=1 \
  --gradient_accumulation_steps=4 \
  --max_train_steps=3000 \
  --learning_rate=5.0e-04 --scale_lr \
  --output_dir=$OUTPUT_DIR \
```


### Example Inference

For Textual inversion, you have to use the placeholder token used in training in the prompts, after loading the learned embeddings in the base model. For Dreambooth, you have to load the fine-tuned model and use \[instance word\] \[class-word\] in the inference prompt.

```python
from pipeline.pipeline_audioldm import AudioLDMPipeline


#Textual Inversion

pipe = AudioLDMPipeline.from_pretrained("audioldm-m-full", torch_dtype=torch.float32).to("cuda")
learned_embedding = "path/to/learnedembedding"
prompt = "A recording of <guitar>"
pipe.load_textual_inversion(learned_embedding)
waveform = pipe(prompt).audios

#DreamBooth
pipeline = AudioLDMPipeline.from_pretrained("path/to/dreambooth/model", torch_dtype=torch.float32).to("cuda")
prompt = "A recording of a sks guitar"
waveform = pipe(prompt).audios
```


# Citation

If you use this code please cite:

```
@article{plitsis2023investigating,
  title={Investigating Personalization Methods in Text to Music Generation},
  author={Plitsis, Manos and Kouzelis, Theodoros and Paraskevopoulos, Georgios and Katsouros, Vassilis and Panagakis, Yannis},
  journal={arXiv preprint arXiv:2309.11140},
  year={2023}
}
```


### Acknowledgments
This code is heavily  base on [AudioLDM](https://github.com/haoheliu/AudioLDM) and [Diffusers](https://github.com/huggingface/diffusers).