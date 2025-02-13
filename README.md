# StarGANv2-VC: A Diverse, Unsupervised, Non-parallel Framework for Natural-Sounding Voice Conversion

## Yinghao Aaron Li, Ali Zare, Nima Mesgarani
이 페이지는 원래 read.md 페이지를 수정한 것이며, **강조** 부분이 수정 부분임을 미리 알려드립니다.

---
## What is StarGANv2-VC : Unsupervised non-parallel many-to-many voice conversion

<Summary>

- Single Discriminator & Generator
- Treats each speaker as an individual domain
- Utilized pre-trained **Joint Detection and Classification Extraction Network(JDC)** to achieve F0-consistent conversion

<Keywords>

**hx** : Latent feature of the source (source AKA input voice)

**F0**: feature from the convolutional layers of the source

**s** : style code of the reference in the target domain (target AKA voice to converge into)

<Model Preview>

**Generator**: input = mel-spectrogram

**F0 network** : pre-trained JDC network that extracts the fundamental frequency from an input mel-spectrogram.

**Mapping network** : generates a style vector hM. The latent code is sampled from a Gaussian distribution to provide diverse style representations in all domains.

**Style Encoder** : given reference mel-spectogram, it extracts the style code hsty. Similar to the mapping network, first processes an input through shared layers across all domains

**Discriminator** : binary classifier,
---
## 모델 이용 방향 및 개선 방향

- 음성데이터에 대한 전처리 작업
- StarGan 의 Gan 학습을 위해 최소 50 에폭 이상 학습 시켜야함
- 다국어 모델보다는 하나의 언어를 학습시켰을 때 학습 결과가 더 좋았음
- 실제 다국어 모델을 위한 ASR을 배포 예정이라 깃헙 원 제작자가 밝힘. 이는 향후 개선 방향
- discriminator 에 대해서 너무 많은 화자가 들어가는 것은 성능을 저하시킬 수 있다 원 저작자 언급. 이 또한 개선 방향으로 작업 가능
---
> We present an unsupervised non-parallel many-to-many voice conversion (VC) method using a generative adversarial network (GAN) called StarGAN v2. Using a combination of adversarial source classifier loss and perceptual loss, our model significantly outperforms previous VC models. Although our model is trained only with 20 English speakers, it generalizes to a variety of voice conversion tasks, such as any-to-many, cross-lingual, and singing conversion. Using a style encoder, our framework can also convert plain reading speech into stylistic speech, such as emotional and falsetto speech. Subjective and objective evaluation experiments on a non-parallel many-to-many voice conversion task revealed that our model produces natural sounding voices, close to the sound quality of state-of-the-art text-tospeech (TTS) based voice conversion methods without the need for text labels. Moreover, our model is completely convolutional and with a faster-than-real-time vocoder such as Parallel WaveGAN can perform real-time voice conversion.

Paper: https://arxiv.org/abs/2107.10394

Audio samples: https://starganv2-vc.github.io/

## Pre-requisites
1. Python >= 3.7
2. Clone this repository:
```bash
git https://github.com/yl4579/StarGANv2-VC.git
cd StarGANv2-VC
```
3. Install python requirements: 
```bash
pip install SoundFile torchaudio munch parallel_wavegan torch pydub
```
4. Download and extract the [VCTK dataset](https://datashare.ed.ac.uk/handle/10283/3443) 
and use [VCTK.ipynb](https://github.com/yl4579/StarGANv2-VC/blob/main/Data/VCTK.ipynb) to prepare the data (downsample to 24 kHz etc.). You can also [download the dataset](https://drive.google.com/file/d/1t7QQbu4YC_P1mv9puA_KgSomSFDsSzD6/view?usp=sharing) we have prepared and unzip it to the `Data` folder, use the provided `config.yml` to reproduce our models. 

## Training
```bash
python train.py --config_path ./Configs/config.yml
```
Please specify the training and validation data in `config.yml` file. Change `num_domains` to the number of speakers in the dataset. The data list format needs to be `filename.wav|speaker_number`, see [train_list.txt](https://github.com/yl4579/StarGANv2-VC/blob/main/Data/train_list.txt) as an example. 

Checkpoints and Tensorboard logs will be saved at `log_dir`. To speed up training, you may want to make `batch_size` as large as your GPU RAM can take. However, please note that `batch_size = 5` will take around 10G GPU RAM.

**What kind of data to use should be specified in "train.txt" and 'val.txt'. If you change the training data, don't forget to edit this part in the config file. Also note that the number of speakers is also changed according to the input data.**

## pretained model

**You can download epoch files and config files trained in each language from our Notion page. Using that file, you can directly try to infer.**


## Inference

Please refer to [inference.ipynb](https://github.com/yl4579/StarGANv2-VC/blob/main/Demo/inference.ipynb) for details. 

The pretrained StarGANv2 and ParallelWaveGAN on VCTK corpus can be downloaded at [StarGANv2 Link](https://drive.google.com/file/d/1nzTyyl-9A1Hmqya2Q_f2bpZkUoRjbZsY/view?usp=sharing) and [ParallelWaveGAN Link](https://drive.google.com/file/d/1q8oSAzwkqi99oOGXDZyLypCiz0Qzn3Ab/view?usp=sharing). Please unzip to `Models` and `Vocoder` respectivey and run each cell in the notebook.

## ASR & F0 Models

The pretrained F0 and ASR models are provided under the `Utils` folder. Both the F0 and ASR models are trained with melspectrograms preprocessed using [meldataset.py](https://github.com/yl4579/StarGANv2-VC/blob/main/meldataset.py), and both models are trained on speech data only. 

The ASR model is trained on English corpus, but it appears to work when training StarGANv2 models in other languages such as Japanese. The F0 model also appears to work with singing data. For the best performance, however, training your own ASR and F0 models is encouraged for non-English and non-speech data. 

You can edit the [meldataset.py](https://github.com/yl4579/StarGANv2-VC/blob/main/meldataset.py) with your own melspectrogram preprocessing, but the provided pretrained models will no longer work. You will need to train your own ASR and F0 models with the new preprocessing. You may refer to repo [Diamondfan/CTC_pytorch](https://github.com/Diamondfan/CTC_pytorch) and [keums/melodyExtraction_JDC](https://github.com/keums/melodyExtraction_JDC) to train your own the ASR and F0 models, for example. 

## References
- [clovaai/stargan-v2](https://github.com/clovaai/stargan-v2)
- [kan-bayashi/ParallelWaveGAN](https://github.com/kan-bayashi/ParallelWaveGAN)
- [tosaka-m/japanese_realtime_tts](https://github.com/tosaka-m/japanese_realtime_tts)
- [keums/melodyExtraction_JDC](https://github.com/keums/melodyExtraction_JDC)
- [Diamondfan/CTC_pytorch](https://github.com/Diamondfan/CTC_pytorch)

## Acknowledgement
The author would like to thank [@tosaka-m](https://github.com/tosaka-m) for his great repository and valuable discussions.


