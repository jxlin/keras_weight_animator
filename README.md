# Keras Weight Animator

Save [Keras](http://keras.io) weight matrices as videos to better understand what and how your neural network models are learning.

## Getting Started

### Dependencies

```bash
# clone the repo, preferably somewhere in your PYTHONPATH
git clone https://github.com/brannondorsey/keras_weight_animator
cd keras_weight_animator

pip install -r requirements.txt
```

In order to render videos from the saved weight images you must also have the following packages to be installed on your machine:

- [GNU Parallel](https://www.gnu.org/software/parallel/)
- [ImageMagick](https://www.imagemagick.org/script/index.php)
- [FFMPEG](https://ffmpeg.org/download.html)

### Using the Module

This package is exposed as a module named `keras_weight_animator`. It exposes a [Keras callback](https://keras.io/callbacks/) function that you can include in any model `fit` method.

```python
## you can optionally add this module to your python path
## by including its parent directory in sys.path
# import sys
# sys.path.append('path/to/keras_weight_animator/..')

from keras_weight_animator import image_saver_callback

# assumes a keras model named "model"
callbacks = [image_saver_callback(model, 'weight_image_sequences')]

model.fit(X_train, y_train, callbacks=callbacks)
```

The two required parameters to `image_saver_callback(...)` are the Keras `model` and an `output_directory` to periodically save weight images to. By default, `keras_weight_animator` saves layer weights every 100 batches to `output_directory` as PNGs in folders named `epoch_XXX-layer_NAME-weights_YY`. Once training is complete, you can optionally create short animated video clips from the image sequences saved in `output_directory` using [`bin/create_image_sequence.sh](bin/create_image_sequence.sh) path/to/output_directory`. This will use `parallel`, `mogrify`, and `ffmpeg` to create a `.mp4` from the image sequences located in each folder of `output_directory`. Video files will be named like `epoch_XXX-layer_NAME-weights_YY.mp4`. 

## Optional Parameters 

`weight_image_sequences(...)` takes a variety of optional keyword arguments.

- `interval` (default=`100`): Save weight images every `interval` batches. `interval=1` would save weights for every batch.
- `cmap` (default='gray'): [Matplotlib color map](http://matplotlib.org/users/colormaps.html) name. I recommend trying some diverging color maps, especially `RdBu`. 
- `render_videos` (default=`False`): Optionally make a system call to `create_image_sequences.sh <output_directory>` at the end of `model.fit()` (using the Keras `on_train_end(...)` function internally). Setting this to `True` will automagically render `.mp4` videos for you (watch the console for errors).

```bash
# example
callbacks = [image_saver_callback(model, 'weight_image_sequences', interval=1000, cmap='gray', render_videos=True)]
model.fit(X_train, y_train, callbacks=callbacks)
```

## Examples

I've included an example usage of the module in [`examples/.py`](example.py). This example uses smartphone accelerometer data from [WISDM](http://www.cis.fordham.edu/wisdm/dataset.php) to classify human activity tasks like walking, standing, sitting, walking upstairs, etc...

This example uses a one layer LSTM to classify a set of 60 data points (representing three seconds of data sampled at 20hz) as belonging to one of six classes. It outputs image sequences and videos to `data/wisdm`.

```bash
# download and uncompress the WISDM data
cd data
curl http://www.cis.fordham.edu/wisdm/includes/datasets/latest/WISDM_ar_latest.tar.gz -o WISDM_ar_latest.tar.gz
tar -xzf WISDM_ar_latest.tar.gz
rm WISDM_ar_latest.tar.gz
cd ..

python example/wisdm.py
```

## Thoughts

Using a bash script to leverage parallel, ImageMagick, and FFMPEG isn't necessarily the most elegant solution, but its the one I had time for. The goal of this here lil' project was to write a quick tool that allows me to better understand how weights change over mini-batch updates in a variety of neural networks. Perhaps in the future I will come back and clean up some of the inelegancies. If you have interest in contributing or maintaining a cleaner version of this lib, please [reach out](mailto: brannon@brannondorsey.com). 

## Attribution and License

This module is ♥ © Brannon Dorsey 2017 ♥, released under an MIT License. You are free to use, modify, distribute, sell, etc... this software under those terms. Example data is from the WIreless Sensor Datamining Actitracker dataset published by Fordham University:

```
Jennifer R. Kwapisz, Gary M. Weiss and Samuel A. Moore (2010). Activity Recognition using Cell Phone Accelerometers, Proceedings of the Fourth International Workshop on Knowledge Discovery from Sensor Data (at KDD-10), Washington DC. [PDF]
```

GNU Parallel is adamant about citation to the point of excession IMHO, but for what its worth, here is their bibtex:

```
@article{Tange2011a,
  title = {GNU Parallel - The Command-Line Power Tool},
  author = {O. Tange},
  address = {Frederiksberg, Denmark},
  journal = {;login: The USENIX Magazine},
  month = {Feb},
  number = {1},
  volume = {36},
  url = {http://www.gnu.org/s/parallel},
  year = {2011},
  pages = {42-47},
  doi = {http://dx.doi.org/10.5281/zenodo.16303}
}

```