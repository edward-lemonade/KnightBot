# Chess Robot

This repo captures my journey to build a prototype chess robot, which may be
turned into a commercial product in the future.

![yolov5][6]

## What is it?

The robot should

 * Visually recongize a physical chess board and chess pieces in it
 * Use A.I. to determine the next move
 * Drive a robot arm to physically move the pieces
 * Verbally interact with the opponent at each stage

## The idea and the challenges

There are existing A.I. models that I can use to do the visual recognition of
the board and pieces. Stockfish is an open source A.I. chess engine that can be
used to determine the chess moves. I have a retired VEX robot and some spare
parts that I can use to build the robot arm. Maybe hack one of my father's
1st gen echo dots to do the voice recognition and Text-To-Speech. For the
brain, I will probably just use my laptop for now. Once it is tested
successfully, maybe I will transfer the brain code to a mini PC powered with a
battery. The visual sensor can be as simple as a 1080p web cam.

Everything is easier said than done. I may have to deal with a lot of engineering
challenges, as shown below.

 * What AI model to use for visual recognition?
 * Do I need to fine-tune or train my own AI model?
 * How to convert the recognized board and pieces into something Stockfish can take?
 * What API does Stockfish provide to generate moves?
 * How to connect the VEX robot arm, VEX brain and my laptop?
 * What would the robot arm look like?
 * How to calculate the arm position and control the movement with visual? 

## Design for visual chessboard recognition

### Components

**Notes**:

Pivoted from using **scipy.cluster.hierachy** to 2 level **1D k-means** of
**fixed** number grouping. It is simplified that the result directly form the
board grid. The end result is very good.

 * Visual intake: use opencv-python library
 * Board detection:
   * use canny-edge and houghline algorithm to find the potential grid lines
   * calculate all the intersections
   * use 1d k-means to group the intersections into 11 rows
   * for each role use 1d k-means to group the intersections into 11 columns
   * the average of the group coordinates form rows and columns of the grid
 * Chess piece recognition - Fine tune pre-trained yolov5
   * Yolov5 is a pre-trained neural network model that does object detection
   * We need to fine-tune the model to recognize each pieces
   * Available datasets can be download from universe.roboflow.com
   * Once board is detected, the image is passed to the model for inference

**Notes**:

Pivoted from using **VGG16** image labeling model to **yolov5** object detection model. The benefits include
* multiple object detection in one prediction, better performance
* bounding box are included

### Test and dataset capture

```
python3 -m venv .
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python capture.py
```
You can follow the prompt in the capturing screen to adjust the parameters with
hot keys such as w/s for changing theta ratio, [] for minLen etc. Press
**space** to capture the frame and start cropping images for each cell. Repeat
until **q** is pressed.

![Canny Edge Sample][2] ![hugh_line][3] ![K-Means Gird][4]

### My setup
![My setup][1]

### Training with VGG16 (Deprecated)

Open the [chessv.ipynb](./chessv.ipynb) in Jupyter Notebook and run each cell
to see the output.

**Notes:**
VSCode has builtin support for Jupyter Notebooks, so you can open the file in
VSCode directly.

The first phase of training yield below confusion matrix.

![Confusion Matrix][5]

### Training with Yolov5

Open the [chessv-yolov5.ipynb](./chessv-yolov5.ipynb) in JupyterNotebook or
VSCode to see the output.

Unfortunately, my laptop is not powerful enough to train the median sized
yolov5m.pt. I had to pivot to use a Google Vertex AI workbench to train with a
large size yolov5x.pt.

Open the [chessv5-yolov5-workbench.ipynb](./chessv-yolov5-workbench.ipynb) to
see the training progress.

After downloading the best weights file **best.pt** after about 60 epochs,
below are some impressive results.

![yolov5 inference][6]

The above captured image was successfully translated into FEN

**4rk2/3n1p2/5Qb1/2P5/2NB4/1p4q1/3P1R2/2K5**

Run below commands to see the tensor board for the yolov5 training.

```
cd runs/train/exp2_google
tensorboard.exe --logdir .
```
You should get a URL for the board such as http://localhost:6006/

![yolov5 overall training performance][7]

## Next Step

 - Keep training the yolov5 model with more dataset
 - Produce local datasets with [label studio](https://labelstud.io/)
 - Design robot arm

[1]: images/my_setup.jpg "My Setup"
[2]: images/sample_canny_edge.jpg
[3]: images/hough_line.jpg
[4]: images/k-mean-grid.jpg
[5]: images/vgg16_confusion_matrix.png "Confusion Matrix"
[6]: images/yolov5_inference2.jpg
[7]: images/yolov5_training_performance.PNG "Yolov5 Training Performance"
