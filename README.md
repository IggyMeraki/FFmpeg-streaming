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

