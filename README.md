# Fedora for machine learning guide

Welcome to the (non-official) Fedora post-installation guide!

This is a guide for setting up the Fedora 42 Workstation Edition for machine learning and general-purpose code development.

## Table of Contents

- [Hardware and software support](#hardware-and-software-support)
- [System configuration](#system-configuration)
  - [Configure](#configure)
  - [Update](#update)
  - [Repositories and package managers](#repositories-and-package-managers)
  - [Drivers](#drivers)
  - [Storage optimization](#storage-optimization)
  - [Tweaks](#tweaks)
  - [Clean DNF cache](#clean-dnf-cache)
- [Setting up development tools](#setting-up-development-tools)
  - [Monitoring](#monitoring)
    - [nmon](#nmon)
    - [Btop++](#btop)
  - [Git](#git)
  - [Virtualization](#virtualization)
  - [Containerization](#containerization)
    - [Docker](#docker)
    - [Vagrant](#vagrant)
  - [Compilers, interpreters and SDKs](#compilers-interpreters-and-sdks)
    - [C/C++](#cc)
    - [Fortran](#fortran)
    - [Python](#python)
    - [Java](#java)
    - [Rust](#rust)
    - [.NET](#net)
  - [Database engines and management systems](#database-engines-and-management-systems)
  - [CUDA Toolkit](#cuda-toolkit)
  - [IDEs and text-editors](#ides-and-text-editors)
- [Environments setup](#environments-setup)
  - [Conda environment for Tensorflow](#conda-environment-for-tensorflow)
- [Notes](#notes)

## Hardware and software support
  
*This guide currently aims for Fedora 37 Workstation Edition.*

For general code development, no hardware restrictions are imposed.

For machine learning development, a NVIDIA graphics card is expected for GPU-accelerated apps using the [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit), but it can be disregarded for ML models utilizing CPU processing only.

## System configuration

This is the system configuration page explaining the basic settings to look for after a fresh install.

### Configure

Configure the dnf package manager in `/etc/dnf/dnf.conf`, an example is given in [dnf.conf.example](https://github.com/mBelisarius/fedora-for-machine-learning/blob/main/dnf.conf.example) (see [dnf documentation](https://dnf.readthedocs.io/en/latest/conf_ref.html)).

Configure zram-generator in `/etc/systemd/zram-generator.conf`, an example is given in [zram-generator.conf.example](https://github.com/mBelisarius/fedora-for-machine-learning/blob/main/zram-generator.conf.example) (see [zram-generator documentation](https://github.com/systemd/zram-generator/blob/main/man/zram-generator.conf.md)).

Configure the swap parameters by adding the following variables in `/etc/sysctl.conf` (see [Linux VM Performance Tuning](https://lonesysadmin.net/tag/linux-vm-performance-tuning/)):
```
vm.vfs_cache_pressure=300
vm.swappiness=30
vm.dirty_background_ratio=7
vm.dirty_ratio=60
```

Change the hostname:
```
sudo hostnamectl set-hostname [hostname]
```

### Update

Update the entire system
```
sudo dnf update -y
sudo dnf install dnf-plugins-core -y
sudo dnf install kernel-devel kernel-headers -y
```

Update the firmware and UEFI (supported only by some manufacturers)
```
sudo fwupdmgr get-devices
sudo fwupdmgr refresh --force
sudo fwupdmgr get-updates
sudo fwupdmgr update -y
```

### Repositories and package managers

#### DNF management

Install the dnf-plugins-core package which provides the commands to manage your DNF repositories:
```
sudo dnf install dnf-plugins-core -y
```

#### Enable RPM Fusion repositories

Add extra Fedora Workstation repositories:
```
sudo dnf install fedora-workstation-repositories -y
```
    
Enable [RPM Fusion](https://rpmfusion.org/Configuration) to both free and non-free repository: 
```
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm -y
```

#### Enable Flatpak
    
Enable [Flathub](https://flatpak.org/setup/Fedora) repository:
```
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Configure [Flathub desktop integration](https://itsfoss.com/flatpak-app-apply-theme/):
```
flatpak remote-add flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install org.kde.KStyle.Adwaita//5.9 -y
flatpak install org.kde.PlatformTheme.QGnomePlatform//5.9 -y
sudo flatpak override --filesystem=$HOME/.themes
sudo flatpak override --filesystem=$HOME/.icons
```

#### Install Spack

"[Spack](https://spack.readthedocs.io/en/latest/index.html) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments. It was designed for large supercomputing centers, where many users and application teams share common installations of software on clusters with exotic architectures, using libraries that do not have a standard ABI. Spack is non-destructive: installing a new version does not break existing installations, so many configurations can coexist on the same system".

Install Spack:
```
git clone --depth=2 https://github.com/spack/spack.git
. spack/share/spack/setup-env.sh
spack compiler find
```

### Drivers

*Make sure to [update the system](#update-the-system) before installing drivers.*

#### GPU drivers for NVIDIA cards

Follow the [NVIDIA Driver Installation Guide](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/).

To preserve video memory after suspend, follow the [Arch Linux Wiki](https://wiki.archlinux.org/title/NVIDIA/Tips_and_tricks#Preserve_video_memory_after_suspend).

Install [Vulkan libraries](https://rpmfusion.org/Howto/NVIDIA#Vulkan):
```
sudo dnf install vulkan -y
```

#### AMD drivers

Fedora ships up to date `mesa` driver which is all you will need.

Install [Vulkan libraries](https://rpmfusion.org/Howto/NVIDIA#Vulkan):
```
sudo dnf install vulkan -y
```

#### Multimedia codec

Some applications require the sound-and-video complement packages as well as hardware accelerated codecs, so for that follow the RPM Fusion Multimedia guide (see [Multimedia on Fedora](https://rpmfusion.org/Howto/Multimedia)):
```
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin
```

Hardware accelerated codec:
<details>
  <summary>Intel</summary>
  <br/>
  
  > ```
  > sudo dnf install intel-media-driver -y
  > ```
</details>

<details>
  <summary>NVIDIA</summary>
  <br/>

  > ```
  > sudo dnf install libva-nvidia-driver.{i686,x86_64}
  > ```
</details>

<details>
  <summary>AMD (needed since Fedora 37 and later)</summary>
  <br/>

  > ```
  > sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
  > sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
  > sudo dnf swap mesa-va-drivers.i686 mesa-va-drivers-freeworld.i686
  > sudo dnf swap mesa-vdpau-drivers.i686 mesa-vdpau-drivers-freeworld.i686
  > ```
</details>

### Storage optimization

Optimize the BRTFS filesystem partition by adding the following tags in `/etc/fstab` (see [BTRFS specific mount options](https://btrfs.readthedocs.io/en/latest/Administration.html#mount-options)). A fstab file example is given [here](https://github.com/mBelisarius/fedora-for-machine-learning/blob/main/fstab.example).
```
compress=zstd:3,discard=async,commit=60,usebackuproot
```

To apply the changes, run
```
sudo systemctl daemon-reload
```

### Tweaks

Use [hyperreal/better_fonts](https://copr.fedorainfracloud.org/coprs/hyperreal/better_fonts/) for better font rendering using free substitutions for popular proprietary fonts from Microsoft and Apple operating systems and enable subpixel (rgb) antialiasing:
```
sudo dnf copr enable hyperreal/better_fonts -y
sudo dnf install fontconfig-font-replacements fontconfig-enhanced-defaults -y
```

### Clean DNF cache

Clean temporary files kept for repositories (see [clean command](https://dnf.readthedocs.io/en/latest/command_ref.html?highlight=clean#clean-command)): 
```
sudo dnf clean all
```


## Setting up development tools

Setting up development tools and multiple programming languages.

### Monitoring

#### nmon

[nmon](https://nmon.sourceforge.net/pmwiki.php) is a resource monitor that outputs a huge amount of important performance information in one go on screen and saves the data in csv file. Install:
```
sudo dnf install nmon -y
```

#### Btop++

[Btop++](https://github.com/aristocratos/btop) is an easy to use console resource monitor that shows usage and stats for processor, memory, disks, network and processes. Install:
```
sudo dnf install btop -y
```

### Git

Install Git and configure it following the [Git quick reference](https://fedoraproject.org/wiki/Git_quick_reference):
```
sudo dnf install git -y
```

### Virtualization

Install QEMU and VMM for virtual machines management (see [Getting started with virtualization](https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-virtualization/)):
```
sudo dnf group install --with-optional virtualization -y
sudo systemctl enable libvirtd
```

### Containerization

#### Docker

Install Docker (see [Install Docker Engine on Fedora](https://docs.docker.com/engine/install/fedora/)):
```
sudo dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

To start the Docker service use:
```
sudo systemctl start docker
```

Verify that Docker was correctly installed and is running by running the Docker hello-world image:
```
sudo docker run hello-world
```

To make Docker start when you boot your system, use the command:
```
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

To manage Docker as a non-root user, create a Unix group called docker and add users to it. Note: This has security implications, read about this in the [official docs](https://docs.docker.com/engine/security/#docker-daemon-attack-surface).
```
sudo groupadd docker && sudo gpasswd -a ${USER} docker && sudo systemctl restart docker
newgrp docker
```

Install [Docker Desktop](https://docs.docker.com/desktop/install/fedora/#install-docker-desktop).

#### Vagrant

[WIP]

### Compilers, interpreters and SDKs

#### C/C++

Install [GNU C++](https://gcc.gnu.org/) and [LLVM/CLang](https://packages.fedoraproject.org/pkgs/llvm/) compilers (see [C++ installation](https://developer.fedoraproject.org/tech/languages/c/cpp_installation.html)):
```
sudo dnf install gcc-c++ gdb -y
sudo dnf install llvm clang clang-analyzer clang-devel lldb lldb-devel python3-lldb llvm-devel -y
```

Install [GNU Make](https://packages.fedoraproject.org/pkgs/make/make/) and [CMake](https://developer.fedoraproject.org/tech/languages/c/cmake.html):
```
sudo dnf install make -y
sudo dnf install cmake -y
```

Install [Cppcheck](https://cppcheck.sourceforge.io/) for static code analysis:
```
sudo dnf install cppcheck -y
```

Install [Valgrind](https://valgrind.org/) for memory debugging, memory leak detection, and profiling:
```
sudo dnf install valgrind valgrind-openmpi -y
```

Install [KCachegrind](https://docs.kde.org/stable5/en/kcachegrind/kcachegrind/index.html):
```
flatpak install flathub org.kde.kcachegrind -y
```

Install [Massif-Visualizer](https://phabricator.kde.org/source/massif-visualizer/), a GUI to visualize output from Massif:
```
flatpak install flathub org.kde.massif-visualizer -y
```

#### Fortran

Install [GNU Fortran](https://gcc.gnu.org/fortran/) compiler (see [Fortran installation](https://developer.fedoraproject.org/tech/languages/fortran/fortran-installation.html)):
```
sudo dnf install gcc-gfortran -y
```

#### Python

Fedora ships with Python for system and package management and should not be used for general purpose coding. So you will need to use virtual environment or other Python distributions. Using the Conda distribution is a common practice and highly recommended.

Install [Conda distribution](https://www.anaconda.com/products/distribution) (either Miniconda or Anaconda, but Miniconda is recommended) for Python environments. Miniconda installation:
```
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -o Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

#### Java

Install [SDKMan](https://sdkman.io/install) to manage parallel versions of multiple Java SDKs:
```
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

Install [Maven](https://developer.fedoraproject.org/tech/languages/java/java-build-tools-installation.html):
```
sudo dnf install maven -y
```

Install [Gradle](https://docs.gradle.org/current/userguide/installation.html#installing_with_a_package_manager):
```
sdk install gradle
```

#### Rust

Install [Rust](https://www.rust-lang.org/tools/install) using Rustup:
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### .NET

Install [.NET SDK](https://learn.microsoft.com/en-us/dotnet/core/install/linux-fedora) and runtime:
```
sudo dnf install dotnet -y
```

Install [Mono](https://developer.fedoraproject.org/tech/languages/dotnet/mono.html):
```
sudo dnf install mono-* nunit -y
```

### Database engines and management systems

Install SQLite (see [SQLite installation](https://developer.fedoraproject.org/tech/database/sqlite/about.html)):
```
sudo dnf install sqlite sqlite-devel sqlite-tcl sqlite-analyzer sqlite-tools lemon sqlitebrowser -y
```

### CUDA Toolkit

There are multiple ways to install and run applications with the CUDA Toolkit. The easiest way is by using a Docker container due to the hability to freely manage versions and dependencies, other options include Conda environment and installing CUDA direct into the OS.

[WIP]

### IDEs and text-editors

Install your preferred text-editor.

[Sublime Text](https://www.sublimetext.com/docs/linux_repositories.html#dnf): 
```
sudo rpm -v --import https://download.sublimetext.com/sublimehq-rpm-pub.gpg
sudo dnf config-manager --add-repo https://download.sublimetext.com/rpm/stable/x86_64/sublime-text.repo
sudo dnf install sublime-text -y
```

[VSCode](https://code.visualstudio.com/docs/setup/linux):
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
sudo dnf install code -y
```

Install JetBrains ToolBox for JetBrains IDEs: [JetBraibs ToolBox](https://www.jetbrains.com/toolbox-app/)

## Environments setup

### Conda environment for Tensorflow

Follow [Tensorflow step-by-step instructions](https://www.tensorflow.org/install/pip#step-by-step_instructions).

## Notes

For any problems, enhancements or questions, feel free to open an [issue](https://github.com/mBelisarius/fedora-for-machine-learning/issues/new).
