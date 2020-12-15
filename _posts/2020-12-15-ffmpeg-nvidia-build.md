# 윈도우에서 NVIDIA GPU 사용하도록 ffmpeg 빌드하기

참고 : [https://docs.nvidia.com/video-technologies/video-codec-sdk/ffmpeg-with-nvidia-gpu/](https://docs.nvidia.com/video-technologies/video-codec-sdk/ffmpeg-with-nvidia-gpu/)

NVIDIA Video Codec SDK Documentation에 보면 상세한 설명이 나와있다. 위 링크에 있는 내용을 요약하여 정리하였다. [Compiling for Windows](https://docs.nvidia.com/video-technologies/video-codec-sdk/ffmpeg-with-nvidia-gpu/#compiling-for-windows) 부분을 보면 된다.

더불어 빌드 진행하면서 겪은 트러블 슈팅을 추가로 정리하였다.

# 목차

[1. MSYS2 설치](#-1-MSYS2-설치)

[2. FFMPEG for NVCODEC](#-2-FFMPEG-for-NVCODEC)

[트러블슈팅](#트러블슈팅)

# 1. MSYS2 설치

우선 윈도우에서 ffmpeg을 빌드하기 위해서는 여러가지 방법들이 있는데, nvidia에서는 msys2를 사용하는 방법을 설명하고 있다.

- msys2 설치 ([www.msys2.org](https://www.msys2.org/))

나는 *C:\msys64* 에 설치하였다. 기존에 설치했던 msys와 경로가 겹치면 적절히 수정해준다. msys2 설치 후 package database와 core system package를 다음과 같이 업데이트 한다.

```bash
$ pacman -Syu
:: 꾸러미 데이터베이스 동기화 중...
 mingw32                                                                           760.3 KiB   654 KiB/s 00:01 [#################################################################] 100%
 mingw32.sig                                                                       438.0   B  0.00   B/s 00:00 [#################################################################] 100%
 mingw64                                                                           761.9 KiB   743 KiB/s 00:01 [#################################################################] 100%
 mingw64.sig                                                                       438.0   B  0.00   B/s 00:00 [#################################################################] 100%
 msys                                                                              273.5 KiB   314 KiB/s 00:01 [#################################################################] 100%
 msys.sig                                                                          438.0   B  0.00   B/s 00:00 [#################################################################] 100%
:: Starting core system upgrade...
경고: terminate other MSYS2 programs before proceeding
의존성 해결 중...
꾸러미 충돌을 찾는 중...

꾸러미 (3) mintty-1~3.4.3-1  msys2-runtime-3.1.7-4  pacman-mirrors-20201208-1

총 다운로드 크기:   3.26 MiB
총 설치 크기:     11.62 MiB
알짜 업그레이드 크기:  -2.94 MiB

:: 설치를 진행하시겠습니까? [Y/n] y
:: 꾸러미 가져오는 중...
 msys2-runtime-3.1.7-4-x86_64                                                        2.6 MiB   273 KiB/s 00:10 [#################################################################] 100%
 mintty-1~3.4.3-1-x86_64                                                           683.8 KiB   485 KiB/s 00:01 [#################################################################] 100%
 pacman-mirrors-20201208-1-any                                                       3.8 KiB  3.66 MiB/s 00:00 [#################################################################] 100%
(3/3) 키링의 키를 검사 중                                                                                      [#################################################################] 100%
(3/3) 꾸러미 무결성 검사 중                                                                                    [#################################################################] 100%
(3/3) 꾸러미 파일 불러오는 중                                                                                  [#################################################################] 100%
(3/3) 파일 충돌 검사 중                                                                                        [#################################################################] 100%
(3/3) 사용 가능한 디스크 공간 검사 중                                                                          [#################################################################] 100%
:: 꾸러미 변경사항을 처리 중...
(1/3) 업그레이드 중 msys2-runtime                                                                              [#################################################################] 100%
(2/3) 업그레이드 중 mintty                                                                                     [#################################################################] 100%
(3/3) 업그레이드 중 pacman-mirrors                                                                             [#################################################################] 100%
```

- -S --sync 옵션 설명
    - -y, --refresh : 서버로부터 패키지 데이터베이스를 다운로드
    - -u, --sysupgrade : 설치된 패키지를 업그레이드

설치가 끝나고 y를 누르면 터미널이 닫히는데 다시 실행시켜 남은 설치를 마저 진행한다.

```bash
$ pacman -Su
:: Starting core system upgrade...
 there is nothing to do
:: Starting full system upgrade...
resolving dependencies...
looking for conflicting packages...

Packages (10) bsdtar-3.5.0-1  curl-7.74.0-1  libcurl-7.74.0-1  libedit-20191231_3.1-2  libgnutls-3.7.0-1  liblz4-1.9.3-1
              libopenssl-1.1.1.i-1  libpcre2_8-10.36-1  libsqlite-3.34.0-1  openssl-1.1.1.i-1

Total Download Size:    6.10 MiB
Total Installed Size:  14.36 MiB
Net Upgrade Size:      -0.16 MiB

:: Proceed with installation? [Y/n] y
:: Retrieving packages...
중략..
:: Running post-transaction hooks...
(1/1) Updating the info directory file...
```

## 1.1 git 설치

```bash
$ pacman -S git
resolving dependencies...
looking for conflicting packages...

Packages (33) expat-2.2.10-1  heimdal-7.7.0-2  openssh-8.4p1-1  perl-Authen-SASL-2.16-2  perl-Clone-0.45-2
              perl-Convert-BinHex-1.125-1  perl-Encode-Locale-1.05-1  perl-Error-0.17029-1  perl-File-Listing-6.04-2
              perl-HTML-Parser-3.72-6  perl-HTML-Tagset-3.20-2  perl-HTTP-Cookies-6.08-1  perl-HTTP-Daemon-6.12-1
              perl-HTTP-Date-6.05-1  perl-HTTP-Message-6.25-2  perl-HTTP-Negotiate-6.01-2  perl-IO-HTML-1.001-1
              perl-IO-Socket-SSL-2.068-1  perl-IO-Stringy-2.113-1  perl-LWP-MediaTypes-6.04-1  perl-MIME-tools-5.509-1
              perl-MailTools-2.21-1  perl-Net-HTTP-6.19-1  perl-Net-SMTP-SSL-1.04-1  perl-Net-SSLeay-1.89_01-3
              perl-TermReadKey-2.38-2  perl-TimeDate-2.33-1  perl-Try-Tiny-0.30-1  perl-URI-1.76-1  perl-WWW-RobotRules-6.02-2
              perl-libwww-6.46-1  vim-8.2.1895-1  git-2.29.2-1

Total Download Size:   15.98 MiB
Total Installed Size:  86.28 MiB

:: Proceed with installation? [Y/n] y
:: Retrieving packages...
생략..
```

# 2. FFMPEG for NVCODEC

NVIDIA GPU 가속기능을 지원하는 ffmpeg을 빌드하려면 다음과 같은 것들이 필요하다.

- ffnvcodec
- ffmpeg

```bash
$ git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
...
$ git clone https://git.ffmpeg.org/ffmpeg.git
...
```

CUDA에 있는 모든 헤더파일과 라이브러리를 nv_sdk폴더에 복사한다. (CUDA 버전은 사용자 환경에 맞게 수정)

- C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.0\include\*
- C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.0\lib\x64\*

x64 Native Tools Command Prompt for VS 2019를 실행한다(버전은 설치되어 있는 비주얼스튜디오에 따라 다를 수 있음). msys2가 설치된 경로로 이동하여 *ming64.exe*가 있는지 확인한 후 실행한다.

```bash
> cd c:\msys64
> ls -al mingw64.exe
> mingw64.exe
```

MinGW64 환경에서 다음과 같이 필요한 패키지를 설치해야 한다.

```bash
$ pacman -S diffutils make pkg-config yasm
resolving dependencies...
looking for conflicting packages...

Packages (4) diffutils-3.7-1  make-4.3-1  pkg-config-0.29.2-4  yasm-1.3.0-2

Total Download Size:   1.28 MiB
Total Installed Size:  5.23 MiB

:: Proceed with installation? [Y/n] y
생략... 
```

다음과 같이 PATH 환경변수를 수정 후 export 해준다. (경로에 드라이브명 및 슬래쉬 방향 유의)

```bash
export PATH="/C/Program Files (x86)/Microsoft Visual Studio/2019/Community/VC/Tools/MSVC/14.24.28314/bin/Hostx64/x64/":$PATH
export PATH="/C/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.16.27023/bin/Hostx64/x64/":$PATH
export PATH="/C/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v11.0/bin/":$PATH
```

nv-codec-headers 폴더로 이동하여 빌드 및 설치한다.

```bash
$ cd nv-codec-headers
$ make install PREFIX=/usr
sed 's#@@PREFIX@@#C:/msys64/usr#' ffnvcodec.pc.in > ffnvcodec.pc
install -m 0755 -d '/usr/include/ffnvcodec'
install -m 0644 include/ffnvcodec/*.h '/usr/include/ffnvcodec'
install -m 0755 -d '/usr/lib/pkgconfig'
install -m 0644 ffnvcodec.pc '/usr/lib/pkgconfig'
```

FFmpeg 소스가 있는 곳으로 이동하여 configure를 수행한다.

- --enable-cuda-sdk 대신 --enable-cuda-nvcc 를 사용한다.
- 가이드 문서에는 --arch=x86_64가 없는데 꼭 넣어야 빌드가 된다.

```bash
$ cd ../ffmpeg/
$ ./configure --enable-nonfree --disable-shared --enable-cuda-nvcc --enable-libnpp --toolchain=msvc --extra-cflags=-I../nv_sdk --extra-ldflags=-libpath:../nv_sdk --arch=x86_64
```

그 다음 아무 문제 없으면 빌드를 진행한다.

```jsx
$ make -j
```

# 트러블슈팅

## a. PATH 문제

./configure 도중 다음과 같은 에러를 만났다.

```bash
$ ./configure --enable-nonfree --disable-shared --enable-cuda-sdk --enable-libnpp --toolchain=msvc --extra-cflags=-I../nv_sdk --extra-ldflags=-libpath:../nv_sdk
cl.exe is unable to create an executable file.
If cl.exe is a cross-compiler, use the --enable-cross-compile option.
Only do this if you know what cross compiling means.
C compiler test failed.

If you think configure made a mistake, make sure you are using the latest
version from Git.  If the latest version fails, report the problem to the
ffmpeg-user@ffmpeg.org mailing list or IRC #ffmpeg on irc.freenode.net.
Include the log file "ffbuild/config.log" produced by configure as this will help
solve the problem.
```

자세한 에러 내용은 ffbuild/config.log 를 열어서 확인할 수 있다.

다음 PATH 설정을 첫번째 항목에서 두번째 항목의 값으로 변경해주었다. 아마도 x64 Native Tools Command Prompt가 VS2019 버전인데 PATH는 이전 버전으로 설정해서 그런 것으로 의심된다.

- export PATH="/C/Program Files (x86)/Microsoft Visual Studio 12.0/VC/bin/amd64/":$PATH
- export PATH="/C/Program Files (x86)/Microsoft Visual Studio/2019/Community/VC/Tools/MSVC/14.24.28314/bin/Hostx64/x64/":$PATH

## b. --arch=x86_64 문제

```bash
$ ./configure --enable-nonfree --disable-shared --enable-cuda-nvcc --enable-libnpp --toolchain=msvc --extra-cflags=-I../nv_sdk --extra-ldflags=-libpath:../nv_sdk
ERROR: failed checking for nvcc.

If you think configure made a mistake, make sure you are using the latest
version from Git.  If the latest version fails, report the problem to the
ffmpeg-user@ffmpeg.org mailing list or IRC #ffmpeg on irc.freenode.net.
Include the log file "ffbuild/config.log" produced by configure as this will help
solve the problem.
```

끝에 --arch=x86_64 를 추가해 줘야 한다. ([https://sacoku.github.io/blog/2019/07/29/FFMPEG-윈도우빌드/](https://sacoku.github.io/blog/2019/07/29/FFMPEG-%EC%9C%88%EB%8F%84%EC%9A%B0%EB%B9%8C%EB%93%9C/) 내용 보다가 우연히 발견)

## c. 빌드 에러 C1091

나는 빌드 과정 중에 다음과 같은 C1091 에러가 발생했다.

```bash
vf_scale_cuda_bicubic.ptx.c
libavfilter/vf_scale_cuda_bicubic.ptx.c(1940): fatal error C1091: 컴파일러 한계 : 문자열 길이가 65535바이트를 초과합니다.
make: *** [ffbuild/common.mak:67: libavfilter/vf_scale_cuda_bicubic.ptx.o] Error 2
```

MSVC로 빌드할 때 CUDA 쪽 코드가 깨지는 문제가 있는데, ffmpeg trac에 다음 링크의 패치로 해결 할 수 있다.

- [https://trac.ffmpeg.org/ticket/9019](https://trac.ffmpeg.org/ticket/9019)
- make clean 후 다시 빌드해야 한다.

## d. 빌드에러 - compute_30

다음과 같이 지원되지 않는 gpu 아키텍쳐라며 nvcc 에러가 났다.

```bash
$ make
AR      libavdevice/libavdevice.a
NVCC    libavfilter/vf_overlay_cuda.ptx
nvcc fatal   : Unsupported gpu architecture 'compute_30'
make: *** [ffbuild/common.mak:102: libavfilter/vf_overlay_cuda.ptx] Error 1
```

configure 내에 compute_30은 compute_61로 sm_30은 sm_61로 변경해준다.

모델별 Compute Capability

- GTX1080 은 6.1
- P5000 또는 RTX8000은 7.5
