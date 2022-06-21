# Building Custom FFMPEG
## Why are we building custom FFMPEG?

Pre-built ffmpeg takes approximately 50MB of filesize. We can compile FFMPEG from source code to reduce filesize to minimum and limit features of FFMPEG to just about what we need.

## System Details

The operation mentioned in this file is executed on Ubuntu 22.04 x64. Total CPU cores is 4.

## Use Cases

Our custom FFMPEG build use case are as follows:
1. To convert .TS files to MP4.
2. To convert .MP4(video only)+.M4A(audio only) to MP4.
3. To convert .MP4 to .MP3.


## Steps to compile custom FFMPEG

The steps to compile FFMPEG for our custom needs are as follows.
Most of the things here are referred from this [Compilation Guide](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu).
### Get the Dependencies
* Execute the below code snippet to install few packages required for compiling. Few packages are exempted on purpose as they're not required for our usese.
``` 
sudo apt-get update -qq && sudo apt-get -y install \
  autoconf \
  automake \
  build-essential \
  cmake \
  git-core \
  meson \
  ninja-build \
  pkg-config \
  texinfo \
  wget \
  yasm 

```
* In your home directory make a new directory to put all of the source code and binaries into:
```
mkdir -p ~/ffmpeg_sources ~/bin

```
## Compilation & Installation

* ***Build NASM*** 

NASM is an assembler used by some libraries like libmp3lame. The code for compiling NASM is as below.
```
cd ~/ffmpeg_sources && \
wget https://www.nasm.us/pub/nasm/releasebuilds/2.15.05/nasm-2.15.05.tar.bz2 && \
tar xjvf nasm-2.15.05.tar.bz2 && \
cd nasm-2.15.05 && \
./autogen.sh && \
PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" && \
make && \
make install
```
> Tip : make -j<"number of cup cores"> can be used to speed up compilation process. Ex: If total cores = 4, "make -j4".
* ***Build libmp3lame*** 

libmp3lame is an mp3 encoder. It is required for converting mp4 to mp3. The code for compiling libmp3lame is as below. 

```
cd ~/ffmpeg_sources && \
wget sourceforge.net/projects/lame/files/lame/3.99/lame-3.99.5.tar.gz -qO- | tar -xz && \
cd lame-* && \
./configure --prefix="$HOME/ffmpeg_build" --enable-nasm --disable-shared && \
make && \
make install
``` 
>--disable-shared tag has been used in the above code to prevent building of shared library.
Details as to why static libraries are preffered can be found [here](https://medium.com/swlh/linux-basics-static-libraries-vs-dynamic-libraries-a7bcf8157779#:~:text=Dynamic%20or%20shared%20libraries%20occur,library's%20files%20at%20compile%20time.).

* ***Build FFMPEG***

FFMPEG can be built by specfying the tags we need for our custom built.
Muxers, Demuxers, Encoders & Decoders required for our use cases are mentioned below:
1. To convert TS files to MP4.

    Demuxers - mpegts(.ts)

    Muxers   - mp4(.mp4)

    Decoders - h264, aac

2. To convert MP4(video only)+M4A(audio only) to MP4.
    
    Demuxers - mp4(.mp4)
    
    Muxers   - mp4(.mp4)

3. To convert MP4 to MP3.

    Demuxers - mp4(.mp4)

    Encoders - libmp3lame

    Muxers - mp3 
    
FFMPEG is built using flags for the specified configuration using the command below.
> Tip : make -j<"number of cup cores"> can be used to speed up compilation process. Ex: If total cores = 4, "make -j4".

```
cd ~/ffmpeg_sources && \
wget -O ffmpeg-snapshot.tar.bz2 https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 && \
tar xjvf ffmpeg-snapshot.tar.bz2 && \
cd ffmpeg && \
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --extra-libs="-lpthread -lm" \
  --extra-ldexeflags="-static" \
  --ld="g++" \
  --bindir="$HOME/bin" \
  --disable-everything --disable-network --disable-autodetect --disable-ffplay --disable-ffprobe --disable-filters --disable-doc --enable-ffmpeg --enable-libmp3lame  --enable-demuxer=mpegts,mov --enable-muxer=mp4,mp3 --enable-decoder=h264,aac --enable-encoder=libmp3lame --enable-parser=aac,h264 --enable-protocol=file 
```
```
PATH="$HOME/bin:$PATH" make && \
make install && \
hash -r

```

>--extra-ldexeflags="-static" has been used in the above code to include libraries like libmp3lame as static.

## Completion
On completion run ffmpeg from ffmpeg_sources. Configuration can be confirmed by :
```
./ffmpeg -buildconf
```
For the configuration in our use case the file size would be around 5.3MB with libmp3lame in static mode.

## References
1. [FFMPEG Official Documentation](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu)

2. [FFMPEG with static libraries like libmp3lame](https://github.com/toots/shine/issues/22)

3. [Sample FFMPEG static build.sh file](https://github.com/zimbatm/ffmpeg-static/blob/master/build.sh)








