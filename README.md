# elikos_jetson

Code destiné à être sur la plateforme Jetson d'NVIDIA.

## *WARNING*

NE PAS TOUCHER AUX PINS GPIO LORSQUE LE JETSON EST EN MARCHE. CELA FAIT COURT-CIRCUITER LE DEV BOARD!

Devine qui a découvert ça _(hint: il a aussi un problème avec les batteries)_...

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
    - [Installation des programmes Elikos](#installation-des-programmes-elikos)
        - [jetson_ws](#jetson_ws)
        - [.bashrc](#bashrc)
- [Entrainement du réseau de neurone](#entrainement-du-réseau-de-neurone)
    - [Compilation de darknet](#compilation-de-darknet)
    - [Création d'un dataset](#création-dun-dataset)
    - [Entrainement](#entrainement)

<!-- /TOC -->

## Setup du Jetson

### Flash

Pour installer le système d'exploitation sur le Jetson, il faut télécharger l'installateur du `Jetpack 3.2`.

<https://developer.nvidia.com/embedded/dlc/jetpack-l4t-3_2>

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
_Il est normal d'avoir certaines erreurs de signature de source apt pour les repos locaux. Simplement les ignorer!_

#### Overclocking

Pour overclocker le Jetson au démarage, créer le fichier `/etc/rc.local` et y ajouter:

```bash
(sleep 60 && /home/nvidia/jetson_clocks.sh) &
```

Ensuite, éxécuter les commandes suivantes:

```bash
sudo /etc/init.d/rc.local start
```

#### Vidange du OS

Puisque l'espace disque est limité sur le Jetson, il est falcultatif mais pertinent de supprimer certains éléments supperflus comme certains logiciels inutiles:
  - Thunderbird
  - Suite LibreOffice
  - Transmission

```bash
sudo apt remove chromium-browser thunderbird libreoffice-* transmission-*
sudo apt autoremove
```

### Installation des dépendances

Afin de faire fonctionner le TX* avec `elikos_jetson`, les programmes suivant doivent être installés

- [ROS](#ROS)
- [LibRealSense](#LibRealSense)
- darknet_ros

#### ROS

Suivre les instructions sur le site de ROS

> <http://wiki.ros.org/kinetic/Installation/Ubuntu>

Nous voulons aussi utiliser les `Catkin Command Line Tools`:
```bash
sudo apt install python-catkin-tools
```

**À noter: Nous voulons installer la version `ros-kinetic-desktop`**

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
git clone https://github.com/jetsonhacks/installLibrealsenseTX2.git
cd installLibrealsenseTX2
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
```

Puis build le tout:

```bash
catkin_make -DCMAKE_BUILD_TYPE=Release
```
### Installation des dépendances

#### `jetson_ws`
Installons maintenant le code qui sera éxécuté sur le Jetson.

```bash
cd ~
mkdir -p jetson_ws/src
cd jetson_ws/src
git clone --recurse-submodules git@github.com:elikos/elikos_jetson.git
cd ..
catkin build
```

#### `.bashrc`
Pour éviter une prise de panique soudaine lors du démarage du Jetson, nous allons ajouter les _workspaces_ au `.bashrc` pour une inclusion automatique:

Ajouter les lignes suivantes au `~/.bashrc`:
```bash
source ~/jetson_lib_ws/devel/setup.bash
source ~/jetson_ws/devel/setup.bash
```

Puis faire la commande:
```bash
source ~/.bashrc
```

### Entrainement du réseau de neurone

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
