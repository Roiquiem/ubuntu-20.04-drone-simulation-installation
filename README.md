# ubuntu-20.04-drone-simulation-installation
This repository shows how to install full drone simulation on ubuntu 20.04. This includes: OpenCV 4.7.0, MAVLink, QGroundControl and PX4-Autopilot

## Table of Contents
* [Installation](#installation)
* [How to Run](#how-to-run-qgc-and-gazebo-classic-with-drone-model-with-camera)

## Installation
### OpenCV 4.7.0
Source: [QEngineering](https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/blob/main/OpenCV-4-7-0.sh)<br>
_made changes to fit non-Jetson-Nano machine._
```bash
set -e

cd ~
sudo sh -c "echo '/usr/local/cuda/lib64' >> /etc/ld.so.conf.d/nvidia-tegra.conf"
sudo ldconfig # will probably return that no file exists - ignore and continue

# install the dependencies - make sure that all the lines are executed
sudo apt-get install -y build-essential cmake git unzip pkg-config zlib1g-dev
sudo apt-get install -y libjpeg-dev libjpeg8-dev libjpeg-turbo8-dev libpng-dev libtiff-dev
sudo apt-get install -y libavcodec-dev libavformat-dev libswscale-dev libglew-dev
sudo apt-get install -y libgtk2.0-dev libgtk-3-dev libcanberra-gtk*
sudo apt-get install -y python3-dev python3-numpy python3-pip
sudo apt-get install -y libxvidcore-dev libx264-dev libgtk-3-dev
sudo apt-get install -y libtbb2 libtbb-dev libdc1394-22-dev libxine2-dev
sudo apt-get install -y gstreamer1.0-tools libv4l-dev v4l-utils qv4l2 
sudo apt-get install -y libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev
sudo apt-get install -y libavresample-dev libvorbis-dev libxine2-dev libtesseract-dev
sudo apt-get install -y libfaac-dev libmp3lame-dev libtheora-dev libpostproc-dev
sudo apt-get install -y libopencore-amrnb-dev libopencore-amrwb-dev
sudo apt-get install -y libopenblas-dev libatlas-base-dev libblas-dev
sudo apt-get install -y liblapack-dev liblapacke-dev libeigen3-dev gfortran
sudo apt-get install -y libhdf5-dev protobuf-compiler
sudo apt-get install -y libprotobuf-dev libgoogle-glog-dev libgflags-dev

# remove old versions or previous builds
cd ~
sudo rm -rf opencv*

# download the latest version
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.7.0.zip 
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.7.0.zip 
# unpack
unzip opencv.zip 
unzip opencv_contrib.zip 
# some administration to make live easier later on
mv opencv-4.7.0 opencv
mv opencv_contrib-4.7.0 opencv_contrib
# clean up the zip files
rm opencv.zip
rm opencv_contrib.zip

# set install dir
cd ~/opencv
mkdir build
cd build

# run cmake
cmake -D CMAKE_BUILD_TYPE=RELEASE \ -D CMAKE_INSTALL_PREFIX=/usr \ -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \ -D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \ -D WITH_OPENCL=OFF \ -D WITH_QT=OFF \ -D WITH_OPENMP=ON \ -D BUILD_TIFF=ON \ -D WITH_FFMPEG=ON \ -D WITH_GSTREAMER=ON \ -D WITH_GTK=ON -D WITH_TBB=ON \ -D BUILD_TBB=ON \ -D BUILD_TESTS=OFF \ -D WITH_EIGEN=ON \ -D WITH_V4L=ON \ -D WITH_LIBV4L=ON \ -D OPENCV_ENABLE_NONFREE=ON \ -D INSTALL_C_EXAMPLES=OFF \ -D INSTALL_PYTHON_EXAMPLES=OFF \ -D PYTHON3_PACKAGES_PATH=/usr/lib/python3/dist-packages \ -D OPENCV_GENERATE_PKGCONFIG=ON \ -D BUILD_EXAMPLES=OFF ..

# run make
make -j #(add your number of jobs. e.g.: -j8 )

sudo rm -r /usr/include/opencv4/opencv2
sudo make install
sudo ldconfig

make clean
sudo apt-get update
```

### PX4-Autopilot (include Gazebo Classic)
```bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
# checkout in the latest release version
cd PX4-Autopilot
git checkout v1.13.3
git submodule update --init --recursive 
# install everything
cd ~
./PX4-Autopilot/Tools/setup/ubuntu.sh
```

### QGroundControl
```bash
sudo usermod -a -G dialout $USER
sudo apt-get remove modemmanager -y
sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
sudo apt install libqt5gui5 -y
sudo apt-get install -y libpulse-dev
```
Logout and Login, and then run:<br>
```bash
# download the appimage:
wget https://d176tv9ibo4jno.cloudfront.net/latest/QGroundControl.AppImage
chmod +x ./QGroundControl.AppImage
# to open the app:
./QGroundControl.AppImage
```

### Mavlink
```bash
cd ~

# Dependencies
sudo apt install python3-pip

# Clone mavlink into the home directory
git clone https://github.com/mavlink/mavlink.git --recursive

cd mavlink
python3 -m pip install -r pymavlink/requirements.txt
python3 -m pymavlink.tools.mavgen --lang=C++11 --wire-protocol=2.0 --output=generated/include/mavlink/v2.0 message_definitions/v1.0/common.xml

cmake -Bbuild -H. -DCMAKE_INSTALL_PREFIX=install -DMAVLINK_DIALECT=common -DMAVLINK_VERSION=2.0
cmake --build build --target install

#in your CMakeLists.txt make sure to have:
find_package(MAVLink REQUIRED)
target_link_libraries(my_program MAVLink::mavlink)

# go to the directory wehre your cpp file is located
cd ../myProgram

# make sure to specify the correct path to the mavlink/install directory, relative to the current location
cmake -Bbuild -H. -DCMAKE_PREFIX_PATH=../../mavlink/install
cd build
make
./myProgram
```

### How to Run QGC and Gazebo-Classic with drone model with camera

Open 2 Terminals
#### QGC Terminal
go to where the QGroundControl.AppImage is located (probably home directory), and run:
```bash
    ./QGroundControl.AppImage
```
QGC should open

#### Gazebo Terminal
run this:
```bash
    cd PX4-Autopilot
    make px4_sitl gazebo_typhoon_h480
``` 
Gazebo Classic with the drone model should open

#### Usage
Now you can takeoff in QGroundControl and see the drone takes-off in Gazebo. <br>
You should see the video in QGC.
