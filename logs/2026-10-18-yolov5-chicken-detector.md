# Engineering Field Report 001 — YOLOv5 Chicken Detector

**Date:** October 18, 2026  
**Project:** Property Intelligence / Blue Iris / CodeProject.AI  
**Result:** Infrastructure repaired and custom-model request verified; reliable `Chicken` detections **not yet achieved**.

## Objective

Detect chickens on the EGGCOOP security camera using a custom YOLOv5 model that emits the class `Chicken`, instead of relying on a generic COCO `bird` label. The longer-term objective is a reusable visual-learning workflow for objects actually encountered on the property—not only animals, but tools, wallets, keys, eggs, and equipment.

## Starting environment

- Windows 11
- CodeProject.AI Server 2.9.5
- Blue Iris, including the EGGCOOP camera
- NVIDIA RTX 3080 GPU
- YOLOv5 object detection module: `ObjectDetectionYOLOv5-6.2`
- Training module: `TrainingObjectDetectionYOLOv5`

This report records observed behavior from this work session; it is **not** a universal installation guide.

## Work completed

### 1. Repaired the YOLOv5 inference environment

The YOLOv5 module initially failed to start. Its shared Python 3.7 virtual environment had missing or inconsistent dependencies. We repaired the environment and verified the module could launch and process live inference requests using CUDA.

Components verified or repaired included:

- Python 3.7.9 virtual environment
- PyTorch 1.13.0 + CUDA 11.7 build
- torchvision 0.14.0 + CUDA 11.7 build
- YOLOv5 6.2.3
- NumPy 1.21.6
- Pillow 9.5.0
- `aiohttp==3.8.6`, after discovering an incomplete import that lacked `ClientSession`

The working environment path was:

```text
C:\Program Files\CodeProject\AI\runtimes\bin\windows\python37\venv\Scripts\python.exe
```

**Verified result:** the YOLOv5 inference module stayed running and handled requests from Blue Iris. General-purpose detection could return objects such as `bird` and `person`.

### 2. Trained the first custom model

Training dataset: `farm_chickens_v1`

- 100 training images
- 33 validation images
- 33 test images
- 100 training epochs
- Intended custom class: `Chicken`
- Output checkpoint: `chicken_detector_v1.pt`

The model was placed at:

```text
C:\Program Files\CodeProject\AI\modules\ObjectDetectionYOLOv5-6.2\custom-models\chicken_detector_v1.pt
```

**Verified result:** the checkpoint existed and the training run completed. **Not verified:** that the model learned to recognize chickens reliably in deployment.

### 3. Connected the custom model to Blue Iris

In the EGGCOOP camera's AI settings, the additional custom model name was set to:

```text
chicken_detector_v1
```

The logs showed requests using that model name:

```text
Detecting using chicken_detector_v1
command 'custom' ['No objects found']
```

Meanwhile, the generic `detect` path could recognize `bird`.

**Interpretation:** the custom-model request was reaching the inference system. This was not merely a missing Blue Iris model-name setting. The custom detection path was still returning no objects.

The Blue Iris alert-confirmation confidence setting was initially 50%. Reducing that setting alone cannot resolve a server-side `No objects found` result; model-side confidence, labels, and direct inference still need examination.

## Dataset inspection: the turning point

We inspected actual image thumbnails in the training set:

```text
C:\Program Files\CodeProject\AI\modules\TrainingObjectDetectionYOLOv5\datasets\farm_chickens_v1\images\train
```

The images were heterogeneous: chicken portraits, tiny chicks, illustrations or cartoons, product packaging, cages, and other scenes, some without an obvious chicken. Many looked unlike the fixed, elevated EGGCOOP camera view with pine straw, fencing, shadows, distance, and partial occlusion.

**Leading hypothesis:** a mismatch between the training-image distribution and deployment-camera images (domain shift) contributed to the lack of detections.

