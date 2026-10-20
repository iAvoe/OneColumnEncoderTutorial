# The Crucible

TL;DR: This is not part of the tutorial, and it is very hard to understand. You probably don't want to know this.

---

## Encoding and Decoding Process

Note: Actual behavior varies depending on hardware and software, user settings, and operating system logic.

### Video Opening & Decoding

1.  The user clicks to open a video file.
2.  The operating system identifies the default application based on the file extension.
3.  The application locates the video in the file system using the file path and reads the file's metadata, including the header and other format information.
4.  Based on the metadata, the application determines the required decoder and other resources, then allocates memory space.
5.  The application invokes the appropriate decoder to parse the video file and decode the video and audio data into uncompressed raw data.
6.  The decoded video data is sent to the GPU (Graphics Processing Unit), while the audio data is sent to the DAC (Digital-to-Analog Converter).
7.  The GPU converts the image signals into images and outputs them to the display.
8.  The display's internal circuitry converts the image data into control signals for individual pixel circuits to render the image.
9.  The DAC converts the audio signals into analog signals and plays them through the speakers.

### Video Encoding

1.  **Encoder Launching**
    - Users can launch the encoder in various ways:
        - If the encoder has built-in demuxing and decoding libraries (such as Lavf), it can automatically demux, decode, and mux video files:
            - GUI software launches the encoder program by sending CLI commands or using CLI tools like Bash or CMD. 
            - The encoder typically muxes the video file automatically based on the output file extension. 
        - If the encoder lacks built-in demuxing and decoding capabilities, it typically utilizes ffmpeg's internal demuxing tools and encoders. 
            - Pipe can be used to connect upstream tools (e.g., ffmpeg, VapourSynth, AVS2YUV) to downstream encoders (e.g., x264/x265).
2.  **Demuxing and Decoding**
    - The demuxing and decoding tools pass the uncompressed video stream (in YUV for MPEG or RAW format) to the encoder, providing the input signal.
        - If the input is an RGB24 RAW color space video stream, the encoder's built-in color space conversion function is typically used to convert it to YUV format. 
        - If the encoder is part of a recording device, the video stream must be split into video frames based on the recording settings.
3.  **Encoding Process**
    - **Lookahead**: Pre-analyzes the complexity, motion, and estimated scene cuts of upcoming frames, and makes decisions regarding I, P, and B frame types.
        - Frame type decisions are determined using a shortest-path algorithm.
    - **Coarse Partitioning**: A conceptual step where the codec operates directly on specific locations based on designated available sizes; operations such as searching proceed assuming this step is already complete.
        - AVC: Macroblock (MB) (16x16)
        - HEVC: Coding Tree Unit (CTU) (Max 64x64, Min 32x32)
        - AV1: Superblock (Max 128x128; however, for compatibility with existing hardware encoding/decoding, many steps are effectively equivalent to 64x64)
    - **Motion Estimation (ME)**: identifies similar regions between the current frame and reference frames and assigns motion vectors to each block.
    - **Motion Compensation (MC)**: compares the actual image with the vectors found to generate a prediction block and compensates for alignment offsets with quarter-pixel precision. 
    - **Fine Partitioning**: Based on the results of ME/MC, macroblocks or CTUs are further subdivided down to the smallest 4x4 or nxn coding blocks. 
    - **Inter-frame Residual Coding**: The actual image is compared with the image derived from motion search and compensation to obtain a residual; this residual is transformed and quantized, stored in P or B frames, and combined with I-frames during decoding to reconstruct the original image.
    - **Intra Prediction**: For an intra-coded block, the encoder evaluates candidate intra-coding modes (such as Angular, DC, Planar, or None) to determine which offers the best redundancy reduction; the selected mode yields the intra-predicted block (the "prediction block").
        - If Rate-Distortion Optimization (RDO) is enabled, the encoder further verifies which mode delivers the best image quality for a given bitrate.
    - **Intra Residual Coding**: The encoder compares the actual image data with the intra-predicted block to generate a residual block; this residual is transformed and quantized, then stored alongside the prediction block so that, during decoding, it can be combined with the prediction mode data to reconstruct the original image. 
    - **Skip Block Coding**: Blocks designated as "skip" bypass standard inter- or intra-frame residual coding (though transform and quantization steps are still applied). 
    - **Transform**: Converts image data from the spatial domain into frequency-domain components, ranging from low-frequency to high-frequency signals. 
    - **Quantization**: Reduces high-frequency signal components based on user-defined quality settings; this step has the most significant impact on both image quality and file size.
    - **Entropy Coding / Syntax Coding**: Encodes the transformed and quantized frequency-domain components into the most compact binary representation possible.
4. **Generating the Video Stream**
    - The encoder packages the generated data stream into a file, organizing it by GOP (Group of Pictures) to produce the complete video stream file.
    - If necessary, metadata such as color space information, SEI (Supplemental Enhancement Information), VUI (Video Usability Information), and HDR side data is appended.
