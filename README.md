# Face-Aware Tripwire Surveillance

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/guna2007/CV_finalised_repo/blob/main/face_aware_tripwire.ipynb)

This computer vision lab uses one recorded camera and a rectangular region. It detects when a tracked person enters or exits the region, compares usable face observations near entry with a reference gallery, and saves a JPEG for each confirmed entry classified as a clear non-match.

| Member | Roll number | Contribution |
|---|---|---|
| Guna | `cs24b2043` | Face detection, enrollment, SFace embeddings and decision rules |
| Susan | `cs23b20xx` | Demo integration, annotated outputs, verification and documentation |
| Vivek | `cs2310xx` | Person detection, ByteTrack, region geometry and crossing state |

## Run the demo

1. Open the notebook in Colab. Keep **CPU** selected, **Enroll demo** enabled and **Upload photo** disabled.
2. Choose **Runtime → Run all**. Setup installs the pinned packages as needed and downloads the source footage and three model files.
3. Play the annotated video and review the event table. To save the alert, set **Alert number** to **1** and run the download cell.

The notebook uses public footage and a built-in reference. For a separate enrollment experiment, enable **Upload photo** and enter a unique alias for a consented, single-face portrait. The gallery is kept in kernel memory.

## Notebook workflow

1. **Setup and inputs.** The notebook checks SHA-256 digests for both cached and downloaded assets. The input is Intel IoT DevKit footage at **768 × 432 pixels and 12 fps**. The region is **(285, 290) to (405, 431)** in source pixels. Frame **996 (83.00 s)** provides the reference for the pseudonym **Person A**.
2. **Detection and tracking.** The pretrained YOLO11n detector keeps person detections with an input size setting of 640 pixels and a confidence floor of 0.15. ByteTrack associates boxes across frames and can use lower-confidence detections to maintain a track. Tracking starts fresh at each clip boundary. Event IDs pair the clip number with the local tracker ID.
3. **Crossing detection.** The bottom-center of the person box approximates the person's position in the image. A track establishes an initial side, then a transition requires three consecutive observations at least five pixels beyond the relevant region edge. A point in the boundary band or an interruption in observations resets the candidate streak. The region touches the bottom of the image, so that edge is excluded from the crossing test. A confirmed entry starts one face decision, and a confirmed exit clears it and rearms the track.
4. **Face comparison.** YuNet detects a single face in the upper 55% of the tracked box. The face must be centered, with confidence of at least 0.80, both dimensions at least 34 pixels and grayscale Laplacian variance of at least 15. SFace aligns the crop using five landmarks and returns a normalized embedding. Samples come from alternate frames, up to eight frames before and ten after entry confirmation. The dot product of two unit embeddings gives their cosine similarity.
5. **Decision and output.** At least three usable samples are required. A match needs a median score of at least **0.48** for the best alias and qualifying votes from at least two thirds of the samples. Each such vote must meet the match threshold and lead any rival alias by at least **0.08**. A clear non-match requires at least two thirds of samples to score at or below **0.30** against every alias. Other entries are **Unverified**. The gap between the cutoffs brackets OpenCV's published SFace reference value of **0.363** on LFW. [OpenCV face recognition tutorial](https://docs.opencv.org/4.x/d0/dd4/tutorial_dnn_face.html)
6. **Review and verification.** The notebook displays the annotated MP4, the crossing table and the JPEG from each clear non-match. The video shows a decision when sampling ends. The table and JPEG retain the time when the crossing was confirmed. The final checks verify the default outcomes, dwell and exit of the non-matching track, JPEG count and dimensions, and video frame count. A separate state test checks re-entry after exit.

Run files appear under `face_aware_tripwire_runtime/`, including `annotated_tripwire.mp4`, `run_summary.json` and `alerts/`. The summary records crossing times, track IDs, decisions, sample counts and processing speed.

## Demonstration and results

Playback concatenates the following half-open source intervals and omits audio. In the derived replay, the code covers the top 41% of each tracked person box after person tracking and before face detection.

| Clip | Source interval (s) | Purpose |
|---|---:|---|
| A | 74.50–81.00 | Enrolled person approaches |
| B | 3.25–19.83 | Another person enters, waits and exits |
| A, derived | 74.50–80.00 | Repeat the entry with the face area covered |

| Playback time (s) | Source time (s) | Track | Decision | Median cosine | Usable faces |
|---:|---:|---|---|---:|---:|
| 2.83 | 77.33 | `1.1` | Match, Person A | 0.571 | 7 |
| 8.92 | 5.67 | `2.1` | Non-match | 0.011 | 5 |
| 22.17 | 18.92 | `2.1` | Exit, rearmed | — | — |
| 25.92 | 77.33 | `3.1` | Unverified, derived | — | 0 |

A fresh local CPU run processed **343 frames in 37.4 s (9.2 fps)**, including video encoding. The non-matching track stayed inside for **159 observed frames** and generated **one JPEG** before its confirmed exit.

![The saved alert for the non-matching track, with the person box and tripwire overlaid](demo_alert_preview.jpg)

## Limitations and future work

Enrollment and evaluation use different time windows from the same camera recording. The results describe the demo, and the cutoffs have yet to be calibrated on independent footage. The box bottom can move with cropping and occlusion. Track loss or ID switches can interrupt the crossing sequence.

Future evaluation should use separate recordings and measure missed crossings, false alerts and the unverified rate across lighting, viewing angles and crowded scenes. The match cutoffs could then be calibrated on held-out examples, with ground-plane calibration used to test whether the box-bottom proxy is reliable.

## Submission files

The classroom deliverables are the [notebook](face_aware_tripwire.ipynb) and [report PDF](face_aware_tripwire_report.pdf). The [LaTeX source](face_aware_tripwire_report.tex) can be compiled from the repository root with `tectonic face_aware_tripwire_report.tex`.

## Acknowledgments

- Intel IoT DevKit provided the walking-and-pause sample. The project uses excerpts, a labeled occlusion replay, and annotated frames under CC BY 4.0. [Source video](https://github.com/intel-iot-devkit/sample-videos/blob/master/face-demographics-walking-and-pause.mp4) · [Direct MP4](https://raw.githubusercontent.com/intel-iot-devkit/sample-videos/master/face-demographics-walking-and-pause.mp4) · [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- Ultralytics provided YOLO11n and the tracking interface. ByteTrack follows the method of Zhang et al. [YOLO11 documentation](https://docs.ultralytics.com/models/yolo11/) · [ByteTrack paper](https://arxiv.org/abs/2110.06864)
- OpenCV Zoo provided the pretrained YuNet and SFace models used for face detection, alignment and embedding comparison. [YuNet](https://github.com/opencv/opencv_zoo/tree/main/models/face_detection_yunet) · [SFace](https://github.com/opencv/opencv_zoo/tree/main/models/face_recognition_sface)
