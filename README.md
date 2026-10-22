# FireSense

### FireRobot-Instruct: visual instruction data for fire-scene reporting

FireSense studies how a compact vision-language model can generate useful fire-scene reports on resource-constrained edge devices. Its task dataset, **FireRobot-Instruct**, connects images to two complementary outputs: **factual scene descriptions** and **explanations of visible fire-related concerns**.

**Release status:** This repository is the project and dataset homepage. **The complete FireRobot-Instruct dataset will be openly released here after publication of the paper**, together with its annotation documentation and image-level partition manifests. The current homepage introduces the dataset and its construction procedure.

[中文介绍](README_zh-CN.md) · [Dataset card](DATASET_CARD.md) · [Project repository](https://github.com/HTZ2025/FireSense)

## Dataset at a glance

| Item | Count / description |
|---|---|
| Source images | 7,072 selected D-Fire images |
| Instruction records | 12,894 |
| Scene-description records | 5,822 |
| Hazard-explanation records | 7,072 |
| Tasks per fire-positive source image | One scene record and one hazard record |
| Tasks per clear-negative source image | One hazard record |
| Report language | English |
| Hazard reference states | `present`, `none`, `insufficient` |
| Partition unit | Source image; all records from one image stay together |

FireRobot-Instruct adds reporting annotations to existing D-Fire images. D-Fire's fire/smoke detection labels guide source selection; the new references describe scene context and the evidence supporting a hazard assessment. The source fire label and the final hazard-reference state have different roles: an image selected using a fire annotation can still receive an `insufficient` reference when its visible detail does not support a meaningful assessment.

## What does one image teach the model?

**Scene description** covers visible entities, conditions, and spatial relations in one to three concise sentences. **Hazard explanation** states a fire-related concern supported by the image, or expresses apparent absence or insufficient evidence in a concise response. Each task has its own instruction and answer.

![Examples of FireRobot-Instruct training records](assets/dataset_examples.png)

The examples above come from the training partition. Blue fields show scene answers, red fields show hazard answers, and gray fields show audit metadata. The supervised target is the assistant's answer text; hazard states and evidence notes are retained for auditing and evaluation.

## How FireRobot-Instruct was constructed

![FireRobot-Instruct construction procedure](assets/dataset_construction.png)

1. **Select source images.** We select 5,822 D-Fire images with at least one fire annotation. A further 1,250 clear-negative images have neither fire nor smoke labels and contain zero detection boxes. Source image identities and original train/test membership are retained.
2. **Draft the positive-image annotations.** One Qwen3.8-Flash request per selected fire-positive image produces a structured draft containing a scene description, hazard status, hazard explanation, and one to four visible-evidence statements. The prompts require descriptions grounded in the image and uncertainty when visibility prevents assessment.
3. **Review against the image.** Human review and correction produce the final positive-image annotations. Each reviewed image yields two independent instruction records, one for each reporting task.
4. **Add clear-negative controls.** The clear-negative images receive a deterministic hazard-only target: “No obvious fire-related concern is visible in the observed view.” These records use the source labels, with their provenance distinguished from the reviewed positive-image annotations.
5. **Validate and partition.** Structural checks validate record identifiers and task pairing. Original test-side selections remain held out; validation images are drawn from the source training side using seed 42. Partitioning precedes model-input resizing, and all task records for an image remain in the same partition. SHA256 checks audit exact-content overlap within the corpus and against the auxiliary caption pool.

## Partitions

| Partition | Images | Scene records | Hazard records | Total records |
|---|---:|---:|---:|---:|
| Training | 5,000 | 4,125 | 5,000 | 9,125 |
| Validation | 707 | 582 | 707 | 1,289 |
| Test | 1,365 | 1,115 | 1,365 | 2,480 |
| **Total** | **7,072** | **5,822** | **7,072** | **12,894** |

The test set contains 1,115 fire-positive source images and 250 clear-negative controls. These two subsets support separate evaluation of varied fire-scene reports and standardized negative responses.

## Hazard reference states

| State | Definition |
|---|---|
| `present` | Visible evidence supports an identifiable fire-related concern. |
| `none` | No apparent fire-related concern is found in an adequately observable view. |
| `insufficient` | Darkness, distance, blur, occlusion, or similar limits prevent a meaningful assessment. |

An identifiable concern remains `present` when its cause, extent, or severity is unknown. A view too limited to assess is `insufficient`, even if no flame is clearly visible. Thus, `insufficient` describes assessability rather than a lower hazard-severity level.

| Partition | Present | None | Insufficient |
|---|---:|---:|---:|
| Training | 3,816 | 976 | 208 |
| Validation | 540 | 134 | 33 |
| Test | 1,035 | 283 | 47 |
| **Total** | **5,391** | **1,393** | **288** |

Models generate free-text reports. During evaluation, a parser extracts the state expressed in the response. An `unresolved` extraction means that the response does not support a determinable state; it is an evaluation outcome, not an additional training label.

## FireSense experiments

The accompanying study uses Qwen3.5-2B to examine task adaptation, prior visual-language merger adaptation, and auxiliary caption replay. FireSense-S provides a common checkpoint for comparing six GGUF precisions and generation throughput on an 8 GB Jetson Orin Nano Super. Firefighting robots are a representative application; the image-and-instruction interface is also relevant to camera-equipped inspection robots, drones, and fixed monitoring systems.

The auxiliary DetectiumFire captions are a separate supervision source and are **not included in the 12,894 FireRobot-Instruct records**.

## Planned public release

After paper publication, this repository will provide:

- Complete FireRobot-Instruct instruction annotations and accompanying audit metadata.
- Image identities and fixed training, validation, and test manifests.
- Annotation instructions, field documentation, and data preparation guidance.
- Training and evaluation code, including the hazard-state extraction protocol.

Source-image access and attribution will follow the upstream dataset's terms. The release will document the terms for the newly created annotations and code separately.

## Source resources

- [D-Fire](https://github.com/gaia-solutions-on-demand/DFireDataset): source imagery and fire/smoke detection annotations.
- [DetectiumFire](https://github.com/ZixuanLiu4869/DetectiumFire): separate auxiliary image-caption supervision used in the training-route experiments.

We acknowledge the creators of these resources. FireRobot-Instruct's contribution is the task-specific reporting supervision, record organization, and evaluation partitions built on the selected source images.
