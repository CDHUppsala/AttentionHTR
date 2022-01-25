# AttentionHTR

PyTorch implementation of an end-to-end Handwritten Text Recognition (HTR) system based on attention encoder-decoder networks. Scene Text Recognition (STR) benchmark model [1], trained on synthetic scene text images, is used to perform transfer learning from the STR domain to HTR. Different fine-tuning approaches are investigated using the multi-writer datasets: Imgur5K [2] and IAM [3]. 

For more details, refer to our paper at arXiv: https://arxiv.org/abs/2201.09390


## Getting started

* Download our pre-trained models from [here](https://drive.google.com/drive/folders/1h6edewgRUTJPzI81Mn0eSsqItnk9RMeO?usp=sharing). Details [below](#our-pre-trained-models).
* Use the pre-trained models for predictions or fine-tuning on additional datasets using an implementation in [`/model`](https://github.com/dmitrijsk/AttentionHTR/tree/main/model), which is a modified version of the [official PyTorch implementation of the STR benchmark](https://github.com/clovaai/deep-text-recognition-benchmark) [1]. Details [below](#predictions-and-fine-tuning).
* Links to datasets: [Imgur5K](https://github.com/facebookresearch/IMGUR5K-Handwriting-Dataset), [IAM](https://fki.tic.heia-fr.ch/databases/iam-handwriting-database).


## Our pre-trained models

Download our pre-trained models from [here](https://drive.google.com/drive/folders/1h6edewgRUTJPzI81Mn0eSsqItnk9RMeO?usp=sharing). The names of the .pth files are explained in the table below. There are 6 models in total, 3 for each character set, corresponding to the dataset they perform best on.


| Character set    | Imgur5K                 | IAM                 | Both datasets                 |
| :---             |     :---:               |          :---:       |          :---:             |
| Case-insensitive | AttentionHTR-Imgur5K.pth | AttentionHTR-IAM.pth | AttentionHTR-General.pth |
| Case-sensitive   | AttentionHTR-Imgur5K-sensitive.pth | AttentionHTR-IAM-sensitive.pth | AttentionHTR-General-sensitive.pth |

Print the character sets using the Python `string` module: `string.printable[:36]` for the case-insensitive and `string.printable[:-6]` for the case-sensitive character set.

Pre-trained STR benchmark models can be downloaded from [here](https://github.com/clovaai/deep-text-recognition-benchmark).

## Use the models for predictions or fine-tuning

### Partitions

Prepare the train, validation (for fine-tuning) and test (for testing and for predicting on unseen data) partitions with word-level images. For the Imgur5K and the IAM datasets you may use our scripts in [`/process-datasets`](https://github.com/dmitrijsk/AttentionHTR/tree/main/process-datasets).

### LMDB datasets

When using the PyTorch implementation of the STR benchmark model [1], images need to be converted into an LMDB dataset. See [this section](https://github.com/clovaai/deep-text-recognition-benchmark#when-you-need-to-train-on-your-own-dataset-or-non-latin-language-datasets) for details. 

### Predictions and fine-tuning

For fine-tuning and predictions use `train.py` and `text.py`, respectively. In both cases use the following arguments:

* `--Transformation TPS --FeatureExtraction ResNet --SequenceModeling BiLSTM --Prediction Attn` to define architecture.
* `--saved_model` to provide a path to a pre-trained model. In case of `train.py` it will be used as a starting point in fine-tuning and in the case of `test.py` it will be used for predictions.
* `--sensitive` for the case-sensitive character. No such argument for the case-insensitive character set.

Specifically for fine-tuning use: 
* `--FT` to signal that model parameters must be initialized from a pre-trained model in `--saved_model` and not randomly. 
* `--train_data` and `--valid_data` to provide paths to training and validation data, respectively. 
* `--select_data "/"` and `--batch_ratio 1` to use all data. Can be used to define stratified batches.
* `--manualSeed` to assign an integer identifyer for the resulting model. The original purpose of this argument is to set a random seed.

Specifically for predicting use:
* `--eval_data` to provide a path to evaluation data.

Note that `test.py` outputs its logs and a copy of the evaluated model into `/result`.

All other arguments are described inside the scripts. Original instructions for using the scripts in `/model` are available [here](https://github.com/clovaai/deep-text-recognition-benchmark#training-and-evaluation).

For example, to fine-tune one of our case-sensitive models on an additional dataset:

```
CUDA_VISIBLE_DEVICES=3 python3 train.py \
--train_data my_train_data \
--valid_data my_val_data \
--select_data "/" \
--batch_ratio 1 \
--FT \
--manualSeed 1
--Transformation TPS \
--FeatureExtraction ResNet \
--SequenceModeling BiLSTM \
--Prediction Attn \
--saved_model saved_models/AttentionHTR-General-sensitive.pth \
--sensitive
```

To use the same model for predictions:

```
CUDA_VISIBLE_DEVICES=0 python3 test.py \
--eval_data my_unseen_data \
--Transformation TPS \
--FeatureExtraction ResNet \
--SequenceModeling BiLSTM \
--Prediction Attn \
--saved_model saved_models/AttentionHTR-General.pth \
--sensitive
```

## Acknowledgements

* Our implementation is based on [Clova AI's deep text recognition benchmark](https://github.com/clovaai/deep-text-recognition-benchmark).
* The authors would like to thank Facebook Research for [the Imgur5K dataset](https://github.com/facebookresearch/IMGUR5K-Handwriting-Dataset).
* The computations were performed through resources provided by the Swedish National Infrastructure for Computing (SNIC) at Chalmers Centre for Computational Science and Engineering (C3SE). 

## References

[1]: Baek, J. et al. (2019). What is wrong with scene text recognition model comparisons? dataset and model analysis. In *Proceedings of the IEEE/CVF International Conference on Computer Vision* (pp. 4715-4723). https://arxiv.org/abs/1904.01906

[2]: Krishnan, P. et al. (2021). TextStyleBrush: Transfer of Text Aesthetics from a Single Example. *arXiv preprint* arXiv:2106.08385. https://arxiv.org/abs/2106.08385

[3]: Marti, U. V., & Bunke, H. (2002). The IAM-database: an English sentence database for offline handwriting recognition. *International Journal on Document Analysis and Recognition*, 5(1), 39-46. https://doi.org/10.1007/s100320200071

## Citation

```
@article{kass2022attentionhtr,
  title={AttentionHTR: Handwritten Text Recognition Based on Attention Encoder-Decoder Networks},
  author={Kass, D and Vats, E.},
  journal={arXiv preprint arXiv:2201.09390},
  year={2022}
}
```

## Contact

Dmitrijs Kass (dmitrijs.kass@it.uu.se)

Ekta Vats (ekta.vats@abm.uu.se)
