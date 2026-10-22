# FireRobot-Instruct dataset card

## Purpose and release status

FireRobot-Instruct supports image-conditioned scene description and visible-hazard explanation for fire-scene monitoring and reconnaissance. The complete dataset will be openly released at [HTZ2025/FireSense](https://github.com/HTZ2025/FireSense) after publication of the paper. This card describes the corpus used in the study.

## Source and selection

The corpus uses 7,072 images selected from D-Fire: 5,822 have at least one source fire annotation, and 1,250 are clear-negative controls with no fire or smoke labels and zero detection boxes. The selected positive pool includes fire-only and fire-with-smoke images. Smoke-only images are outside the current selection.

Fire-positive selection is based on the upstream detection labels. The new hazard reference is determined from the evidence and context visible in the image, so source selection and hazard status are recorded separately.

## Annotation procedure

One Qwen3.8-Flash request through OpenRouter drafts four fields per selected fire-positive image: `scene_description`, `hazard_status`, `hazard_explanation`, and `visible_evidence`. Generation uses temperature zero with additional reasoning disabled. Human review and correction compare the draft with the image before inclusion.

Scene references describe visible objects, conditions, and relations in one to three concise English sentences. Hazard references connect a concern to visible evidence or express apparent absence or insufficient evidence. The instructions exclude claims about hidden fire, heat intensity, smoke toxicity, structural stability, unseen victims, and future spread. They also exclude robot-control commands.

Clear-negative records use the source-label selection rule and a standardized hazard-only answer. Their provenance is distinguished from the reviewed positive-image records.

## Record organization

The prepared JSONL records associate an image with task-conditioned conversational messages. The roles and metadata have the following uses:

| Component | Use |
|---|---|
| Image reference and source identity | Locate the original image and preserve provenance |
| Task identifier | Distinguish scene description from hazard explanation |
| System/user messages | Specify report scope and output style |
| Assistant message | Supervised answer text |
| Hazard status | Reference state for auditing and evaluation |
| Visible-evidence notes | Audit the basis of a reviewed hazard annotation |
| Partition and provenance information | Keep related task records together and distinguish annotation sources |

Each selected fire-positive image supplies two task records; each clear-negative image supplies one hazard record. Hazard metadata does not introduce a separate classification head: models learn through assistant-response text supervision.

## Partitions and counts

| Split | Positive images | Negative images | Scene records | Hazard records | All records |
|---|---:|---:|---:|---:|---:|
| Train | 4,125 | 875 | 4,125 | 5,000 | 9,125 |
| Validation | 582 | 125 | 582 | 707 | 1,289 |
| Test | 1,115 | 250 | 1,115 | 1,365 | 2,480 |
| Total | 5,822 | 1,250 | 5,822 | 7,072 | 12,894 |

Original D-Fire test-side selections remain held out. Validation is drawn from the source training side with seed 42. All records associated with one image remain together, and input resizing occurs within the assigned partitions. Exact-content SHA256 checks found no overlap across the prepared corpus partitions or against the auxiliary DetectiumFire pool.

## Hazard labels

- **Present:** visible evidence supports an identifiable fire-related concern.
- **None:** an adequately observable view has no apparent fire-related concern.
- **Insufficient:** limited visibility prevents a meaningful assessment.

Uncertainty about an observed concern's cause, extent, or severity does not remove a `present` judgment. When a distant bright region cannot be distinguished from ordinary illumination, the evidence may instead support `insufficient`.

The three states have 5,391, 1,393, and 288 records, respectively. The insufficient split is 208 training, 33 validation, and 47 test records. Of the 283 test `none` records, 250 are clear-negative controls and 33 come from the fire-positive source pool.

## Evaluation supported by the corpus

The paper measures lexical and semantic agreement with reviewed reference answers and extracts hazard states from generated reports. It evaluates the fire-positive and clear-negative hazard subsets separately, as well as their pooled results. Selected image-grounded comparisons inspect the support for individual response statements.

The state extractor uses `unresolved` for responses whose expressed state cannot be determined. Such responses stay in the overall agreement denominator. This outcome is separate from the three dataset reference states.

## Reuse and attribution

The public release will include annotation documentation, the fixed split manifests, and preparation/evaluation guidance. Source-image access follows the [D-Fire repository](https://github.com/gaia-solutions-on-demand/DFireDataset). Upstream images and newly created annotations/code will have their terms documented separately.

DetectiumFire contributes 6,031 training captions and 1,510 validation captions to separate experiments. These captions are not part of FireRobot-Instruct's inventory.
