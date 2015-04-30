该MFS - 媒体文件夹同步GitHub的页面
=======================================

对于改善HTML5的支持，大多数服务器需要保持2个或更多copys媒体文件在不同的格式。该脚本监视特定的文件夹，使用Linux的inotify的变化，并使用FFMPEG转换视频/音频格式的所有希望。
<BR/>
这个版本在这里保留所有submited音频的OGG和MP3版（OGG或MP3格式）。
要修改或改善这一点，编辑settings.py和更新文件类型列表。
里面的文件类型列表中的每个百科指的是一群媒体元素与它的ffmpeg的编解码器。
比方说，你需要有，用于音频，格式MP3和OGG以及视频WEBM和MP4。你会做这样的
```
filetypes = [
    {
        '.ogg': {'acodec':'libvorbis'},
        '.mp3': {'acodec':'libmp3lame'},
    },
    {
        '.mp4': {'vcodec':'mpeg4','acodec':'libvo_aacenc'},
        '.webm': {'vcodec':'libvpx','acodec':'libvorbis'},
    },
]
```
使用它，你只需要调用脚本并要监视的文件夹后。这个脚本会监视所有文件夹递归，但不会跟随符号连接。
```
./media_folder_sync.py /var/www/media
```

要在生产中使用，一个类似的工具 <a href="http://supervisord.org/">Supervisor</a> is highly recommended.

提示与FFMPEG
----------------
FFmpeg是一个真棒工具，视频/音频转换，但它也通常分布，主要是因为在美国和法律上的原因的一些其他国家没有他的全部能力。
要知道你的容器支持的ffmpeg的安装目录，输入：

```
ffmpeg -format
```
And for a list of codecs:
```
ffmpeg -codecs
```
A simple tip on how to get a very complete ffmpeg on Debian:
```
#Remove your existing installation of ffmpeg
apt-get remove --purge ffmpeg 

#This ubuntu mirrors has some codec dev packages, you can also remove it after the apt-get install if you prefer
echo 'deb http://archive.ubuntu.com/ubuntu natty main restricted universe multiverse' >/etc/apt/sources.list.d/ubuntu.list
apt-get update
apt-get install make automake g++ bzip2 python unzip patch subversion ruby build-essential git-core checkinstall yasm texi2html libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libsdl1.2-dev libtheora-dev libvdpau-dev libvorbis-dev libvpx-dev libx11-dev libxfixes-dev libxvidcore-dev zlib1g-dev

git clone git://git.videolan.org/x264.git
cd x264

#This disable-asm is only important if, without it, the configure complains about an old asm compiler.
./configure --enable-shared --disable-asm 
make
make install
cd ..

wget http://webm.googlecode.com/files/libvpx-v0.9.7-p1.tar.bz2
tar -xjf libvpx-v0.9.7-p1.tar.bz2
cd libvpx-v0.9.7-p1
./configure
make
make install
cd ..
ldconfig

git clone git://source.ffmpeg.org/ffmpeg
cd ffmpeg
./configure --enable-gpl --enable-version3 --enable-nonfree --enable-postproc --enable-libfaac --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libxvid --enable-x11grab
make
make install
cd ..

ldconfig
```

Deploy example on Debian with Supervisor
----------------------------------------
Here is a deploy example of media-sync on debian, considering you already have ffmpeg installed (see above for a ffmpeg tip)
```
apt-get install python-virtualenv python-pyinotify
cd /opt
mkdir media-folder-sync
cd media-folder-sync
virtualenv vpython-mfs
. vpython-mfs/bin/activate
pip install supervisor
git clone https://github.com/thor27/media-folder-sync.git
echo_supervisord_conf > /etc/supervisord.conf
vim /etc/supervisord.conf
```
At the end of the file, add:
```
[program:mfs]
command=/opt/media-folder-sync/vpython-mfs/bin/python /opt/media-folder-sync/media-folder-sync/media_folder_sync.py /var/www/files/ -d /opt/media-folder-sync/database.db
```
Replace the path /var/www/files/ with the folder you want to monitor with MFS.
Now, create links for supervisor executables:
```
ln -s `pwd`/vpython-mfs/bin/supervisorctl /usr/local/bin/
ln -s `pwd`/vpython-mfs/bin/supervisord /usr/local/bin/
deactivate
```
So, to start supervisor just run supervisord
```
supervisord
```
To check if media-folder-sync is running, or to start/stop it, just use supervisorctl
```
supervisorctl status mfs
supervisorctl start mfs
supervisorctl stop mfs
supervisorctl restart mfs
```
To stop supervisor process (and also media-folder-sync):
```
supervisorctl shutdown
```
More info on how to use or configure supervisor, go to the official website: <a href="http://supervisord.org/">http://supervisord.org/</a>
