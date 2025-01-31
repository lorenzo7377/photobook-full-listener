# Listener Model for PhotoBook Game

This repo houses the official PyTorch implementation for the following paper

- **Listener Model for the PhotoBook Referential Game with CLIPScores as Implicit Reference Chain**  
  Shih-Lun Wu, Yi-Hui Chou, and Liangze Li  
  _Annual Meeting of the Association of Computational Linguistics (**ACL**) 2023_  
  [[**ArXiv**](https://arxiv.org/abs/2306.09607)]
## Installation

```bash
conda create -n photobook python=3.8.10
conda activate photobook
pip install -r requirements.txt

python
>>> import nltk
>>> nltk.download('punkt')
```

<!-- * get `logs.zip` and `images.zip` at [photobook_dataset](https://github.com/dmg-photobook/photobook_dataset/), unzip and save inside `data/` -->

## Data Preprocessing

1. Read `../data/data_splits.json` and save processed log data to `../data/{split}_sections.pickle`

     ```bash
     cd preprocess
     python dialogue_segmentation.py
     ```

2. Generate CLIP score

* Read `../data/{split}_sections.pickle` and save data to `../data/{split}_clean_sections.pickle`

  ```bash
  python process_section.py
  ```

3. Extract image features with Segformer

* Save features at `../data/image_feats.pickle`, the saved data is a dictionary (key: image path, value: hidden features)

  ```bash
  python process_image.py
  ```

## Training and Inference

* Edit hyperparams in `model/variables.py`
* Training (with the best-performing configuration)

  ```zsh
  python3 train.py config_paper/EXPERIMENT_JSON exp/EXPERIMENT_NAME
  ```

* To reproduce logged ablations, use different `EXPERIMENT_JSON` for each run.
    * Ours - `vlscore_all.json`
    * + VisAttn - `vlscore_visattn.json`
    * - CLIPScore - `base_deberta.json`
    * - CLIPScore + VisAttn - `visattn.json`
    * - Dense learning signals - `vlscore_all.json` (change `DLS` to `False`
        in `model/variables.py`)
* Tweak random seeds for optimal performance.

* Inference

  ```zsh
  python3 inference.py exp/EXPERIMENT_NAME
  ```

## Baseline Model Adapted from [Takmaz et al., 2020](https://aclanthology.org/2020.emnlp-main.353/)
* Model implementation is based on [official PhotoBook repo](https://github.com/dmg-photobook/ref-gen-photobook/blob/main/models/listener/models/model_bert_att_ctx_hist.py)
* To run the Takmaz baseline

  ```zsh
  cd takmaz_baseline/
  python3 train.py
  ```

* To use reference chains extracted using CLIPScore
  * set `REF_CHAIN_PATH = ref_chain_img_clipscore.pickle` in `takmaz_baseline/variables.py`

* Note that different random seeds might be needed in
  `takmaz_baseline/variables.py` for optimal results in different experiments.

## Utterance-based Reference Chain Extraction
* This part is largely inherited from [official PhotoBook repo](https://github.com/dmg-photobook/ref-gen-photobook/tree/main/chain-extraction), except that we add the option to use CLIPScore as the scoring metric.

* To reproduce the whole extraction and evaluation procedure described in [(Takmaz et al., 2020)](https://aclanthology.org/2020.emnlp-main.353/), run these commands in `chain-extraction`.

  ```zsh
  python src/extract_segments.py out/all_segments.dict --stopwords --meteor --from_first_common --utterances_as_captions
  python src/make_chains.py out/all_segments.dict out/all_chains.json --score f1
  python src/make_gold_chains.py out/gold_chains.json --from_first_common --first_reference_only
  python src/make_dataset.py out/all_chains.json out/gold_chains.json out/dataset

  python src/extract_segments.py out/eval_segments.dict --path_game_logs data/logs/test_logs.dict --stopwords --meteor --from_first_common --utterances_as_captions
  python src/eval_chains.py out/eval_segments.dict
  ```

* To alternatively use CLIPScore as part of the scoring in extraction, just
  add the `--clipscore` option when running `extract_segments.py` above.