**Important limitation:** dataset mismatch has not been proven to be the only cause. Bounding-box annotation quality, class metadata, preprocessing, model confidence thresholds, and direct checkpoint inference still require verification. Training completion does not establish detection quality.

## Next experiment: local photography, not random image gathering

The next dataset should be designed around what the deployed cameras will see. Cell phones make collection practical without first mastering a video-export interface.

### Photography protocol

1. Take approximately **200–300 phone photographs** in several sessions, then select **75–100 meaningfully different images** for an initial labeled dataset. These are starting targets, not requirements.
2. Capture actual chickens on the property: hens, rooster, groups, single birds, different postures and movement.
3. Vary distance, angle, height, lighting, shadows, ground cover, fencing, and partial occlusion.
4. Include wide shots where chickens occupy only a small part of the frame; do not rely on clean close-up portraits.
5. Capture empty scenes and common lookalikes as **negative examples** to evaluate false alarms.
6. Supplement phone photographs with some actual Blue Iris camera frames. The final detector must work on the camera's resolution and viewpoint.
7. Select for diversity, not just quantity. Fifty representative, correctly annotated images can be more useful than a hundred irrelevant images, though performance must be measured rather than assumed.

### Labeling and evaluation

- Draw a correct bounding box around **every visible chicken** in each positive training image; use a consistent `Chicken` class definition.
- Review annotations for missing objects, inaccurate boxes, and accidental non-chicken labels.
- Keep separate training, validation, and test sets. Split by **capture session or video segment** where possible, so near-identical images do not leak across sets.
- Reserve unseen **EGGCOOP camera frames** for final evaluation; phone-only test performance is insufficient.
- Track missed chickens, false positives, confidence, and representative failure examples.
- Preserve v1 unchanged as a baseline and name the new dataset/model `farm_chickens_v2` / `chicken_detector_v2`.
- Before assuming retraining solves everything, test the checkpoint directly and inspect its class names and inference thresholds.

## Proposed iterative workflow

```text
Observe deployment environment
    -> collect representative photographs
    -> select diverse frames
    -> label every target object
    -> review annotation quality
    -> split by session
    -> train a versioned model
    -> evaluate on unseen camera footage
    -> catalog failures
    -> collect targeted new examples
    -> repeat
```

The same workflow can later support object classes such as `Keys`, `Wallet`, `Tool`, and `Egg`. Identifying one *specific* wallet or keyring is a separate, harder task than recognizing a general category. Tracking where an object was last seen requires additional temporal and camera-context logic.

## Lessons learned

1. **Separate infrastructure failures from model-quality failures.** A running inference server does not imply a useful detector.
2. **Verify the whole chain.** Confirm training output, checkpoint placement, request routing, server response, and final alert behavior separately.
3. **Inspect the data, not just the epoch count.** A model can train for 100 epochs on data that poorly represents its intended job.
4. **Train for the camera's world.** Phone photography is a powerful collection tool, but deployment-camera footage remains the test of whether learning transfers.
5. **Treat unfinished work as an experiment with a next step.** The target detection was not achieved today; the repaired environment, diagnosis, and revised plan are reusable progress.

## Next-session checklist

- [ ] Preserve the v1 checkpoint and logs
- [ ] Inspect YOLO annotation text files and trained class metadata
- [ ] Test direct inference with representative EGGCOOP still frames
- [ ] Photograph chickens in varied property conditions
- [ ] Choose a visual bounding-box annotation workflow
- [ ] Build a leakage-resistant v2 train/validation/test split
- [ ] Train v2 and compare against v1 on the same held-out camera frames
- [ ] Record false positives, misses, and improvements

## Safety and publication notes

Do not publish private security footage, precise camera locations, personal addresses, access credentials, or identifiable images of others without appropriate permission. Document procedures and aggregate results publicly; keep sensitive datasets local unless intentionally reviewed for release.

---

**Field conclusion:** The immediate goal—seeing a `Chicken` detection box—remains open. Today's durable result was repairing the inference environment and learning to design training data around the environment where the system must actually perform.
