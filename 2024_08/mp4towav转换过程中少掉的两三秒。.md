## Issue:

![Snipaste_2024-08-06_20-06-08](https://cdn.jsdelivr.net/gh/MrXnneHang/Blog_img/BlogHosting/img/24/08/202408062007520.jpeg)

原视频mp4和m4a一样长。

但是转录成其他的长度都发生了变化，然后在根据wav->srt的时候自然而然的音频发生了错位。

测试软件有:

**Moo0 Mp3Converter v1.38 Installer ->44100Hz**

**上面那个软件**

**ffmpeg默认选项:**

```cmd
ffmpeg -i input.mp4 output.wav
```

结果无一例外，而且转出来都是13:37。

## Solve:

但是今天，一个不会代码的人反而solve了这个issue，说起来也挺神奇的。现在有ai真的会把门槛降低，但我也知道这也需要付出大量时间，不停debug，换方式问。他很执着啊。

```python
def extract_audio(self, video_path):
    if not os.path.isfile(video_path):
        self.progress_queue.put((video_path, "文件不存在", 0))
        return

    output_dir = os.path.dirname(video_path)
    output_filename = os.path.splitext(os.path.basename(video_path))[0] + ".wav"
    output_path = os.path.join(output_dir, output_filename)

    self.progress_queue.put((video_path, "", 0))
    try:
        command = [
            FFMPEG_PATH, 
            '-i', video_path, 
            '-vn', 
            '-af', 'aresample=async=1',
            '-acodec', 'pcm_s16le', 
            '-ar', '44100', 
            '-ac', '2', 
            '-y',  # 这个选项会覆盖输出文件
            output_path
        ]

        process = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

        while True:
            output = process.stderr.read(1)
            if process.poll() is not None and output == b'':
                break
                if output:
                    progress = self.calculate_progress(output)
                    self.progress_queue.put((video_path, "", progress))

                    process.wait()
                    if process.returncode == 0:
                        self.progress_queue.put((video_path, f"成功提取音频到: {output_path}", 100))
                    else:
                        self.progress_queue.put((video_path, f"提取失败: {video_path}", 0))

                    except subprocess.CalledProcessError as e:
                        self.progress_queue.put((video_path, f"发生错误: {video_path}", 0))
```

用这个，并不会少几秒。而且并不是简单地添加silence，而是和原视频的音频对齐。

也就是说，之所以转换出bug是编解码器的问题。

![Snipaste_2024-08-06_20-16-24](https://cdn.jsdelivr.net/gh/MrXnneHang/Blog_img/BlogHosting/img/24/08/202408062016744.jpeg)

```cmd
'-vn', 
'-af', 'aresample=async=1',
'-acodec', 'pcm_s16le', 
'-ar', '44100', 
'-ac', '2', 
```

我不太喜欢用ffmpeg的原因之一就是它太抽象了。这些参数我一个没看懂。但不得不说它很强。