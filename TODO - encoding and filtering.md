# Encoding and Filtering

## Types of compressions

**Uncompressed (Raw)**
- Everything as-is, usually unintelligibly massive for video files
- Storage and RAM read-write (IO) cost scales in “lightspeed”
- Roughly no compute cost on decoding (reconstructing from compressed data)

**Bitrate**
- The size unit of *data flowing* per second
- In b/s, Kb/s, Mb/s, Gb/s. Here “b” is for bit, (Byte÷8)
- Or bps, Kbps, Mbps, Gbps, ...

**Lossless compression**
- Gigantic amount of data stored for video files
- Storage and RAM read-write (IO) cost scales very quickly
- Medium compute cost on decoding
- Archive formats (`.zip`, `.rar`, `.7z`) utilizes lossless compressions

**Lossy compression**
- Depends on quality level
  - Higher quality stores more details, closer to lossless compression
  - Lower quality stores very little details
- Due to the details dropped, lower quality compression reduces decoding compute cost

## Formats

**AVC H.264 (x264.exe)**
- Launched in 2003, the most widely supported supported standard
- It is too good that no codec so far can replace it
  - Codecs that are faster does not reach the same compression or quality
  - Codecs that are better in compression or quality are too slow in comparison

**HEVC H.265 (x265.exe)**
- Launched in 2013, a buffed AVC, very slow and hits the CPU harder than Prime95 occasionally
- Patent swamp

**AV1 (AV1AN, RAV1E, SVT-AV1, etc.)**
- Launched in 2018 largely based on Google's VPX codec (standard wise), but improved to a level that can beat HEVC consistently, especially in animations
- Currently most people's mobile phone lacks hardware decoding support

**Common Color Spaces**
| CS | Components | Characteristics | Purpose | Support Range |
|---|---|---|---|---|
| RGB | Red, Green, Blue | General purpose | Universal for displays and cameras | Record, display |
| ARGB | Alpha, Red, Green, Blue | Transparency channel | Facilitates layered editing | Edit, storage |
| CMYK | Cyan, Magenta, Yellow, Black | Subtractive color model | Reduces ink usage in dark/colored prints | Printing |
| YCbCr / YUV, YPbPr | Y (Luminance), Cb (Blue-diff.), Cr (Red-diff.) | Human Vision System (HVS) Compression & Optimization | Shrink chroma resolution since your eye can't tell | Storage |
- This is a small collection

## General Principles

**Performance and lags**
- A combination of computing performance and data transfer (IO) performance
- If the internet or hard drive slows or on 100% load, the lag is due to an IO bottleneck
- If the CPU or GPU is on 100% load, the lag is due to the computing bottleneck

**Garbage in, garbage out**
- **Encoding**
  - Lossy encoding mode can introduce artifacts, which will be treated as details in the next encoding run
  - So, avoid re-encoding a lossy video, unless there are major benefits (half the file size, original video codec/format too old, etc.)
- **Upscaling**
  - Metaphor: A novel with 1 word deleted after every word, an AI model would predict the missing words better than static algorithms, but only the original creater can truly restore it
    - The best approach is to find the original copy of this novel
  - So, avoid upscaling unless the source video resolution is too small (even playing on phone requires upscaling to view in full-screen)
    - 1cenc provided heavier upscale filters than a typical video player offers

**Color accuracy:**
- 1cenc used libplacebo for color conversions in AviSynth, VapourSynth and ffmpeg with recommended settings to ensure color is accurately converted
- However, color conversion involves floating point values and divisions (inherently inaccurate)


### Encoding
**Trade-off expectations**
- Prefect world: having an infinitly fast computer
- Reality: fast → $$$$$$
- Solution: compromise 1 target to satisfy the other 2
```
   [Target]    → Bigger ←     [Target]
  Fast-quick      File      Image clarity
             ***************
              *************
     ↘        ***********       ↙
        Bad     *********     Slow
      Quality    *******    Progress
            ↖    *****    ↗
                   ***
                    *
                [Target]
               Small file
```

**Hardware encoder (fused ASIC) expetations**
- Hardware codecs have additional trade-offs on die size and heat management
- Mobile phone and GPUs: limited die area for computing and IO areas

**Encoding speed expectations**
- Measured in ***Frames per second (fps)***
- When taking 5 second to encode 1 video frame, then it's 0.5fps
- A 10-hour-long clip in 24fps, encoding at 200fps, then it takes about \(\frac{10 \times 60 \times 60 \times 24}{200} = 4320\) seconds to finish

**File size expectations**
- Poor analogy: A 1920x1080 video is roughly equivalent to playing/storing 2073600 audio tracks
- This does not mean larger files → better quality, but more of a size baseline on *duration times codec (AVC, HEVC, AV1)*

**Speed scaling expectations**
- From 1920x1080 to 3840x2160, the increase of pixels quadruples (4x), not doubles
- Since there are many *complicated algorithms* a video encoder executes, this process can scale in 10x or even 20x slower

