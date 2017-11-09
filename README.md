# elikos_jetson

Code destiné à être sur la plateforme Jetson d'NVIDIA.

<!-- TOC -->

- [elikos_jetson](#elikos_jetson)
    - [Setup du Jetson](#setup-du-jetson)
        - [Flash](#flash)
            - [Configuration d'installation](#configuration-dinstallation)
            - [Connection réseau pour l'installation](#connection-réseau-pour-linstallation)
        - [OS](#os)
            - [Changement de mot de passe des utilisateurs](#changement-de-mot-de-passe-des-utilisateurs)
            - [Mettre a jour les sources et paquets préinstallés](#mettre-a-jour-les-sources-et-paquets-préinstallés)
            - [Overclocking](#overclocking)
            - [Vidange du OS](#vidange-du-os)
        - [Installation des dépendances](#installation-des-dépendances)
            - [ROS](#ros)
                - [Workspaces](#workspaces)
            - [`librealsense`](#librealsense)
            - [`darknet_ros`](#darknet_ros)
                - [Téléchargement de la configuration de darknet](#téléchargement-de-la-configuration-de-darknet)

<!-- /TOC -->

## Setup du Jetson

### Flash

Pour installer le système d'exploitation sur le Jetson, il faut télécharger l'installateur du `Jetpack 3.1`.

<https://developer.nvidia.com/embedded/dlc/jetpack-l4t-3_1>

Ensuite, il faut que suivre les étapes de l'installateur.

#### Configuration d'installation

![Config](https://github.com/elikos/elikos_jetson/images/install_config.png "Install config")

#### Connection réseau pour l'installation

Pour un temps d'installation optimal, nous recommandons de connecter le Jetson par **ethernet** à un routeur ayant un accès internet. L'ordinateur d'installation devra être connecté à ce même routeur en wifi ou par ethernet si possible.

### OS

#### Changement de mot de passe des utilisateurs

Pour des raisons de sécurité **apparentes** il est **impératif** de changer les mots de passe par défaut du Jetson.

Le mot de passe par défaut du compte `nvidia` est `nvidia`

```bash
sudo passwd nvidia
sudo passwd ubuntu
```

#### Mettre a jour les sources et paquets préinstallés

ROS ne pourra pas être installé si les paquets ne sont pas mis à jour.

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

#### Overclocking

Pour overclocker le Jetson au démarage, créer le fichier `/etc/rc.local` et y écrire:

```bash
#!/bin/sh -e

(sleep 60 && /home/nvidia/jetson_clocks.sh) &

exit 0
```

Ensuite, éxécuter les commandes suivantes:

```bash
sudo chown root /etc/rc.local
sudo chmod 755 /etc/rc.local
sudo /etc/init.d/rc.local start
```

#### Vidange du OS

Puisque l'espace disque est limité sur le Jetson, il est falcultatif mais pertinent de supprimer certains éléments supperflus.

- Dossiers temporaire d'installation (situés dans le dossier `/home/nvidia/`):
  - Cuda
  - OpenCV
  - cuDNN
- Les sources des «repo» locaux dans `/etc/apt/sources.list.d/`
- Les logiciels innutiles:
  - Thunderbird
  - Suite LibreOffice
  - Transmission
  - **VIM**

### Installation des dépendances

Afin de faire fonctionner le TX1 avec `elikos_jetson`, les programmes suivant doivent être installés

- [ROS](#ROS)
- [LibRealSense](#LibRealSense)
- darknet_ros

#### ROS

Suivre les instructions sur le site de ROS

> <http://wiki.ros.org/kinetic/Installation/Ubuntu>

**À noter: Nous voulons installer la version `ros-kinetic-ros-base`**

##### Workspaces

Le Jetson aura une architecture à double workspace

- `~/jetson_ws`
  - Contiendra les différentes nodes conçues par Élikos qui seront exécutés sur le Jetson
- `~/jetson_lib_ws`.
  - Contiendra les pilotes et autres librairies requises au bon fonctionnement des nodes de `~/jetson_ws`.

#### `librealsense`

Le paquet `ros-kinetic-librealsense` est **incompatible** avec le Jetson.
Pour y remédier, nous devons compiler une version spécifique de `librealsense` et de `realsense-camera`.

Premièrement, installer `librealsense` sur le système:

> github.com/jetsonhacks a déjà fait un script pour automatiser l'installation

```bash
cd ~
git clone https://github.com/jetsonhacks/installLibrealsenseTX1.git
cd installLibrealsenseTX1
git reset --hard e9f4f92abfdcbab822d152db72f7eeba903f6332
# À noter que le commit auquel nous resettons est le dernier commit testé
chmod +x installLibrealsense.sh
./installLibrealsense.sh
```

Ensuite, installer `librealsense` dans la workspace ROS
**L'installation de `librealsense` se fera dans `~/jetson_lib_ws`.**

```bash
cd ~/jetson_lib_ws/src
git clone https://github.com/IntelRealSense/librealsense.git
cd librealsense
git checkout v1.12.1
wget https://raw.githubusercontent.com/jetsonhacks/installLibrealsenseTX1/master/patches/uvc-v4l2.patch
patch -p1 -i uvc-v4l2.patch
# Patch appliquée pour la reconnaissance du module uvc du kernel
```

Puis cloner `realsense-camera`:

```bash
cd ~/jetson_lib_ws/src
git clone https://github.com/intel-ros/realsense.git
cd realsense
git checkout 1.8.0
cd ../..
sudo rosdep -y install --from-paths src --ignore-src --rosdistro kinetic
catkin_make
```

#### `darknet_ros`

Pour lier *darknet* à *ROS*, nous utiliserons [darknet_ros](https://github.com/leggedrobotics/darknet_ros).

**L'installation de `darknet_ros` se fera dans `~/jetson_lib_ws`.**

```bash
cd ~/jetson_lib_ws/src
git clone https://github.com/leggedrobotics/darknet_ros.git
cd darknet_ros
git reset --hard 0c6531a00ebd1a4375c57eb84077f9c18ab963f7
# À noter que le commit auquel nous resettons est le dernier commit testé
```

##### Téléchargement de la configuration de darknet

**TODO**