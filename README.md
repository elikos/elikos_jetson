# elikos_jetson
Code destiné à être sur la plateforme Jetson d'NVIDIA.

<!-- TOC -->

- [elikos_jetson](#elikos_jetson)
    - [Setup du Jetson](#setup-du-jetson)
        - [OS](#os)
        - [Installation des dépendances](#installation-des-dépendances)
            - [ROS](#ros)
        - [LibRealSense](#librealsense)
    - [Réseau de neuronne](#réseau-de-neuronne)

<!-- /TOC -->

## Setup du Jetson
### OS
- `sudo passwd nvidia`
- `sudo apt update;sudo apt upgrade`
- Overclocking
- PC decrapifier
### Installation des dépendances
Afin de faire fonctionner le TX1 avec `elikos_jetson`, les programmes suivant doivent être installés
- [ROS](#ROS)
- [LibRealSense](#LibRealSense)
- darknet_ros

#### ROS
Suivre les instructions sur le site de ROS

> http://wiki.ros.org/kinetic/Installation/Ubuntu

**À noter: Nous voulons installer la version `ros-kinetic-ros-base`**

### LibRealSense
Le paquet `ros-kinetic-librealsense` est **incompatible** avec le Jetson.
Pour y remédier, nous devons compiler une version spécifique de `librealsense` et de `realsense-camera`.

Premièrement, installer `librealsense` sur le système:

> github.com/jetsonhacks a déjà fait un script pour automatiser l'installation

```bash
cd ~
git clone https://github.com/jetsonhacks/installLibrealsenseTX1.git
cd installLibrealsenseTX1
git reset --hard e9f4f92abfdcbab822d152db72f7eeba903f6332
chmod +x installLibrealsense.sh
./installLibrealsense.sh
```

*À noter que le commit auquel nous resettons est le dernier commit testé*

Ensuite, installer `librealsense` dans la workspace ROS
```bash
cd jetson_ws/src
git clone https://github.com/IntelRealSense/librealsense.git
cd librealsense
git checkout v1.12.1
wget https://raw.githubusercontent.com/jetsonhacks/installLibrealsenseTX1/master/patches/uvc-v4l2.patch
patch -p1 -i uvc-v4l2.patch
# Patch appliquée pour la reconnaissance du module uvc du kernel
```

Puis cloner `realsense-camera`:

```bash
git clone https://github.com/intel-ros/realsense.git
cd realsense
git checkout 1.8.0
cd ../..
sudo rosdep -y install --from-paths src --ignore-src --rosdistro kinetic
catkin_make
```

## Réseau de neuronne