### Filtering

**Repair first, stylize after, downscale in the end**
- This order is very important
- Imagine cooking the food and later washes it

**Colorspace Profile (CSP) Conversion**
- 1cenc provided the “to Bt.709” conversion, there are 2 conversion types in general:
- **Low to high:** Old colorspace standard (Bt.601), or Narrow Content Gamut (NCG)
- **High to low:** New colorspace standards that technically can list more color values than the current Bt.709, marked as Wide Content Gamut (WCG)
- **HDR/HLG/DOVI to 709:** New colorspace standards combined with High Dynamic Range (HDR)
- Keeping content in high gamut or HDR became a problem when trying to edit with different footages
- Most people don't have device that can properly play HDR (consider Vesa HDR400 isn't HDR)
- So the general principle is to go with Bt.709

**Chroma Subscampling**
- Most video source use 4:2:0 chroma subsampling to sae bandwidth, and most hardware deocder only works with 4:2:0 for all codecs
  - Currently, SVT-AV1 only works with 4:2:0
- If the source video is to be played than archived, convert to YUV 4:2:0 is a good idea

**Denoise**
- Remove unwanted “pixels in a different value than neighbours” effect
- As the behavior stated above, this filter can damange the texture if going hard
- **Noise:** A random pattern as a result from CMOS, film tape crystals, etc.
  - Unnatural noise: Intentially added noise on fully digital-made artworks, such as animations, which already conquered noise introductions
- **Dither:** A pattern to create color gradient by alternating colors in pixels, a technique evolved from old computers that only display a few colors

**Crop**
- Remove unwanted outer areas, edges, or letterbox/pillerbox borders
- Use it before resizing, so the borders are not factored into scaling algorithms

**Resize**
- Change video dimensions (↑/↓) to desired specs
- **Downscale in the end:** Heavy filtering (denoising, debanding, restoration) should occur in native resolution, not a scaled resolution
- Unlock aspect ratio: For videos with wrong width (or height)
  - playback should clearly show the picture is suqeezed/streched to an unnatural proportion (egg-heads)

**Deband**
- Reduces banding artifact via dither, noise and other modifications.
- **Banding:** usually introduced in low bitdepth (8bit and below), The luma-chroma value lacks density to show gradient → staircase gradient instead
- Deband filter should engage later than denoise filter

**Blur & Sharpen**
- A stylizing effect that detect edges, then enhances/reduces its contrast
- **Stylize after repair:** Avoid enhance noise, banding, ringing artifacts

**Subtitle burn-in**
- A stylizing effect that bakes subtitle into video frames. This can be useful for website playback where CC subtitle support is limited or none
- A compromize when involved font files are too large
- Must engage later than all repairing and stylizing filters
- If resizing can result in subtitle size mismatch, the correct move is to edit subtitle file to reduce font size and then burn-in, not burn-then-resize

---

## Encoding Modes

### Constant Rate Factor (CRF)
A factor that influences bitrate. A function maps this value to compression intensity for each video frame in different cases.
- Useful when image quality is a priority
- Usually refers to “the quality setting”

### Average BitRate (ABR)
A target for bitrate to become. Setting this one needs realistic expectations, which requires experiences to set
- Useful only when a hard limit exists

### x264 Basic Parameters

**General Purpose:**
- Parameters tuned to very-slow tier that works for all footages, but thanks to the increasing computing power, it is now the fastest lossy encoding setting

**Stock Footage:**
- General Purpose with mode decision tuned to perserve the source over comporession

### x265 Basic Parameters

**General Purpose:**
- Parameters tuned to slightly slower than x265's native slow preset, works for all footages

**Film / IRL Shoot:**
- Optimized for HQ footage sources, similar to x265's native veryslow preset
- Maxmizing temporal redundancies (longer reference range)
- Increased double-checkings (↑RDO)
- Increased adaptive quantization loads (↑intra-frame block quality distribution effort )

**Stock Footage:**
- General Purpose with mode decision tuned to perserve the source over comporession
- Reduced P, B (reference) frames to lower decoding complexity, so editing software operates smoother

**Anime:**
- Optimized for Animation, similar to x265's native veryslow preset
- Maxmizing block splitting efforts
- HEVC adaptive quantizer
- Longer consecutive B (reference) frames
- Accept some losslessly coded blocks

**Stress Test:**
- Puts extremly intensive load on the CPU, disregarding quality and performance
- May damage or kill your computer if the CPU, PSU or any power ICs are not properly cooled

### SVT-AV1 Basic Parameters

**Peak Qualiy:**
- Gives the best image quality in 1cenc, period
- Still depends on the CRF or ABR sliders for the target/base quality

**High Compression:**
- Similar to peak quality, but turned quality boost settings off

**Fast:**
- Turned off more things than high-compression setting
- Still not as fast as x264, but otherwise better

### Shared Basic Parameters

**None:**
- The default, or use your customization via the “Additional Encoder Parameters” box below

