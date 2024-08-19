**Windows端：**

列出所有可用音频设备：

```sh
ffmpeg -list_devices true -f dshow -i dummy
```

发送音频流的命令：

```sh
ffmpeg -f dshow -i audio="麦克风阵列 (Realtek(R) Audio)" -f mpegts udp://192.168.50.162:8000
```

**Linux端：**

**直接播放音频流：**

```sh
ffmpeg -i udp://@:8000 -f alsa hw:1,0
```

**使用`ffplay`播放音频流：**

```sh
ffmpeg -i udp://@:8000 -f s16le -ar 48000 -ac 2 - | ffplay -f s16le -ar 48000 -ac 2 -
```





### 示例：

### **在Linux Debian上设置音频流发送和接收**

1. **从Linux发送音频到Windows：**

   ```sh
   ffmpeg -f alsa -i default -f mpegts udp://192.168.50.106:12345
   ```

2. **从Windows接收音频并播放：** 在另一个终端窗口或后台进程中运行以下命令：

   ```sh
   ffmpeg -i udp://@12346 -f s16le -ar 44100 -ac 2 - | ffplay -
   ```

### 在Windows上设置音频流发送和接收

1. **从Linux接收音频并播放：** 打开命令提示符或PowerShell，并运行：

   ```sh
   ffmpeg -i udp://@:12345 -f s16le -ar 44100 -ac 2 - | ffplay -
   ```

2. **从Windows发送音频到Linux：** 运行以下命令：

   ```sh
   ffmpeg -f dshow -i audio="Microphone" -f mpegts udp://192.168.50.162:12346
   ```



## 使用RTP进行音频传输

### 发送：

Windows：

```sh
ffmpeg-f dshow -i audio="麦克风阵列 (Realtek(R) Audio)" -c:a aac -f rtp rtp://192.168.50.151:12345 -sdp_file output.sdp
```

**`-f dshow`**：

- 这是 FFmpeg 的输入格式选项，指定使用 DirectShow (`dshow`) 作为输入设备。DirectShow 是 Windows 上的多媒体框架，通常用于捕获音频和视频设备的数据。

**`-i audio="麦克风阵列 (Realtek(R) Audio)"`**：

- 这是输入选项，指定了您要从哪个音频设备捕获数据。在这种情况下，音频输入设备是 `"麦克风阵列 (Realtek(R) Audio)"`。您可以使用这个设备名来捕获来自麦克风的音频。

**`-c:a aac`**：

- 这是编码选项，指定要使用 AAC（Advanced Audio Coding）作为音频编码器。AAC 是一种常见的音频编码格式，广泛用于流媒体应用。

**`-f rtp`**：

- 这是输出格式选项，指定输出格式为 RTP（Real-time Transport Protocol）。

**`rtp://192.168.50.151:12345`**：

- 这是输出 URL，指定音频流将通过 RTP 协议发送到 IP 地址 `192.168.50.151` 和端口 `12345`。请确保目标设备可以接收此 RTP 流。

**`-sdp_file output.sdp`**：

- 这个选项指定生成一个 SDP 文件，文件名为 `output.sdp`。该文件包含 RTP 会话的信息，可以用于配置接收方接收和解码 RTP 流。

Linux：

```sh
ffmpeg -f alsa -i default -c:a aac -f rtp rtp://192.168.50.151:12345 -sdp_file output.sdp
```

接收：

保证上述发送端所保存的output.sdp也在接收端当前目录下

```sh
ffplay -protocol_whitelist "file,udp,rtp" -i output.sdp
```



 SDP 文件描述了一个 RTP 音频流的会话配置，它包含了音频流的传输信息、编码格式、以及其他媒体描述信息。

```
v=0
```

- **v=0**：表示 SDP 的版本号为 0。

```sh
o=- 0 0 IN IP4 127.0.0.1
```

- o=- 0 0 IN IP4 127.0.0.1

  ：这是会话的创建者和会话标识符。

  - `o=-`：表示会话创建者未指定用户名。
  - `0 0`：会话和版本标识符，通常用于会话的唯一性。
  - `IN IP4 127.0.0.1`：表示会话创建者的网络地址，`127.0.0.1` 是本地主机地址。

```
s=No Name
```

- **s=No Name**：会话名称字段。此处为 "No Name"，表示未为此会话指定特定的名称。

```
c=IN IP4 192.168.50.151
```

- **c=IN IP4 192.168.50.151**：连接信息，表示该 RTP 流的接收地址为 `192.168.50.151`，这意味着音频流将发送到此 IP 地址。这通常是接收 RTP 数据包的地址。

```
t=0 0
```

- **t=0 0**：时间信息，表示该会话的起始和结束时间为 "0"，通常表示会话的持续时间为无限。

```
a=tool:libavformat 58.76.100
```

- a=tool:libavformat 58.76.100：这是一个属性字段，表示 SDP 文件是由 

  libavformat工具生成的，版本为 58.76.100。

```
m=audio 12345 RTP/AVP 97
```

- m=audio 12345 RTP/AVP 97：媒体描述信息。
  - `audio`：表示传输的是音频流。
  - `12345`：表示音频流传输的**端口号**为 `12345`。
  - `RTP/AVP`：表示 RTP 传输协议，`AVP` 意为音频/视频配置。
  - `97`：表示动态负载类型（payload type），将在后续字段中指定其具体格式。

```
b=AS:128
```

- **b=AS:128**：带宽信息。`AS` 表示会话带宽是 `128 kbps`，指定音频流的最大带宽。

```
a=rtpmap:97 MPEG4-GENERIC/48000/2
```
