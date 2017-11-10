# elikos_jetson

Code destiné à être sur la plateforme Jetson d'NVIDIA.

<!-- TOC -->

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
        - [librealsense](#librealsense)
        - [darknet_ros](#darknet_ros)
            - [Téléchargement de la configuration de darknet](#téléchargement-de-la-configuration-de-darknet)
- [Entrainement du réseau de neurone](#entrainement-du-réseau-de-neurone)
    - [Compilation de darknet](#compilation-de-darknet)
    - [Création d'un dataset](#création-dun-dataset)
    - [Entrainement](#entrainement)

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

#### librealsense

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

#### darknet_ros

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

## Entrainement du réseau de neurone

Pour que le réseau de neurone fonctionne, il faut d'abord l'entrainer avec les images à détecter

### Compilation de darknet

Darknet peut être cloné directement à partir du repo GitHub.

```bash
git clone https://github.com/pjreddie/darknet.git
cd darknet
```

Pour une raison mystérieuse, cette dernière version de Darknet peut être incompatible avec certaines cartes grahique. Pour régler ce problème, il est possible d'utiliser la version d'AlexeyAB. Cette version ne doit **pas** être utilisé sur le Jetson.

```bash
git clone https://github.com/AlexeyAB/darknet
cd darknet
```

Avant de compiler darknet, il faut télécharger et installer CUDA 8.
<https://developer.nvidia.com/cuda-80-ga2-download-archive>

Il faut ensuite modifier le Makefile pour activer Cuda en modifiant le flag `GPU=1` (et `OPENCV=1` si désiré et installé).

```bash
make
```

### Création d'un dataset

Afin de créer le dataset, il est possible d'utiliser l'utilitaire d'AlexayAB.

```bash
git clone https://github.com/AlexeyAB/Yolo_mark
```

Suivre les étapes décrites sur la page github de Yolo_mark <https://github.com/AlexeyAB/Yolo_mark>.

### Entrainement

Pour entrainer le dataset produit par Yolo_mark, copier le dossier `x64` dans le dossier `darknet`. Ensuite, télécharger les *weights* d'entrainement du site de darknet dans le dossier `darknet/x64/Release`. Finalement, exécuter la commande suivante.

```bash
./darknet detector train x64/Release/data/obj.data x64/Release/yolo-obj.cfg x64/Release/darknet19_448.conv.23
```

À chaque 100 itérations, darknet sauvegarde les résultats dans le dossier `darknet/backup`.
**Note:** Il est possible de diminuer le paramètre `batch` et augmenter le paramètre `subdivisions` dans le fichier yolo-obj.cfg si il n'y a pas assez de mémoire graphique.