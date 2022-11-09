# AutoCut: 通过字幕来剪切视频

AutoCut对你的视频自动生成字幕。然后你选择需要保留的句子，AutoCut将对你视频中对应的片段裁切并保存。你无需使用视频编辑软件，只需要编辑文本文件即可完成剪切。

## 使用例子

假如你录制的视频放在 `2022-11-04/` 这个文件夹里。那么运行

```bash
autocut -d 2022-11-04
```

> 提示：如果你使用OBS录屏，可以在 `设置->高级->录像->文件名格式` 中将空格改成`/`，既 `%CCYY-%MM-%DD/%hh-%mm-%ss`。那么视频文件将放在日期命名的文件夹里。

AutoCut将持续对这个文件夹里视频进行字幕抽取和剪切。例如，你刚完成一个视频录制，保存在 `11-28-18.mp4`。AutoCut将生成 `11-28-18.md`。你在里面选择需要保留的句子后，AutoCut将剪切出 `11-28-18_cut.mp4`，并生成 `11-28-18_cut.md` 来预览结果。

你可以使用任何的Markdown编辑器。例如我常用VS Code和Typora。下图是通过Typora来对 `11-28-18.md` 编辑。

![](imgs/typora.jpg)

全部完成后在 `autocut.md` 里选择需要拼接的视频后，AutoCut将输出 `autocut_merged.mp4` 和对应的字幕文件。

## 安装

首先安装 Python 包

```
pip install git+https://github.com/mli/autocut.git
```

> 上面将安装 [pytorch](https://pytorch.org/)。如果你需要GPU运行，且默认安装的版本不匹配的话，你可以先安装Pytorch。如果安装 Whipser 出现问题，请参考[官方文档](https://github.com/openai/whisper#setup)。

另外需要安装 [ffmpeg](https://ffmpeg.org/)

```
# on Ubuntu or Debian
sudo apt update && sudo apt install ffmpeg

# on Arch Linux
sudo pacman -S ffmpeg

# on MacOS using Homebrew (https://brew.sh/)
brew install ffmpeg

# on Windows using Scoop (https://scoop.sh/)
scoop install ffmpeg
```

## 更多使用选项

### 转录某个视频生成`.srt`和`.md`结果。

```bash
autocut -t 22-52-00.mp4
```

1. 如果对转录质量不满意，可以使用更大的模型，例如

    ```bash
    autocut -t 22-52-00.mp4 --whisper-model large
    ```

    默认是`small`。更好的模型是`medium`和`large`，但推荐使用GPU获得更好的速度。也可以使用更快的`tiny`和`base`，但转录质量会下降。

2. 如果你视频中有较多的长停顿，可以用`--vad`来使用格外的VAD模型预先识别这些停顿，使得对时间戳识别更准确。


### 剪切某个视频

```bash
autocut -c 22-52-00.mp4 22-52-00.srt 22-52-00.md
```

1. 默认视频比特率是 `10m`，你可以更加需要调大调小。
2. 如果不习惯Markdown文件，你也可以直接在`srt`文件里删除不要的句子，在剪切时不传入`md`文件名即可。就是 `autocut -c 22-52-00.mp4 22-52-00.srt`


### 一些小提示


1. 讲得流利的视频的转录质量会高一些，这因为是Whisper训练数据分布的缘故。对一个视频，你可以先粗选一下句子，然后在剪出来的视频上再剪一次。

2. ~~最终视频生成的字幕通常还需要做一些小编辑。你可以直接编辑`md`文件（比`srt`文件更紧凑，且嵌入了视频）。然后使用 `autocut -s 22-52-00.md 22-52-00.srt` 来生成更新的字幕 `22-52-00_edited.srt`。注意这里会无视句子是不是被选中，而是全部转换成`srt`。~~

2. 最终视频生成的字幕通常还需要做一些小编辑。但`srt`里面空行太多。你可以使用 `autocut -s 22-52-00.srt` 来生成一个紧凑些的版本 `22-52-00_compact.srt` 方便编辑（这个格式不合法，但编辑器，例如VS Code，还是会进行语法高亮）。编辑完成后，`autocut -s 22-52-00_compact.srt` 转回正常格式。

3. 用Typora和VS Code编辑markdown都很方便。他们都有对应的快捷键mark一行或者多行。但VS Code视频预览似乎有点问题。

4. 视频是通过ffmpeg导出。在Apple M1芯片上它用不了GPU，导致导出速度不如专业视频软件。

5. 默认输出编码是 `utf-8`. 你可以通过`--encoding`指定其他编码格式。但是需要注意生成字幕文件和使用字幕文件剪辑时的编码格式需要一致。例如使用 `gbk`.

    ```bash
    autocut -t test.mp4 --encoding=gbk
    autocut -c test.mov test.srt test.md --encoding=gbk
    ```

    如果使用了其他编码格式（如gbk等）生成md文件并用Typora打开后，该文件可能会被Typora自动转码为其他编码格式，此时再通过生成时指定的编码格式进行剪辑时可能会出现编码不支持等报错。因此可以在使用Typora编辑后再通过VS Code等修改到你需要的编码格式进行保存后再使用剪辑功能。