5. **Multiplexing (mux) the Video Stream**
    - In networking terms: encapsulation
        - The video stream can be encapsulated into formats like `.mp4`, `.mkv`, or `.mov` using various methods:
        - If the encoder has built-in muxing/demuxing libraries (such as Lavf), it can automatically perform encapsulation based on the file extension specified in the output command line. 
        - If the encoder lacks built-in muxing/demuxing capabilities, it typically utilizes FFmpeg's internal encoding and muxing tools to automatically encapsulate the video once encoding is complete.
    - You can use tools such as [ffmpeg](http://ffmpeg.org/download.html), [MP4Box](https://www.videohelp.com/software/MP4Box), or [MKVToolNix](https://mkvtoolnix.download/) to perform the encapsulation (muxing).
    - Encapsulating files simplifies the player's program logic (for example, allowing the audio and video formats to be identified before reading the streams, rather than opening all decoders and subsequently closing incompatible ones). 
    - If necessary, include additional files such as audio streams/tracks, subtitle tracks, and fonts (though this depends on the compatibility of the container format).

---

## 编解码的过程

注：实际情况下会因为软硬件，用户设定，以及操作系统逻辑的不同而变。

### 视频的打开

1.  用户点击打开视频文件
2.  操作系统根据文件扩展名，查找默认的应用程序
3.  应用程序根据文件路径，在文件系统中定位视频，并读取文件的元数据，包括文件头和其他格式信息
4.  应用程序根据文件的元数据确定所需的解码器和其他资源，然后在计算机的内存中分配空间
5.  应用程序调用适当的解码器来解析视频文件，并将视频和音频数据解码为未压缩的原始数据
6.  解码后的视频数据传递给图形处理芯片 GPU，而音频数据则传递给声音转换芯片 DAC。
7.  GPU 将图像信号转换为图像，输出到显示器
8.  显示器上的芯片将图像数据转换为对每个像素上电路的控制，并显示出来
9.  DAC 将音频信号转换为模拟信号，并通过扬声器播放出来

### 视频的压制
1.  **打开编码器**
    -   用户可以通过多种方法打开编码器：
        -   如果编码器内置了如 Lavf 的解封装和解码动态链接库，则编码器可以自动解封装、解码和封装视频文件：
            -   GUI 软件通过发送 CLI 命令，或者使用像 Bash/CMD 这样的 CLI 工具来打开编码器程序。
            -   编码器通常根据输出文件的后缀名自动封装视频文件。
        -   如果编码器没有内置解封装解码功能，则通常会使用 ffmpeg 内部的解封装工具和编码器。
            -   可以使用 pipe 将上游如 ffmpeg、VapourSynth、AVS2YUV 连接到下游如 x264/5 的编码器
2.  **解封装和解码**
    -   解封装解码工具将已解压缩的 YUV for MPEG 或 RAW 格式视频流传递给编码器，编码器得到输入信号。
        -   如果输入是 RGB24 的 RAW 色彩空间视频流，通常需要使用编码器内置的色彩空间转换功能将其转换为 YUV 格式。
        -   如果编码器是录像设备的一部分，则需要根据录像设置将视频流拆分为视频帧。
3.  **编码过程**
    -   **前瞻进程（Lookahead）**：预先分析未来几帧的复杂度、运动和场景切换 scenecut 的估计、I-P-B 帧类型决策
        -   帧类型决策由最短路径算法（Shortest Path）确立
    -   **粗分块**：虚拟步骤，编解码器直接根据指定可用的大小开始在特定位置上操作，因此搜索等操作默认这步已经完成
        -   x264：宏块 Macroblock MB（16x16）
        -   x265：编码树单元 Coding Tree Unit CTU（最大 64x64、最小 32x32）
        -   AV1：特大块 Superblock（最大 128x128，但为了兼容现有硬编硬解、所以很多步骤上等效于 64x64）
    -   **动态搜索**：Motion Estimation（ME）找到当前帧与参考帧之间的相似区域，并为每个块分配动态向量。
    -   **动态补偿**：Motion Compensation（MC）对比实际画面与搜索到的向量生成预测块，并补偿¼子像素精度的对齐偏差。
    -   **细分块**：根据 MEMC，进一步细分宏块或编码树单元到最小的 4x4 或 nxn 编码块上。
    -   **帧间残差编码**：对比实际画面与动态搜索补偿后所得图像得到残差，对残差进行变换和量化，存储到 P-B 帧上，在解码时与 I 帧叠加以得到原始画面。
    -   **帧内预测**：在帧内编码块上检查候选出冗余最好的帧内编码方法（夹角、DC、趋平、无），选中方法后得到帧内预测后的块，即预测块。
        -   开启率失真优化（RDO）则进一步检查画质同码率下画质是否也最好
    -   **帧内残差编码**：对比实际画面与帧内预测得到的残差块，进行变换和量化，存储在预测块中，在解码时与预测模式叠加以得到原始画面。
    -   **跳过块编码**：跳过帧间或帧内残差编码的块，进行变换和量化。
    -   **变换**：将图像从空间域转换到低频到高频信号分量之间的每个级别，也称为频域。
    -   **量化**：根据用户设置的质量级别，削减高频信号分量，这一步骤对画质和文件大小影响最大。
    -   **熵编码/文本编码**：将变换和量化后的频域分量统计为最小可能的二进制数。
4.  **生成视频流**
    -   编码器将生成的数据流逐个 GOP Group of Pictures 打包为文件，得到完整视频流文件
    -   如有必要，附加如色彩空间、Supplemental Enhancement Information SEI、VUI、HDR 等元数据
5.  **封装视频流**
    -   用户可以通过多种方法将视频流封装为 .mp4、.mkv、.mov 等格式：
        -   如果编码器内置了像 Lavf 的解封装解码动态链接库，编码器可以根据输出命令行的文件后缀名自动完成。
        -   如果编码器没有内置解封装解码功能，则通常会使用 ffmpeg 内部的编码器和解封装工具，使编码完成后自动封装。
        -   可以使用工具如 [ffmpeg](http://ffmpeg.org/download.html)、[MP4Box](https://www.videohelp.com/software/MP4Box)，[MKVToolNix](https://mkvtoolnix.download/) 等来进行封装。
    -   封装文件使得播放器的程序逻辑变得简单（例如，在读取视频流之前就获取视音频格式，而不是先打开所有解码器，再关掉不兼容的解码器）
    -   如有必要，附加音频流/音轨、字幕轨、字体等文件（但要根据封装文件的兼容性判断）

