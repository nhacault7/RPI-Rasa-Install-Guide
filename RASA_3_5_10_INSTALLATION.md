# Installing Rasa 3.5.10 on Raspberry Pi 4 w/ Raspberry Pi OS 64-bit (Debian Bullseye)

## Installing OS

To get started install the [Raspberry Pi OS Imager](https://www.raspberrypi.com/software/)

After installing the imager select CHOOSE OS > Raspberry Pi OS (other) > Raspberry Pi OS (64-bit)

- **_NOTE: I have also tested that Rasa works with Ubuntu Desktop 23.04 (64-bit) on the Raspberry Pi as well so feel free to choose that if you prefer Ubuntu_**

Then select your storage device and write the image to it

Once you have finished writing the OS attach your storage device to your Raspberry Pi, power it on and go through the installation steps and restart the Pi when prompted

## Adding Debian Bookworm to Debian Bullseye SourceList -- Skip this step if you are using Ubuntu 23.04 --

To get the proper updated versions of some packages needed by Rasa's package requirements we need to upgrade from Debian Bullseye to Bookworm

First open the source lists file from the terminal

```.env
sudo nano /etc/apt/sources.list
```

Then replace every instance of "bullseye" with "bookworm" in the first three lines. It should look as follows

```.env
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb http://security.debian.org/debian-secuirty bookwork-security main contrib non-free non-free-firmware
deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
```

Press Ctrl + s then Crtl + x to save and exit

## Updating OS and Installing Required Packages

Once in the desktop open the terminal to update Debian and install the required packages. Select y to any prompts

```.env
sudo apt-get update
sudo apt-get upgrade -y 
sudo apt-get dist-upgrade -y
```
Now reboot the system and you should be in Debian Bookworm. -- Ignore if using ubuntu --

- **_NOTE: The default desktop environment may not look or work properly, if you'd like to change it follow this [tutorial](https://www.makeuseof.com/how-to-install-a-desktop-on-raspberry-pi-os-lite/), I would recommend going with the gnome desktop option_**

Now lets install the required packages

```.env
sudo apt install -y build-essential git curl librdkafka-dev
```

## Installing Mamba

Next we will install Mamba, this is a newer version of Conda that runs much faster and will help with installing all our Python packages

```.env
cd ~/Documents
wget https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-pypy3-Linux-aarch64.sh
bash Mambaforge-pypy3-Linux-aarch64.sh
```

This will start the installation process. Go through the installer saying yes to all the options.

- **_NOTE: this will cause Mamba to always activate when opening the terminal. To stop this you can issue the command "conda config --set auto_activate_base false"_**

Now close and reopen the terminal and you should see (base) added to the terminal line. 

Update mamba by running these commands and selecting "y" to any prompts


```.env
mamba update mamba 
mamba update --all
```

- **_NOTE: if you get an error after performing mamba update --all this should be okay, just rerun both commands and ensure everything is up to date_**

## Setting Up A Virtual Environment

Now that Mamba is running and up to date we need to create a python environment for the project. This allows us to install whatever Python packages we want without clutering the main environment

We will be using python 3.10, selecting y to any prompts

```.env
mamba create -n RASA Python==3.10
mamba activate RASA
python -m pip uninstall pip
python -m ensurepip
python -m pip install -U pip
```

## Installing Bazelisk

Next we install Bazelisk, this is a launcher for Bazel that will help with creating the Tensorflow Addons and Text Packages from source

```.env
cd ~/Documents
wget https://github.com/bazelbuild/bazelisk/releases/download/v1.17.0/bazelisk-linux-arm64
chmod +x bazelisk-linux-arm64
sudo mv bazelisk-linux-amd64 /usr/local/bin/bazel
```
Now run the version command twice. 
- The first time will download the release to your machine. 
- The second time should confirm the bazel version

```.env
bazel --version
```

## Installing Tensorflow

Now that we have all our tools setup and ready to go lets install Tensorflow version 2.11.0

```.env
pip install tensorflow==2.11.0
```

## Installing Tensorflow Addons

Now we have to build Tensorflow Addons version 0.19 which is compatible with Tensorflow 2.11

```.env
cd ~/Documents
git clone https://github.com/tensorflow/addons.git
cd addons
git checkout r0.19
python ./configure.py
bazel build --enable_runfiles build_pip_pkg
bazel-bin/build_pip_pkg artifacts
pip install artifacts/tensorflow_addons-0.19.0-*.whl
```

## Installing Tensorflow Text

Now for the final requirement we build Tensorflow Text. This one will take a while so sit back, relax and make sure your Pi has a healthy power supply

```.env
cd ~/Downloads
git clone https://github.com/tensorflow/text
cd text 
git checkout 2.11
source oss_scripts/configure.sh
bazel build --enable_runfiles oss_scripts/pip_package/build_pip_package
./bazel-bin/oss_scripts/pip_package/build_pip_package artifacts
pip install artifacts/tensorflow_text-*.whl
```

## Installing Rasa

And now for the moment of truth, we install Rasa

```.env
cd ~/Documents
mkdir Rasa
cd Rasa
pip install "rasa==3.5.10"
rasa init
```

Hopefully this helped, and happy training!