# Advanced: Distributed training sample project

One of the areas that we focus on with Gradient is distributed training which can be extremely valuable in terms of decreasing training time but is notoriously difficult to orchestrate.  We put together a sample project which provides example code for both singlenode and multinode \(distributed\) training to showcase how easy it is to take a basic training job and scale it up across multiple instances on Gradient.   

The sample project is an object detection demo based on Detectron using PyTorch and the COCO dataset.  It also includes a step at the end to take your trained model and deploy it as an API endpoint. Here is a link the project on GitHub: 

{% embed url="https://github.com/Paperspace/object-detection-segmentation" %}

## Training & Evaluation

We provide an example script in "training/train\_net.py" that is made to train your model. You can use this as a reference to write your own training script.

### Setup Dataset

This demo has built-in support for a few datasets. Please check out [docs on using Datasets with Gradient](https://docs.paperspace.com/gradient/experiments/using-experiments/experiment-datasets)

The datasets are assumed to exist in a directory `/data/DATASET`. Under this directory, the script will look for datasets in the structure described below, if needed.

```text
/data/coco/
```

```text
# Example Code 
dataset_dir = os.path.join(os.getenv("DETECTRON2_DATASETS", "/data"), "coco")
```

**Expected dataset structure for COCO instance/keypoint detection:**

```text
coco/
  annotations/
    instances_{train,val}2017.json
    person_keypoints_{train,val}2017.json
  {train,val}2017/
    # image files that are mentioned in the corresponding json
```

You can download a tiny version of the COCO dataset, with `training/download_coco.sh`.

### **COCO Dataset**

Probably the most widely used dataset today for object localization is COCO: Common Objects in Context. Provided here are all the files from the 2017 version, along with an additional subset dataset created by fast.ai. Details of each COCO dataset is available from the COCO dataset page. The fast.ai subset contains all images that contain one of five selected categories, restricting objects to just those five categories; the categories are: chair couch tv remote book vase.

[fast.ai subset](https://s3.amazonaws.com/fast-ai-coco/coco_sample.tgz)

[Train images](https://s3.amazonaws.com/fast-ai-coco/train2017.zip)

### Run Training on Gradient

#### Gradient CLI Installation

How to install Gradient CLI - [docs](https://docs.paperspace.com/gradient/get-started/install-the-cli)

```text
pip install gradient
```

Then make sure to [obtain an API Key](https://docs.paperspace.com/gradient/get-started/install-the-cli#obtaining-an-api-key), and then:

```text
gradient apiKey XXXXXXXXXXXXXXXXXXX
```

### Train on a single GPU

_Note: training on a single will take a long time, so be prepared to wait!_

```text
gradient experiments run singlenode \
  --name mask_rcnn \
  --projectId <some project> \
  --container devopsbay/detectron2-cuda:v0 \
  --machineType p2.xlarge \
  --command "sudo python training/train_net.py --config-file training/configs/mask_rcnn_R_50_FPN_1x.yaml --num-gpus 1 SOLVER.IMS_PER_BATCH 2 SOLVER.BASE_LR 0.0025" \
  --workspace https://github.com/Paperspace/object-detection-segmentation.git \
  --datasetName coco \
  --datasetUri s3://fast-ai-coco/train2017.zip \
  --clusterId <cluster id>
```

The coco dataset is downloaded to the `./data/coco/traing2017` directory. Model results are stored in the `./models` directory.

### Running distributed multi-node on a Gradient Enterprise private cloud cluster

In order to run an experiment on a [Gradient private cluster](https://docs.paperspace.com/gradient/gradient-private-cloud/about), we need to add a few additional parameters:

```text
gradient experiments run multinode \
  --name mask_rcnn_multinode \
  --projectId <some project> \
  --workerContainer devopsbay/detectron2-cuda:v0 \
  --workerMachineType p2.xlarge \
  --workerCount 7 \
  --parameterServerContainer devopsbay/detectron2-cuda:v0 \
  --parameterServerMachineType p2.xlarge \
  --parameterServerCount 1 \
  --experimentType GRPC \
  --workerCommand "sudo python training/train_net.py --config-file training/configs/mask_rcnn_R_50_FPN_1x.yaml --num-machines 8" \
  --parameterServerCommand "sudo python training/train_net.py --config-file training/configs/mask_rcnn_R_50_FPN_1x.yaml --num-machines 8" \
  --workspace https://github.com/Paperspace/object-detection-segmentation.git \
  --datasetName coco \
  --datasetUri s3://fast-ai-coco/train2017.zip \
  --clusterId <cluster id>
```

## Deploying your model on Gradient

This example will load the previously trained model and launch a web app application with a simple interface for making predictions.

```text
gradient deployments create \
  --name mask_rcnn4 --instanceCount 1 \
  --imageUrl devopsbay/detectron2-cuda:v0 \
  --machineType p2.xlarge \
  --command "sudo python demo/app.py" \
  --workspace https://github.com/Paperspace/object-detection-segmentation.git \               
  --deploymentType Custom \
  --clusterId <cluster id> \
  --modelId <model id>
```

![Example](https://github.com/Paperspace/object-detection-segmentation/raw/master/demo/samples/detect.jpeg?raw=true)



