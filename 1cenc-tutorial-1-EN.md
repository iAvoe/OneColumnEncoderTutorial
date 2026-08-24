# 1cenc Basic Workflow Tutorial

This document is aimed at beginners, explaining the complete workflow from downloading to starting video encoding. The instructions are written in great detail and omit performance-related usage tips.

## How to Report Issues

You can report issues via [GitHub Issues](https://github.com/iAvoe/OneColumnEncoder/issues) or the [NazoRip project comments section](https://nazorip.site/archives/1593/). Before reporting, please confirm that the issue is indeed caused by 1cenc. It is best to include screenshots, runtime log copies, ffprobe logs, or other auxiliary information to aid troubleshooting.

## Download and "Installation"

Running 1cenc requires at least Windows 10. Version 1809 / 21H2 (LTSC) or higher is recommended; the minimum is 1607.
- 64-bit OS: download `1cenc-win-x64.7z`
- 32-bit OS: download `1cenc-win-x86.7z`
- The 32-bit version is not recommended unless you are willing to tinker

> Most operating systems are 64-bit, and some bundled tools do not support 32-bit OS. Although it can run, functionality is limited.

### Download and "Installation" — Extracted File List

<img src="./img-cn/1-Extract-Archive.png" alt="Extracted file list" width=300 />

- `1cenc` folder: bundled tools, queue list data, encoding runtime logs, settings, and other data
- `OneColumnEncoder.exe`: main program
- `OneColumnEncoder.pdb`: debug information (upload when reporting issues)
- `x64-upstreams-encoders.7z`: archive containing **video processing tools** such as ffmpeg, ffprobe, x264, etc.
    - These tools are too large (triggering GitHub limits) and need to be extracted separately
    - For 32-bit OS, use `x86-upstreams-encoders.7z`

**Bundled Video Processing Tools**
- `ffmpeg.exe`: video decoding, ffmpeg filters, video frame encoding preview, repartition boundary preview, auto muxing
- `ffprobe.exe`: video source analysis (generates partial encoder parameters, checks format compatibility and corruption)
- `x264[...].exe`: AVC video encoder
- `x265[...].exe`: HEVC video encoder
- `SvtAv1EncApp[...].exe`: AV1 video encoder

> 1cenc with all tools exceeds 350MB, mainly due to FFmpeg and the built-in .NET 9.0 dependency. For a lightweight installation, you can ask AI to help you compile from source.
> For detailed tool descriptions, see the video encoding tool download tutorial: [NazoRip](https://nazorip.site/archives/1482); [GitHub](https://github.com/iAvoe/encoding-tools-download-tutorial)

**Other Tools**
- `x64-AVS-VS-plugins`: AviSynth and VapourSynth video filters (requires AviSynth and VapourSynth to be installed and imported)
- `x64-CloudinarySSIMULACRA2.1|GoogleButteraugli`: bundled image quality scoring tools for previewing the impact of encoding parameter settings

#### Video Processing Tool Placement

The .exe tools from `x86|x64-upstreams-encoders.7z` can be placed anywhere. However, 1cenc has an auto-import feature that searches the following locations, making it smoother:

>     .
>     ├── ①
>     ├── OneColumnEncoder.exe
>     ├── OneColumnEncoder.pdb
>     ├── x64-upstreams-encoders
>     │    └── ②
>     └── 1cenc
>          ├── ③
>          ├── x64-upstreams-encoders
>          │   └── ④
>          ├── x64-AVS-VS-plugins
>          │   └── ...
>          ├── x64-CloudinarySSIMULACRA2.1
>          │   └── ...
>          └── x64-GoogleButteraugli
>             └── ...
>

---

## First Launch

Double-click `1cenc.exe` to open. On first launch, you will be asked whether to auto-import. Click Confirm to import:

<img src="./img-cn/2-Auto-Import.png" alt="Auto-import prompt" width=400 />

If you clicked wrong or missed it, you can manually import, or check "Run auto-import on next open" in the settings page.

<img src="./img-cn/3-Rerun-Auto-Import.png" alt="Re-run auto-import" width=1000 />

> If you want to open 1cenc.exe from elsewhere, you can create a shortcut or Start Menu entry.

Since VapourSynth and SVFI are system-installed software, 1cenc's auto-import search will attempt to find and import them. The image below shows auto-import importing 4 extracted tools along with 2 installed tools simultaneously.

<img src="./img-cn/4-Import-Result-And-Click-Select.png" alt="Import result" width=550 />

### First Launch — Main Interface Overview

The main interface of 1cenc consists primarily of **titles**, **checklist**, **ItmeCards**, and **buttons**. Tool cards may select or open windows when clicked, depending on their function.

#### Tool ItemCards

The main building blocks of the interface. Composed of black borders, black text, and a light gray background. They support expand/collapse, hover highlight, selection, and error state display. Once familiar, collapsed state makes operations more convenient.
- If a tool shows an abnormal state (red border), re-import the tool

<img src="./img-cn/5-Tool-ItemCard.png" alt="Tool card" width=1000 />

Since paths can be very long, tool cards provide two ways to display the full path:
- Tooltip that appears after hovering the cursor for a while
- In the hint text below after clicking to select the tool card

<img src="./img-cn/6-Hover-Hint.png" alt="Hover hint" width=350 />

#### Titles and Buttons

**Titles** indicate the operation step sequence and instructions, and include the expand/collapse button on the right.

<img src="./img-cn/7-Main-Page-Title-Buttons.png" alt="Titles and buttons" width=550 />

**Buttons** perform operations or open windows, but are disabled when prerequisites are not met.

> Note: "3. Select dependency file" is for AvsToPipeMod's AviSynth.dll dependency. Ignore if not needed.

#### Checklist

Various check items, including whether tools are selected, conditions for unlocking the "Start Encoding" button, and potential issues with the video source.
- The checklist also supports collapse state
- Click list items to view details

<img src="./img-cn/8-Checklist.png" alt="Checklist" width=700 />

**4 Checklist Item States:**
- Not checked — black text, closed-eye icon: the check item is not relevant to the current workflow and is skipped
- Passed — green text, checkmark icon: normal or conditions met
- Warning — yellow text, double exclamation icon: may cause abnormal encoding results, but does not disable the "Start Encoding" button
- Error — red text, cross icon: base conditions not met or encoding cannot start; disables the "Start Encoding" button

#### Video Source ItemCard and Encoding Config ItemCard

These cards have different functions, so their appearance differs from the default white-background black-text style. Video source cards typically open a file selection window when clicked; encoding config cards typically open a configuration window.

<img src="./img-cn/9-Source-And-Config-ItemCards.png" alt="Video source and config cards" width=1300 />

**Video Source Card Functions:**
- **Video Source**: select a single video source
- **Video Queue**: select multiple video sources or Blu-ray playlist (PLAYLIST) folders
- **Video Source Concat**: select multiple video sources, output a concatenated video
- **Video Source Repart**: select a video source folder (multiple sources) or Blu-ray playlist (PLAYLIST) folder, specify split points, output multiple split or merged-then-split videos (can be simply understood as splitting)

---

### First Launch — Settings Overview

The settings page is the slowest window to open in 1cenc — due to font settings loading.

<img src="./img-cn/10-Settings.png" alt="Settings" width=500 />

#### Defaults and Recommended Values

**File Overwrite Confirmation — Cooldown MB Divisor**
- When a file to be overwritten is detected, calculates a cooldown duration based on the file size
- Recommended to keep default to prevent large files from being easily overwritten; minimum is 1

**Encoding Log TXT**
- Saves runtime logs from the selected upstream program (ffmpeg, VapourSynth, ...) and downstream program (x264, x265, SVT-AV1) to files during encoding tasks
- Recommended to enable for troubleshooting; the default file count limit is based on common anime episode counts (i.e., queue length)

**Auto Mux Switch**
- Muxes the encoded video stream into `.mkv` format
- Since x265 does not support auto mux (only exports `.hevc`) and causes loss of frame rate and other metadata, it is recommended to enable this for x265
- x264 muxes to `.mp4`, SVT-AV1 muxes to lightweight `.ivf`; these formats have audio compatibility limitations, so disabling auto mux disables the audio processing mechanism

**Auto Mux — Audio Processing**
- Due to `.mkv`'s good compatibility and support by Adobe editing software, **single file** and **queue mode** recommend copying the audio stream directly to the muxed output
- **Concat mode** and **repart mode** involve audio stream editing; some audio streams may not be supported or have poor support, so the default is set to re-encode
- Although re-encoding supports Opus format, the default is intentionally set to AAC for compatibility

---

## First Encoding

1. Select FFmpeg
2. Select x264
3. Select a video source
    - Each time a new video source is selected, 1cenc automatically runs video analysis; otherwise click the "Run Video Source Analysis" button
    - FFProbe is automatically selected, so no manual selection is needed
4. Click "Start Encoding"
    - You can also click "Output Filename and Path" to adjust the output location

<img src="./img-cn/11-Start-First-Encode.png" alt="Start first encoding" width=600 />

After clicking "Start Encoding", 1cenc will first pop up an encoding and muxing parameter dialog, which supports right-click to copy text.
- The image shows a two-stage structure: **Encoding** → **Muxing**
    - i.e., "**Video Encoding** → **Audio Copy or Encode + Muxing**"
- In **concat mode** and **repart mode**, with auto mux audio processing set to "Re-encode to AAC/Opus", the encoding command becomes a three-stage structure
    - i.e., "**Video Encoding** → **Audio Encoding** → **Muxing**"
- If you see `--master-display` or `G(xxx,yyy),B(zzz,aaa),R...` parameters, it means the source video's HDR metadata appendix/sidedata has been recognized and converted to encoding parameters

<img src="./img-cn/12-Inspect-Encoding-Param.png" alt="Inspect parameters" width=400 />

After clicking confirm again, the main interface will hide to free memory and reduce background interference, and the **Encoding Monitor** window will open:

<img src="./img-cn/13-Encoding-Monitor.png" alt="Encoding monitor" width=800 />

### First Encoding — Encoding Monitor Overview

**Progress Bar**
- If the video metadata contains total frame count, a progress bar is shown; otherwise a barber pole animation is displayed (stops when complete)

**Memory Usage**
- Check if the current computer's memory is a bottleneck
- Check if some programs should be closed to free more memory
- If the computer has multiple NUMA nodes, you can theoretically open several encoding processes

**Process Logs**
- Left side: upstream program (ffmpeg, VapourSynth, ...) logs
- Right side: downstream program / encoder logs
- The divider in the middle can be dragged to resize the two windows
- Three buttons at the bottom can copy the current log text or cycle through font sizes

**Time Calculator**
- View elapsed time, estimated completion time, and other time status
- If the video metadata does not contain total frame count, estimated time information will not be updated

**Rich Text Parsing**
- Recognizes font, color, and other command symbols (SVT-AV1 writes these)

**Save Log**
- Saves the process logs from this encoding session as a `.txt` file in the `1cenc` directory

**Browse and Process Operations**
- **Interrupt Upstream / Downstream Program**
    - The upstream program handles decoding, so it completes more frames than the downstream program (encoder)
    - The choice depends on whether you want to encode a few more frames (interrupt upstream) or emergency stop (interrupt downstream)
- **Close**
    - Unlocked after both upstream and downstream programs have been interrupted / exited

After encoding completes, 1cenc will start the auto mux operation, producing a second segment of upstream program log:

<img src="./img-cn/14-Encoding-Monitor-On-Fin.png" alt="Encoding monitor completion state" width=800 />

---

## Encoding Again (Full Workflow)

***Follow the configuration / click sequence shown in the image below to re-run this encoding:***

<img src="./img-cn/15-Start-First-Encode-Again.png" alt="Start encoding again" width=600 />

### Encoding Again ④ — Filter Editor Overview

Some command lines / script lines in the filter editor are generated based on analysis data, so its buttons are only unlocked after "Run Video Source Analysis" is completed.
- The filter editor automatically selects the tab (the upstream program here is ffmpeg)

<img src="./img-cn/16-Filter-Scribe-Modal.png" alt="Filter editor" width=500 />

Most filter parameters in the image show "N/A" — because the video source does not need these corrections. Conversely, if the video source has variable frame rate, non-square pixels (SAR correction needed), etc., the corresponding filter command lines will be generated.

***Follow the image below to generate a scale-down command using the resolution scaling controls, then paste it into the filter parameter box at the top and click Confirm.***

<img src="./img-cn/17-Filter-Scribe-Modal-Apply.png" alt="Filter editor apply" width=500 />


When clicking Confirm to save, an "Overwrite ffprobe JSON" confirmation dialog will appear. This is because the filter modifies the video source resolution, changing the basis for encoding parameter construction, requiring manual recalibration.

***Click "Update ffprobe JSON".***

<img src="./img-cn/18-Source-Reviser.png" alt="Source reviser" width=350 />

> Note: VapourSynth filter editor supports single-frame filter effect preview

### Encoding Again ⑤ — Copy Raw JSON

When 1cenc makes strange checklist judgments or triggers unexpected warnings, you can use this to troubleshoot.

*** (Optional) Open your preferred text editor, paste it in, and inspect it***

### Encoding Again ⑥ — Output Filename and Path

Used to preview how filenames appear under various device / UI width constraints, while avoiding encoding failures caused by filenames. Useful for managing large numbers of files or when publishing.

Set a new filename and specify a new output location here.

<img src="./img-cn/19-Output-Filename-And-Path.png" alt="Output settings" width=350 />

### Encoding Again ⑦ — Parallelism Scheduling

Configure which NUMA node the upstream program and encoder run on. On computers with more than one CPU–memory node, due to BIOS settings or using TR (Threadripper) / EPYC (霄龙) processors, there is a chance that multiple node options will appear.

**Limit Upstream / Downstream Program Threads to Physical Core Count**
- The encoder (downstream program) is a compute-heavy, cache-light program (though cache usage depends on video resolution)
    - Limiting to physical core count achieves an "HPC disable hyperthreading strategy" speedup, while not wasting the low-overhead benefit of hyperthreading for compute-light, cache-heavy programs
- This is what makes 1cenc faster and more stable than general encoding software

**Pipe Buffer**
- Increases cache usage to avoid performance bottlenecks — theoretically. In practice, no significant change was observed, and there is no negative impact, so it can be ignored.

***You can keep the defaults or check all items here, then confirm***

<img src="./img-cn/20-Parallelism-Settings.png" alt="Parallelism settings" width=500 />

### Encoding Again ⑧ — Encoding Parameters (and Preview)

Configure and preview encoder / downstream program parameters here:
1. Select an appropriate CRF value based on the **CRF Scale** hint at the bottom of the CRF mode tab
2. In **Custom Parameters — Base Parameters**, select the preset most relevant to your video source
3. In **Custom Parameters — Keyframe Interval Seconds**, select a value matching your playback scenario based on the hint below
4. (Ignore other settings)
5. At the bottom of the right preview window, click **Preview**, then wait a while
    1. Wait for the current frame encoding to complete
    2. (Optional) Wait for the SSIMULACRA and Butteraugli quality scores in blue text at the bottom right to finish
6. Drag the divider between SOURCE and ENCODE for precise comparison
7. (Optional) Drag the **Frame Position** progress bar to preview again
8. (Optional) Change the **encoder** in the top-left corner of the preview window and preview again

***Adjust CRF mode parameter values, select base parameters, and keyframe interval according to the instructions above, then click Confirm at the bottom of the left panel***

<img src="./img-cn/21-Encode-Settings.png" alt="Encoding settings" width=1200 />

### Encoding Again — Clip Sampling

Specify a segment of the video to encode. This function trims excess intro/outro and also serves sharing or sample testing needs (such as encoding group review workflows) to demonstrate **normal subjective quality**.

***Try using the duration slider to expand the segment to 60 seconds, drag the yellow portion on the timeline to the beginning, and click "Start Sampling"***

<img src="./img-cn/22-Clip-Sampler.png" alt="Clip sampler" width=450 />

After confirming "Start Sampling", a debug window will pop up, but this time no muxing command will appear.

***Click Confirm to start encoding***

<img src="./img-cn/23-Inspect-Sampling-Param.png" alt="Inspect sampling parameters" width=400 />

---

## Basic Workflow Tutorial Complete

### Unmentioned Content (Advanced Use Cases)

- Blu-ray chapter import
- Video source queue mode
- Video source concat mode
- Video source repart mode (basic splitting, meat platter handling, merge platter handling)
- Basic AviSynth and VapourSynth filter processing with queue, concat, and repart mode variations
- Multi-NUMA-node multi-instance strategies (title PID numbers, source folder splitting, queue splitting)
