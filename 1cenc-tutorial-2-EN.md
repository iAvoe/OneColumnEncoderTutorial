# 1cenc Advanced Workflow Tutorial

This document explains Blu-ray format details, features for advanced encoding workflows, usage, and examples. Unlike the basic tutorial, it does not show every step in full.

**How to Report Issues**
- You can report issues via [GitHub Issues](https://github.com/iAvoe/OneColumnEncoder/issues) or the [NazoRip comment section](https://nazorip.site/archives/1593/). Before reporting, please verify the issue is caused by 1cenc. It is best to include screenshots, runtime log copies, ffprobe logs, or other auxiliary information to aid troubleshooting.


## Fork (Self-clone)

In simple terms, a fork clones the software's complete current state to create two identical running instances with no clear primary or secondary instance. 1cenc may be among the first encoding programs to support this feature, but the concept is not new: it also appears in Git (version control), VMFork (virtual-machine management), and some PVE online games.

**Instance**
- In computer science, an instance is a concrete realization created from a model or blueprint. Instances based on the same model usually share a data structure, but the values stored in each instance are independent. —Wikipedia

<img src="./img2-all/1-Fork-Usage.png" alt="Fork" width=800 />

> Fork eliminates repeated launches and repeated configuration.
> Distinguish running instances by checking the PID (process ID) shown on the taskbar or in the program title bar.

### Fork — Basic Workflow
1. **Complete the shared configuration**
   * Select the upstream tool (filtering) and downstream tool (encoder)
   * Import the source video
   * Define common video filters and encode settings as needed
2. **Create enough instances for the required targets**
   * 2: Lossless compression | Lossy compression
   * 2: Burn-in subtitles | Embedded subtitles
   * 4: x264-1080p | x264-720p | x265-1080p | x265-720p
   * N: Try different encoding parameter values to find the best result, or simply for testing
3. **Differentiate each instance's configuration**
   * Individual video filters, encode settings, etc.
   * Output Settings (Path and Filename): Use different output filenames

### Fork — NUMA Node Distribution (Optional)
1. **Complete the shared configuration**
   * Select the upstream tool (filtering) and downstream tool (encoder)
   * Import sources in Queue Mode or Repart Mode (described below)
   * Define common video filters and encode settings as needed
2. **Create one instance for each available NUMA node**
3. **Differentiate each instance's workload**
   * Selectively delete duplicate tasks so the work is distributed evenly across N nodes
     * Queue Mode → Open Queue Editor → Delete button
     * Repart Mode → Open Repart Editor → Find the Output Queue → Press Delete
     * “Sort by file size” can further improve the distribution
   * Parallelism Control: Assign each instance to an exclusive compute node

> Queue Mode and Repart Mode do not support manually specifying filenames, but removing duplicate tasks also prevents overwriting.
> This produces an effect similar to x265's thread-pool scheduling in x264...

### Fork — Other Uses

**Look back during encoding**
- The main window is hidden while encoding. Fork lets you review the settings in another instance.

**Dense scheduling**
- Limit every instance to 1–4 cores (or one CCD on an AMD ZEN CPU).
- Individual tasks become slower, but improved cache scheduling can improve encoder redundancy (lossless compression) and the efficiency of large batches.
- High-resolution video may exceed available memory.

---

## Blu-ray Format

**Note: Content may originate from illegal sources. Before proceeding, please consult to local and international copyright regulations as well as regional laws regarding the legality of the content itself (such as the US DMCA or EU copyright directives), and opt for resources that are no longer under copyright protection.**

### File Structure and Encoding Mode Selection

Since many readers are likely encountering Blu-ray discs for the first time, this section is deliberately detailed and contains many inferences and conclusions. Skip it if you already know the material.
- The following structure was listed by `Get-PSTree`.

**Basic structure:**
```
BDMV/
├─ index.bdmv        [entry] title index/navigation entry
├─ MovieObject.bdmv  [navigation] command objects/playback control
├─ PLAYLIST/         [playlists] *.mpls, determines playback order
├─ CLIPINF/          [clip information] *.clpi, corresponds to m2ts numbers
├─ STREAM/           [actual streams] *.m2ts ★content type can be inferred from size
├─ META/DL/          [metadata/trademarks] .xml/.jpg
├─ BACKUP/           [backup] duplicates key files and may be omitted here
└─ CERTIFICATE/      [certificate] id.bdmv, may be omitted here
```

#### TV Series Episode File Structure

Using a two-disc multi-episode TV anime as an example; some folders remain collapsed to highlight useful content.
```
        [BDMV][220705][One Piece：Season Eleven Voyage Nine (ep.733-746)][USA][Puto]
           ├── OnePiece_S11_V9_D1
           │   └── BDMV
 456.00  B │       ├── index.bdmv       [entry/navigation]
 113.23 KB │       ├── MovieObject.bdmv [navigation commands]
  93.82 KB │       ├── AUXDATA          [auxiliary data: fonts/sound/menu]
 113.67 KB │       ├── BACKUP           [backup, omitted]
  96.05 KB │       ├── CLIPINF          [clip information] 00002~00064.clpi, corresponding to m2ts
           │       ├── META             [metadata]
  54.35 KB │       ├── PLAYLIST         [playlists] 00001~00047.mpls, not fully corresponding to m2ts
  45.55 GB │       └── STREAM
  15.40 MB │           ├── 00002.m2ts
   2.04 MB │           ├── 00003.m2ts
  17.67 MB │           ├── 00004.m2ts
   6.51 MB │           ├── 00005.m2ts
   5.36 MB │           ├── 00006.m2ts
 102.00 KB │           ├── 00007.m2ts
   1.80 MB │           ├── 00010.m2ts
 276.00 KB │           ├── 00011.m2ts
  18.00 KB │           ├── 00012.m2ts
  18.00 KB │           ├── 00013.m2ts
   6.39 GB │           ├── 00014.m2ts  [probably main episode] likely episode 733, about 24 minutes
  54.31 MB │           ├── 00015.m2ts  [possibly preview image]
   6.39 GB │           ├── 00016.m2ts  [probably main episode] likely episode 734
  54.31 MB │           ├── 00017.m2ts  [possibly preview image]
   6.39 GB │           ├── 00018.m2ts  [probably main episode] likely episode 735
  54.35 MB │           ├── 00019.m2ts  [possibly preview image]
   6.39 GB │           ├── 00020.m2ts  [probably main episode] likely episode 736
  54.33 MB │           ├── 00021.m2ts  [possibly preview image]
   6.39 GB │           ├── 00022.m2ts  [probably main episode] likely episode 737
  54.30 MB │           ├── 00023.m2ts  [possibly preview image]
   6.39 GB │           ├── 00024.m2ts  [probably main episode] likely episode 738
   3.58 MB │           ├── 00025.m2ts
   5.36 MB │           ├── 00026.m2ts
   8.14 MB │           ├── ...(19 files of 8.14 MB omitted: 00027~00045; 7 files of 5.36 MB: 00047~00053)
   8.14 MB │           ├── 00054.m2ts
  66.00 KB │           ├── 00055.m2ts
 216.00 KB │           ├── 00056.m2ts
  14.81 MB │           ├── 00057.m2ts
  54.33 MB │           ├── 00058.m2ts
   6.39 GB │           ├── 00059.m2ts  [probably main episode] likely episode 739
  54.28 MB │           ├── 00060.m2ts
 168.33 MB │           ├── 00063.m2ts
   1.80 MB │           └── 00064.m2ts
           └── OnePiece_S11_V9_D2
                └── ...(same structure; too many files, omitted here)
```

> Pitfall: without careful inspection, `00059.m2ts` may be missed, resulting in one fewer episode than everyone else.

**Playback observations:**
- The main-episode guesses were correct
- `00025~00055.m2ts` are all black screens
- The items previously identified as “preview images” are actually credits for English voice actors and the company
  - Given the age of the production of main-episode, adding separate credits prevent unnecessary re-encoding of source

**Conclusion: the episodes are already separated correctly, but anime fans place importance on voice-actor credits, so the extra credits should be joined to the end of each main episode (Repart Mode required).**

---

#### Concert File Structure

Using a 4-disc concert set as an example; some folders remain collapsed to highlight the relevant content
```
        [BDMV][251008] 結束バンド TOUR “We will B”
           ├── Kessoku Band TOUR “We will B” DISC1
 554.00  B │   ├── BDMV
 120.00  B │   │   ├── index.bdmv
 434.00  B │   │   ├── MovieObject.bdmv
 554.00  B │   │   ├── BACKUP
  64.55 KB │   │   ├── CLIPINF
           │   │   ├── META
   1.64 KB │   │   ├── PLAYLIST
  43.78 GB │   │   └── STREAM
  42.38 GB │   │       ├── 00000.m2ts
  37.59 MB │   │       ├── 00001.m2ts
   7.81 MB │   │       ├── 00002.m2ts
  21.13 MB │   │       ├── 00003.m2ts
   1.33 GB │   │       ├── 00004.m2ts
 192.00 KB │   │       ├── 00005.m2ts
 264.00 KB │   │       └── 00006.m2ts
 104.00  B │   └── CERTIFICATE
           ├── Kessoku Band TOUR “We will B” DISC2
 862.00  B │   ├── BDMV
 144.00  B │   │   ├── ...
 718.00  B │   │   ├── MovieObject.bdmv
 862.00  B │   │   ├── BACKUP
  49.75 KB │   │   ├── CLIPINF
           │   │   ├── META
   1.92 KB │   │   ├── PLAYLIST
  29.53 GB │   │   └── STREAM
  16.88 GB │   │       ├── 00000.m2ts
  37.59 MB │   │       ├── 00001.m2ts
   7.81 MB │   │       ├── 00002.m2ts
  21.13 MB │   │       ├── 00003.m2ts
   1.27 GB │   │       ├── 00004.m2ts
  11.31 GB │   │       ├── 00005.m2ts
  30.00 KB │   │       ├── 00006.m2ts
  30.00 KB │   │       └── 00008.m2ts
 104.00  B │   └── CERTIFICATE
           ├── Kessoku Band TOUR “We will B” DISC3
 554.00  B │   ├── BDMV
 120.00  B │   │   ├── index.bdmv
 434.00  B │   │   ├── MovieObject.bdmv
 554.00  B │   │   ├── BACKUP
  50.93 KB │   │   ├── CLIPINF
           │   │   ├── META
   1.31 KB │   │   ├── PLAYLIST
  31.41 GB │   │   └── STREAM
  30.35 GB │   │       ├── 00000.m2ts
  37.59 MB │   │       ├── 00001.m2ts
   7.81 MB │   │       ├── 00002.m2ts
  21.13 MB │   │       ├── 00003.m2ts
1020.69 MB │   │       ├── 00004.m2ts
 102.00 KB │   │       ├── 00005.m2ts
 132.00 KB │   │       └── 00006.m2ts
 104.00  B │   └── CERTIFICATE
           └── Kessoku Band TOUR “We will B” DISC4
 862.00  B     ├── BDMV
 144.00  B     │   ├── index.bdmv
 718.00  B     │   ├── MovieObject.bdmv
 862.00  B     │   ├── BACKUP
               │   ├── META
   1.91 KB     │   ├── PLAYLIST
  24.63 GB     │   └── STREAM
  11.64 GB     │       ├── 00000.m2ts
  37.59 MB     │       ├── 00001.m2ts
   7.81 MB     │       ├── 00002.m2ts
  21.13 MB     │       ├── 00003.m2ts
   1.50 GB     │       ├── 00004.m2ts
  11.42 GB     │       ├── 00005.m2ts
  30.00 KB     │       ├── 00006.m2ts
  30.00 KB     │       └── 00008.m2ts
 104.00  B     └── CERTIFICATE
```

> Pitfall: file paths contain both double quotes and spaces, making them difficult to write with command-line tools

Judging video content types from file sizes above reveals some details
- The size distribution of DISC1 resembles DISC3, and DISC2 resembles DISC4 most closely
- If these were complete uninterrupted long videos, the structure of DISC2 should match DISC1
    - **Preliminary conclusion: playback observation is needed to see what is going on**
```
DISC1 BDMV/STREAM/
00000.m2ts  42.38 GB   [main feature/concert body]
00001.m2ts  37.59 MB   [common short clip: menu/warning/logo]
00002.m2ts   7.81 MB   [common short clip]
00003.m2ts  21.13 MB   [common short clip]
00004.m2ts   1.33 GB   [bonus/advertisement/menu video]
00005.m2ts  192 KB     [small icon/placeholder/usually unplayable]
00006.m2ts  264 KB     [small icon/placeholder/usually unplayable]

DISC2 BDMV/STREAM/
00000.m2ts  16.88 GB   [main feature/main content A]
00001.m2ts  37.59 MB   [common short clip: menu/warning/logo]
00002.m2ts   7.81 MB   [common short clip]
00003.m2ts  21.13 MB   [common short clip]
00004.m2ts   1.27 GB   [bonus/advertisement/menu video]
00005.m2ts  11.31 GB   [main feature/main content B or long bonus]
00006.m2ts  30 KB      [small icon/placeholder/usually unplayable]
00008.m2ts  30 KB      [small icon/placeholder/usually unplayable]

DISC3 BDMV/STREAM/
00000.m2ts  30.35 GB   [main feature/concert body]
00001.m2ts  37.59 MB   [common short clip: menu/warning/logo]
00002.m2ts   7.81 MB   [common short clip]
00003.m2ts  21.13 MB   [common short clip]
00004.m2ts  1020.69 MB [approx. 1GB: menu/short bonus/advertisement]
00005.m2ts  102 KB     [small icon/placeholder/usually unplayable]
00006.m2ts  132 KB     [small icon/placeholder/usually unplayable]

DISC4 BDMV/STREAM/
00000.m2ts  11.64 GB   [main feature/main content A]
00001.m2ts  37.59 MB   [common short clip: menu/warning/logo]
00002.m2ts   7.81 MB   [common short clip]
00003.m2ts  21.13 MB   [common short clip]
00004.m2ts   1.50 GB   [bonus/advertisement/menu video]
00005.m2ts  11.42 GB   [main feature/main content B or long bonus]
00006.m2ts  30 KB      [small icon/placeholder/usually unplayable]
00008.m2ts  30 KB      [small icon/placeholder/usually unplayable]
```

**Playback observations:**
- DISC1: first half of the concert
- DISC2: rehearsals and interviews during the intermission
- DISC3: second half of the concert
- DISC4: first and second halves of another concert (~~possibly... filler~~)

**Conclusion: already separated into episodes correctly; no joining or splitting needed (single-video mode; Queue Mode is sufficient, Repart Mode is unnecessary).**

---

#### Movie File Structure 1

```
        [BDMV][210623]スーパー戦隊MOVIEレンジャー2021 コレクターズパック 豪華版[Blu-ray]
   3.00 GB ├── DISC 2.iso
   3.13 GB ├── DISC 3.iso
           ├── DISC 1
   2.06 KB │   ├── BDMV
 276.00  B │   │   ├── ...
           │   │   ├── META
   8.37 KB │   │   ├── PLAYLIST
  43.31 GB │   │   └── STREAM
   8.53 GB │   │       ├── 00000.m2ts [movie feature] (surprisingly high compression... because the whole film is only 38 minutes)
  48.00 KB │   │       ├── 00001.m2ts
   3.27 GB │   │       ├── 00002.m2ts [special] (also counts as content)
 373.99 MB │   │       ├── 00003.m2ts [opening theme MV]
 209.58 MB │   │       ├── 00004.m2ts [ending theme MV]
 522.00 KB │   │       ├── 00005.m2ts
 498.00 KB │   │       ├── 00006.m2ts
 522.00 KB │   │       ├── 00007.m2ts
  48.00 KB │   │       ├── 00008.m2ts
 384.00 KB │   │       ├── 00009.m2ts
   4.58 GB │   │       ├── 00010.m2ts [stage greeting 1]
   3.63 GB │   │       ├── 00011.m2ts [stage greeting 2] (one month after #1)
   1.36 GB │   │       ├── 00012.m2ts [short trailer]
   6.52 GB │   │       ├── 00013.m2ts [feature behind-the-scenes]
   6.34 GB │   │       ├── 00014.m2ts [long trailer]
 385.13 MB │   │       ├── 00015.m2ts [guess: special] (short)
 122.64 MB │   │       ├── 00016.m2ts [advertisement]
   3.23 GB │   │       ├── 00017.m2ts [release event 1] (before stage greeting 2)
   4.26 GB │   │       ├── 00018.m2ts [release event 2] (...more meetings anyway...)
 366.00 KB │   │       ├── 00019.m2ts
 378.00 KB │   │       ├── 00020.m2ts
 360.00 KB │   │       ├── 00021.m2ts
 390.00 KB │   │       ├── 00022.m2ts
 480.00 KB │   │       ├── 00023.m2ts
 270.00 KB │   │       ├── 00024.m2ts
 444.00 KB │   │       ├── 00025.m2ts
   8.41 MB │   │       ├── 00028.m2ts
 433.17 MB │   │       ├── 00029.m2ts [guess: menu]
  67.64 MB │   │       ├── 00030.m2ts
   8.96 MB │   │       ├── 00031.m2ts
   2.26 MB │   │       ├── 00032.m2ts
   2.25 MB │   │       └── 00033.m2ts
 104.00  B │   └── CERTIFICATE
 320.67 MB ├── SCANS
  96.94 MB └── Bonus CD
```

> Pitfall: DISC2 and DISC3 are not Blu-ray discs; they use a DVD structure

**Playback observations:** DISC2 and DISC3 are DVD versions of DISC1 (feature and special).

**Conclusion: already separated into episodes correctly; no joining or splitting needed (Queue Mode is sufficient, Repart Mode is unnecessary).**

#### Movie File Structure 2

```
        Shakugan.no.Shana.The.Movie.2007.ANiME.DUAL.COMPLETE.BLURAY-ANiMEHD
   2.85 KB ├── BDMV
 264.00  B │   ├── index.bdmv
   2.59 KB │   ├── MovieObject.bdmv
   2.85 KB │   ├── BACKUP
  51.27 KB │   ├── CLIPINF
   5.29 KB │   ├── PLAYLIST
  20.78 GB │   └── STREAM
 255.62 MB │       ├── 00000.m2ts [guess: menu]
   2.60 MB │       ├── 00002.m2ts
   5.27 MB │       ├── 00003.m2ts
   5.44 MB │       ├── 00004.m2ts
   5.71 MB │       ├── 00005.m2ts
   2.55 MB │       ├── 00010.m2ts
  17.28 GB │       ├── 00012.m2ts [feature]
   5.07 MB │       ├── 00013.m2ts
 193.61 MB │       ├── 00014.m2ts
 217.07 MB │       ├── 00015.m2ts
 284.91 MB │       ├── 00016.m2ts
  73.76 MB │       ├── 00017.m2ts
 103.01 MB │       ├── 00018.m2ts
 259.05 MB │       ├── 00019.m2ts
 317.54 MB │       ├── 00020.m2ts
 170.98 MB │       ├── 00021.m2ts
 177.29 MB │       ├── 00022.m2ts
   1.21 GB │       ├── 00023.m2ts [opening theme MV]
 250.35 MB │       ├── 00024.m2ts
  11.37 MB │       └── 00025.m2ts
 104.00  B └── CERTIFICATE
```

**Conclusion: already separated into episodes correctly; no joining or splitting needed (Queue Mode is sufficient, Repart Mode is unnecessary).**

---

## Blu-ray Playlists

As the examples above show, judging source types solely from file structure and sizes is tedious and error-prone. A better approach is to parse playlists, reproducing the browsing experience of a Blu-ray player.

1cenc reads the PLAYLIST folder of a Blu-ray structure via ChapterTools Core, summarizes every playlist and sub-playlist, and lets you build an encoding queue or repart queue by selecting the lists you need.

### PLAYLIST (.mpls)

Playlist files. One `.mpls` file usually defines one Playlist, which uses PlayItems to reference specific `.m2ts` streams and to specify the video, audio, and subtitle combination.

Common structures by Playlist organization and purpose can be summarized as follows:

#### One Video, Multiple Subtitles

Multiple Playlists point to the **same video** but select different subtitle/audio combinations
* **One file, multiple Playlists**: one `.mpls` contains several Playlists with different subtitles
* **Multiple files, one Playlist each**: several `.mpls` files each contain one Playlist pointing to the same video Clip with different subtitles
* **Characteristics**:
  * Playback durations are nearly identical
  * Referenced `.m2ts` files overlap heavily
  * Differences usually lie in subtitle, audio, and other stream selections

#### Feature and Bonus Content

Different Playlists correspond to independent content such as the feature, concert body, bonuses, and behind-the-scenes footage

* **One file, multiple Playlists**: one `.mpls` contains several Playlists pointing to different content
* **Multiple files, one Playlist each**: each `.mpls` corresponds to one piece of content
* **Characteristics**:
  * The feature usually has the **longest playback duration**
  * Movies and concerts usually have fewer Playlists, and each Playlist references a relatively concentrated set of Clips
  * Series usually have several Playlists of similar duration
  * Series Playlist/Clip numbers often follow continuous or periodic patterns
  * A single Playlist may reference more than 4 Clips to form one complete episode

#### Advertisements, MVs, and Shorts
Advertisements, music videos (MVs), trailers, short bonuses, and other short content.

* **Common case**: one `.mpls` corresponds to one main Playlist
* **Characteristics**:
  * Playback duration is clearly shorter than the feature
  * Usually references only a few `.m2ts` Clips
  * Playlist/Clip numbers sometimes show weak sequential or grouping patterns
  * On anime BDs, these may also appear as standalone shorts such as OPs, EDs, PVs, or CMs

> The above is purely empirical judgment (bluntly speaking, educated guessing), not a classification standard

---

## Importing Blu-ray Playlists

### Queue Mode

A tool for quickly importing the feature from a Blu-ray disc structure in Queue Mode and Repart Mode.

1. Click "Queue Mode" in the 1cenc main window
2. Select "Confirm" in the "Import Blu-ray?" dialog
  - <img src="./img2-all/2-Queue-Mode-Import-Branch.png" alt="Queue Mode import branch" width=500 />
3. Select the `PLAYLIST` folder inside a Blu-ray (`BDMV`) folder and confirm — the Blu-ray playlist selector appears:
  - <img src="./img2-all/3-BDPlaylistSelector.png" alt="Blu-ray playlist selector 1" width=700 />
4. Identify the cluster containing the feature video stream in the first column
    - The example uses the concert source above, so judge by longest duration — select the first cluster
5. From the Playlists in the second column, select the feature list and click Add at the bottom of that column
    - The cluster in the example contains only one Playlist, so add it directly
6. Decide whether more Playlists need encoding, add them to the Final Playlist, and click "Done"
    - Judged here as nothing more to add

**Final Playlist:**

<img src="./img2-all/4-BDPlaylistSelector-1.png" alt="Blu-ray playlist selector 2" width=700 />

**Queue import complete:**

<img src="./img2-all/5-BDPlaylistSelector-2.png" alt="Blu-ray playlist selector 3" width=500 />

> Meanwhile, this "terrifying" path with spaces and double quotes was parsed correctly

### Repart Mode
1. Click "Repart Mode" in the 1cenc main window
2. Select "Confirm" in the "Import Blu-ray?" dialog
3. Select the Final Playlist and confirm
  <img src="./img2-all/6-BDPlaylistSelector-3.png" alt="Blu-ray playlist selector 4" width=500 />

This time you can see an extra warning: *The source video is flagged as interlaced.* — conversion may be required before processing.

#### Blu-ray Interlaced Video

For historical reasons, the original Blu-ray standard has limited support at 1080p:
- 1080p @ 23.976 / 24 fps (progressive, movies)
- 1080i @ 29.97 / 25 fps (interlaced, i.e. 59.94i / 50i, TV)
- 1080p @ 59.94 / 50 fps (not supported by BD-ROM; belongs to the Ultra HD 4K Blu-ray specification)

As a result, most Blu-ray players reject 30fps progressive sources. This produced two compatibility strategies:
1. Signal an interlaced decoding scheme via SPS/PPS to implement pseudo-29.97fps
    - Actually progressive; visible as soon as you play it
    - Can be re-encoded losslessly, but some "stubborn players" reportedly force extra deinterlacing, losing quality
2. Encode as interlaced, possibly with 3:2 or another Pulldown pattern, to implement 1080i 29.97
    - Requires frame-by-frame inspection to identify the Pulldown pattern and select the correct restoration filter set

> 1cenc has limited Pulldown restoration ability, so the traditional manual handling is still needed for now; it takes a long explanation and is omitted here
> There may be more than two compatibility strategies

**Playback observations:** the source in the figure is pseudo-interlaced video (case 1), so you can click Confirm to continue.

#### Repart Editor

Used to re-episode N input videos into M output videos (virtual concatenation + splitting). The mechanism is not original to 1cenc; it references [Haruite/BluraySubtitle](https://github.com/Haruite/BluraySubtitle), a powerful fully automatic Blu-ray BDRip project.

**Virtual concatenation:** assumes all sources have already been joined into one long stream; the concat-then-split encoding commands are only created during final command generation

<img src="./img2-all/7-RepartConfModal.png" alt="Repart editor" width=650 />

- Top: virtual overall timeline, formed by joining the input sources at the lower left in order
  - Vertical lines: split lines, producing the final video queue
- Middle: split-line editing tools for adding, moving, deleting, and fine-tuning split positions
- Bottom: output queue list, updated when split lines change; order can be adjusted manually when split lines stay fixed
- Right: multi-frame preview for confirming split positions, rendered whenever any split line is selected

**Basic operations**
- **Blu-ray playlist import:** chapter information is converted into split lines — keep or remove them depending on whether splitting is needed
- **Regular import:** enter the timestamp of each real split into the split-line editor and click Add (or double-click an approximate timeline position, then refine it)
- Double-click empty timeline space to add a new split line
- Drag split lines for approximate moves, but they cannot cross neighboring split lines

**Deciding whether to keep split lines**
Using the concert source above as an example:
- Preview → the venue lights go out at a split line — presumably split by song
  - Confirm by playback inspection, then split according to episode needs
- Some split lines sit very close together → preview inspection → black-screen content — presumably short breaks and stage preparation
  - Keep the split lines so these segments can be excluded from the output list

---

## Advanced Workdlow Tutorial Complete

### Unmentioned Content
- Abnormal Blu-ray formats — conjoined discs, over-segmented discs