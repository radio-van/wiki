# Contents

- [GPU Node performance measurement](#GPU Node performance measurement)
    - [Node configuration](#GPU Node performance measurement#Node configuration)
    - [ffmpeg compilation](#GPU Node performance measurement#ffmpeg compilation)
    - [driver headers](#GPU Node performance measurement#driver headers)
    - [measurments](#GPU Node performance measurement#measurments)
        - [1st processing w/o CUDA](#GPU Node performance measurement#measurments#1st processing w/o CUDA)
        - [1st processing w CUDA](#GPU Node performance measurement#measurments#1st processing w CUDA)
        - [2nd processing w/o CUDA](#GPU Node performance measurement#measurments#2nd processing w/o CUDA)
        - [2nd processing w CUDA](#GPU Node performance measurement#measurments#2nd processing w CUDA)
- [CPU Node performance measurement](#CPU Node performance measurement)
    - [Node configuration](#CPU Node performance measurement#Node configuration)
    - [measurements](#CPU Node performance measurement#measurements)
        - [1st processing](#CPU Node performance measurement#measurements#1st processing)
        - [2nd processing](#CPU Node performance measurement#measurements#2nd processing)

# GPU Node performance measurement

## Node configuration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-test
  namespace: experiments
spec:
  containers:
  - command: 
    - sleep 
    - infinity
    name: my-gpu-container
    resources:
      limits:
       nvidia.com/gpu: 1
    image: nvidia/cuda:9.0-devel
    env:
    - name: NVIDIA_DRIVER_CAPABILITIES
      value: "all"
```

## ffmpeg compilation
```bash
apt-get update && apt-get -y install curl xz-utils git gcc yasm pkgconf

cd /usr/local/src/
git clone https://git.ffmpeg.org/ffmpeg.git
cd ffmpeg
git checkout release/3.6

./configure \
  --enable-cuda \
  --enable-cuvid \
  --enable-nvenc \
  --enable-nonfree \
  --enable-libnpp \
  --extra-cflags=-I/usr/local/cuda/include \
  --extra-cflags=-I/usr/local/include \
  --extra-ldflags=-L/usr/local/cuda/lib64
make -j 10
make install
```

## driver headers
**(optional)**  

```bash
cd /usr/local/src/
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd nv-codec-headers && make && make install

export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig"
```

## measurments
### 1st processing w/o CUDA
```bash
time ffmpeg -hide_banner -t 600.0 -i big.mp4 -filter_complex "[0:v]split=1[s0];[s0]scale=height=-2:sws_flags=lanczos:width=trunc(iw*0.5/2)*2[s1];[s1]split=2[s2][s3]" -map [s2] -map 0:a -f mp4 -acodec aac -ar 44100 -pix_fmt yuv420p -preset veryfast -vcodec libx264 output1.mp4 -map [s3] -f image2 -frames 1 cover.jpg -y -y
```
results::
```bash
node 1:36
local 4:39
prod 2:39
```
### 1st processing w CUDA
```bash
time ffmpeg -hide_banner -threads 8 -t 600.0 -hwaccel cuvid -c:v h264_cuvid -i big.mp4 -filter_complex "[0:v]scale_npp=h=-2:interp_algo=lanczos:w=trunc(iw*0.5/2)*2[s1];[s1]split=2[s2][s3];[s3]hwdownload,format=nv12[s4]" -map [s2] -map 0:a -acodec aac -ar 44100 -preset fast -vcodec h264_nvenc output1_cuda.mp4 -map [s4] -vframes 1 cover.jpg -y -y
```
results::
```bash
cuda-node 0:50
```
### 2nd processing w/o CUDA
```bash
time ffmpeg -i output1.mp4 -filter_complex "[0:v]split=1[s0];[s0]split=2[s1][s2];[s1]crop=h=iw*ih/(iw+0):w=iw*ih/(ih+778)[s3];[s3]scale=force_original_aspect_ratio=increase:h=ih+778:w=iw+0[s4];[s4]gblur=sigma=100[s5];[s5][s2]overlay=eof_action=repeat:format=rgb:x=0:y=389[s6];[s6]scale=height=-2:sws_flags=lanczos:width=trunc(iw*0.78125/2)*2[s7];[s7]split=2[s8][s9]" -map [s8] -map 0:a -f mp4 -acodec aac -ar 44100 -pix_fmt yuv420p -preset veryfast -vcodec libx264 final_output_wo_gpu.mp4 -map [s9] -f image2 -frames 1 cover.jpg -y -y
```
results::
```bash
node 4:57
local 9:48
prod 10:11
```
### 2nd processing w CUDA
```bash
time ffmpeg -hide_banner -threads 8 -t 600.0 -hwaccel cuvid -c:v h264_cuvid -i output1_cuda.mp4 -filter_complex "[0:v]split=2[s1][s2];[s1]hwdownload,format=nv12,crop=h=iw*ih/(iw+0):w=iw*ih/(ih+778)[s3];[s3]hwupload,scale_npp=h=ih+778:w=iw+0[s4];[s4]hwdownload,format=nv12,gblur=sigma=100[s5];[s2]hwdownload,format=nv12[s22];[s5][s22]overlay=eof_action=repeat:x=0:y=389[s6];[s6]hwupload,scale_npp=h=-2:interp_algo=lanczos:w=trunc(iw*0.78125/2)*2[s7];[s7]split=2[s8][s9];[s9]hwdownload,format=nv12[s10]" -map [s8] -map 0:a -acodec aac -ar 44100 -preset fast -vcodec h264_nvenc final_output1_cuda.mp4 -map [s10] -vframes 1 cover.jpg -y -y
```
results::
```bash
cuda-node 1:44
```

# CPU Node performance measurement

## Node configuration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ffmpeg-test
  namespace: experiments-cpu
spec:
  containers:
  - command:
    - sleep
    - infinity
    image: eu.gcr.io/smp-io/smmplanner-app:630
    imagePullPolicy: IfNotPresent
    name: my-cpu-container
    resources:
      requests:
        cpu: "15"
    volumes:
     - name: data
       emptyDir:
         medium: Memory
 ```
 
## measurements
### 1st processing
```bash
time seq 1 N | parallel -j N -I{} 'ffmpeg -hide_banner -t 600.0 -i big{}.mp4 -filter_complex "[0:v]split=1[s0];[s0]scale=height=-2:sws_flags=lanczos:width=trunc(iw*0.5/2)*2[s1];[s1]split=2[s2][s3]" -map [s2] -map 0:a -f mp4 -acodec aac -ar 44100 -pix_fmt yuv420p -preset veryfast -vcodec libx264 output{}.mp4 -map [s3] -f image2 -frames 1 cover{}.jpg -y -y'
```
results::
 ```bash
+-----threads------+-----time------+
|                 7/16             |
|    1             |       62      |
|    3             |      103      |
|    5             |      208      |
+----------------15/16-------------+
|    1             |       63      |
|    3             |       82      |
|    5             |      106      | 
|    7             |      140      |
|    8             |      177      |
|   13             |      253      |
+------------------+---------------+
```

### 2nd processing
combined command for 1-7 threads
```bash
time seq 1 1 | parallel -j 3 -I{} 'ffmpeg -i output{}.mp4 -filter_complex "[0:v]split=1[s0];[s0]split=2[s1][s2];[s1]crop=h=iw*ih/(iw+0):w=iw*ih/(ih+778)[s3];[s3]scale=force_original_aspect_ratio=increase:h=ih+778:w=iw+0[s4];[s4]gblur=sigma=100[s5];[s5][s2]overlay=eof_action=repeat:format=rgb:x=0:y=389[s6];[s6]scale=height=-2:sws_flags=lanczos:width=trunc(iw*0.78125/2)*2[s7];[s7]split=2[s8][s9]" -map [s8] -map 0:a -f mp4 -acodec aac -ar 44100 -pix_fmt yuv420p -preset veryfast -vcodec libx264 final_output_wo_gpu{}.mp4 -map [s9] -f image2 -frames 1 cover{}.jpg -y -y' && echo '----------------------------------' && time seq 1 3 | parallel -j 3 -I{} 'ffmpeg -i output{}.mp4 -filter_complex "[0:v]split=1[s0];[s0]split=2[s1][s2];[s1]crop=h=iw*ih/(iw+0):w=iw*ih/(ih+778)[s3];[s3]scale=force_original_aspect_ratio=increase:h=ih+778:w=iw+0[s4];[s4]gblur=sigma=100[s5];[s5][s2]overlay=eof_action=repeat:format=rgb:x=0:y=389[s6];[s6]scale=height=-2:sws_flags=lanczos:width=trunc(iw*0.78125/2)*2[s7];[s7]split=2[s8][s9]" -map [s8] -map 0:a -f mp4 -acodec aac -ar 44100 -pix_fmt yuv420p -preset veryfast -vcodec libx264 final_output_wo_gpu{}.mp4 -map [s9] -f image2 -frames 1 cover{}.jpg -y -y' && echo '----------------------------------' && time seq 1 5 | parallel -j 5 -I{} 'ffmpeg -i output{}.mp4 -filter_complex "[0:v]split=1[s0];[s0]split=2[s1][s2];[s1]crop=h=iw*ih/(iw+0):w=iw*ih/(ih+778)[s3];[s3]scale=force_original_aspect_ratio=increase:h=ih+778:w=iw+0[s4];[s4]gblur=sigma=100[s5];[s5][s2]overlay=eof_action=repeat:format=rgb:x=0:y=389[s6];[s6]scale=height=-2:sws_flags=lanczos:width=trunc(iw*0.78125/2)*2[s7];[s7]split=2[s8][s9]" -map [s8] -map 0:a -f mp4 -acodec aac -ar 44100 -pix_fmt yuv420p -preset veryfast -vcodec libx264 final_output_wo_gpu{}.mp4 -map [s9] -f image2 -frames 1 cover{}.jpg -y -y' && echo '----------------------------------' && time seq 1 7 | parallel -j 7 -I{} 'ffmpeg -i output{}.mp4 -filter_complex "[0:v]split=1[s0];[s0]split=2[s1][s2];[s1]crop=h=iw*ih/(iw+0):w=iw*ih/(ih+778)[s3];[s3]scale=force_original_aspect_ratio=increase:h=ih+778:w=iw+0[s4];[s4]gblur=sigma=100[s5];[s5][s2]overlay=eof_action=repeat:format=rgb:x=0:y=389[s6];[s6]scale=height=-2:sws_flags=lanczos:width=trunc(iw*0.78125/2)*2[s7];[s7]split=2[s8][s9]" -map [s8] -map 0:a -f mp4 -acodec aac -ar 44100 -pix_fmt yuv420p -preset veryfast -vcodec libx264 final_output_wo_gpu{}.mp4 -map [s9] -f image2 -frames 1 cover{}.jpg -y -y'
```

results::
```bash
+-----threads------+-----time------+--2nd process--+
+-----------------7/8--------------+---------------+
|    1             |       75      |      306      |
|    3             |      125      |      442      |
|    5             |      198      |      635      |  
|    7             |      271      |     1137      |
+------------------+---------------+---------------+
```

