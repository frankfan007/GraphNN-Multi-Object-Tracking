## MOT (Multi Object Tracking) using Graph Neural Networks

<p align="center">
  <img src="anim.gif" width="500">
</p>

This repository largely implements the approach described in [Learning a Neural Solver for Multiple Object Tracking](https://arxiv.org/abs/1912.07515). This implementation achieves ~58% MOTA on the MOT16 test set using PCA to reduce the dimensionality of the visual features. Note that the paper learns the network providing the ReID embeddings in an end-to-end way which is also supported in this implementation but was not trained fully by myself due to hardware limitations.

Note that this is not the offical implementation of the paper which will be published [here](https://github.com/dvl-tum/mot_neural_solver).

### Setup
Install the conda environment
```
conda create -f environment.yml
```
Install [torchreid](https://github.com/KaiyangZhou/deep-person-reid)
```
pip install git+https://github.com/KaiyangZhou/deep-person-reid.git
```

---

### Train
The implementation supports the [MOT16](https://motchallenge.net/data/MOT16/) dataset for training and testing.

#### Preprocessing
Run `python src/data_utils/preprocessing.py` which creates and saves a graph representation for the scene. In detail, the sequences are 
split into subsets with one overlapping frame each.
``` 
usage: preprocessing.py [-h] [--output_dir OUTPUT_DIR] [--pca_path PCA_PATH]
                        [--dataset_path DATASET_PATH] [--mode MODE]
                        [--threshold THRESHOLD] [--save_imgs]
                        [--device {cuda,cpu}]

optional arguments:
  -h, --help            show this help message and exit
  --output_dir OUTPUT_DIR
                        Outout directory for the preprocessed sequences
  --pca_path PCA_PATH   Path to the PCA model for reducing dimensionality of
                        the ReID network
  --dataset_path DATASET_PATH
                        Path to the root directory of MOT dataset
  --mode MODE           Use train or test sequences (for test additional work
                        necessary)
  --threshold THRESHOLD
                        Visibility threshold for detection to be considered a
                        node
  --save_imgs           Save image crops according to bounding boxes for
                        training the CNN (only required if this is wanted)
  --device {cuda,cpu}   Device to run the preprocessing on.
```
`PCA_PATH` is a serialized Scikit-Learn PCA model which can be fit using the `fit_pca(...)` function in 
`src/data_utils/preprocessing.py`. Already fit PCA model can be downloaded [here](https://drive.google.com/file/d/1gXDdgKgbkgqpWnYbQ1nBkNGckJZBxi_k/view?usp=sharing). `MODE` should be set to `train`.

#### Training script
Training accepts the preprocessed version of the dataset only.
```
usage: train.py [-h] --name NAME --dataset_path DATASET_PATH
                [--log_dir LOG_DIR] [--base_lr BASE_LR] [--cuda]
                [--workers WORKERS] [--batch_size BATCH_SIZE]
                [--epochs EPOCHS] [--train_cnn] [--use_focal]

optional arguments:
  -h, --help            show this help message and exit
  --name NAME           Name of experiment for logging
  --dataset_path DATASET_PATH   Directory of preprocessed data
  --log_dir LOG_DIR     Directoy where to store checkpoints and logging output
  --base_lr BASE_LR
  --cuda
  --workers WORKERS
  --batch_size BATCH_SIZE
  --epochs EPOCHS
  --train_cnn           Choose to train the CNN providing node embeddings
  --use_focal           Use focal loss instead of BCE loss for edge classification
```

### Testing
#### Obtain detections
Run `src/data_utils/run_obj_detect.py` to use a pre-trained FasterRCNN for detection on the sequences. The FasterRCNN model weights can be downloaded [here](https://drive.google.com/file/d/12FlTPh5gjPqvY2u0N5Wxn089Hb1gFUb5/view?usp=sharing).
```
usage: run_obj_detect.py [-h] [--model_path MODEL_PATH]
                         [--dataset_path DATASET_PATH] [--device DEVICE]
                         [--out_path OUT_PATH]

Run object detection on MOT16 sequences and generate output files with
detections for each sequence in the same format as the `gt.txt` files of the
training sequences

optional arguments:
  -h, --help            show this help message and exit
  --model_path MODEL_PATH
                        Path to the FasterRCNN model
  --dataset_path DATASET_PATH
                        Path to the split of MOT16 to run detection on.
  --device DEVICE
  --out_path OUT_PATH   Output directory of the .txt files with detections
```
The output files can then easily be copied to the respective sequence folder, e.g., as `MOT16-02/gt/gt.txt` for the
produced `MOT16-02.txt` file.  
In this way, we can just use the same pre-processing script from the training script.

#### Preprocessing
See Train section. Use with `--mode test` to use the test folder of the MOT16 dataset.

#### Inference
Run `src/data_utils/inference.py` to obtain tracks as `.txt.` file from a single (!), preprocessed sequence. This means this script has to be executed for each test sequence independently. Pretrained model weights can be downloaded from [here](https://drive.google.com/file/d/1Ocy1ugsnIgCdb-DnKSuyI2127VTvlTq_/view?usp=sharing).
```
usage: inference.py [-h] [--preprocessed_sequence PREPROCESSED_SEQUENCE]
                    [--net_weights NET_WEIGHTS] [--out OUT]

optional arguments:
  -h, --help            show this help message and exit
  --preprocessed_sequence PREPROCESSED_SEQUENCE
                        Path to the preprocessed sequence (!) folder
  --net_weights NET_WEIGHTS
                        Path to the trained GraphNN
  --out OUT             Path of the directory where to write output files of
                        the tracks in the MOT16 format

```

---

### Acknowledgements
* For the ReID network providing node embeddings [this](https://arxiv.org/abs/1905.00953) approach is used 
implemented in [torchreid](https://github.com/KaiyangZhou/deep-person-reid). 
* Dataset implementations, a pre-trained FasterRCNN and other utility in `src/tracker` were provided by the challenge.
