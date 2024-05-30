# Building Robot Pi OS

## Build Envrionment
1. Build machine: [Ubuntu 22.04](https://releases.ubuntu.com/jammy/)
2. Repo configuration tool: [kas](https://kas.readthedocs.io/en/latest/)

## Ubuntu prerequisites:
```
sudo apt-get install gawk wget git diffstat unzip gcc-multilib \
        build-essential chrpath socat cpio python3-pip python3-pexpect \
        xz-utils debianutils iputils-ping \
        python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev xterm \
        g++-multilib locales lsb-release python3-distutils time \
        liblz4-tool zstd file
sudo locale-gen en_US.utf8
```

## Installing kas

To create a Python virtual environment to install kas run these commands:
```
python3 -m venv venv
source venv/bin/activate
pip3 install kas
```

## Clone the robotpi-os-setup project
Create top-level project directory such `robotpi`. 
```
cd robotpi
git clone git@github.com:whni/robotpi-os-setup.git
```
The directory hierarchy should looks like:
```
robotpi
|-- robotpi-os-setup
```

## Build the image
There are multiple kas configration files under robotpi-os-setup directory:
1. `robotpi-os-setup/robotpi-os-qemux86-64.yml`
2. `robotpi-os-setup/robotpi-raspberrypi4-64.yml`
3. `robotpi-os-setup/robotpi-rb5-64.yml`
```
kas checkout robotpi-os-setup/robotpi-os-xxxx.yml
```
This will generate project configuration files such as local.conf and bblayers.conf
under build/conf and download all required layers under layers directory:
```
robotpi
|-- robotpi-os-setup
|-- layers
    |-- ...
    |-- meta-robotpi
|-- build
    |-- ...
    |-- conf
        |-- local.conf
        |-- bblayers.conf
```

Then move to the top directory and use bitbake to generate the target image:
```
source layers/meta-robotpi/robotpi-init-build-env
bitbake robotpi-image-turtlebot3-core
```

The above kas and bitbake steps can be also achieved by one-line command (NOT recommended):
```
kas build robotpi-os-setup/robotpi-os-xxxx.yml
```

## Writing the image
### Raspberry Pi
The raspberry pi image can be found as:
```
build/BUILD-${DISTRO}-xxxx/deploy/images/raspberrypi4-64/robotpi-image-turtlebot3-core-humble-raspberrypi4-64-${timestamp}.rootfs.wic.bz2
```
If using [Balena Etcher](https://etcher.balena.io/), you may provide it with
this file directly.

### QEMU
Run the following command on your host machine
```
runqemu qemux86-64
runqemu qemux86-64 qemuparams="-cpu IvyBridge" nographic
```

## Development on meta-robotpi layer repo
under `robotpi-os-setup/robotpi/robotpi.yml`, we can see the repo setup of robotpi layer container,
which is currently tracking the remote master branch.

Once we have finished the kas checkout process, [meta-robotpi](https://github.com/whni/meta-robotpi.git)
remote repo will be pulled into `layers/meta-robotpi` directory, where any local changes can be made for
feature development and testing. However, `layers/meta-robotpi` is just a temporary git checkout repo
for build purpose. For safety, you need to move the verified changes over to your own checkout copy of
`layers/meta-robotpi` with git ssh link and push it to the remote branch:
```
git clone git@github.com:whni/meta-robotpi.git -b master
# move your local changes from layers/meta-robotpi to here
git add --all
git commit -m "your commit message"
git push
```

> [!WARNING]
> Please note that whenever you run kas checkout or build command, all layer repos under `layers`
> directory will be sync to the upstream version based on kas yml. All local commits will be abandoned,
> while local uncommitted changes can be retained. Please remember to save your changes before running
> any kas operations

