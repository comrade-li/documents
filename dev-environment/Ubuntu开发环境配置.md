# Ubuntu开发环境配置

## 1. Ubuntu24.04不支持ras1024的警告消除

```shell
echo 'APT::Key::Assert-Pubkey-Algo ">=rsa1024";' | sudo tee /etc/apt/apt.conf.d/99weakkey-warning
```

## 2. 彻底阻止Ubuntu重新安装snap

```shell
echo 'Package: snapd
Pin: release a=*
Pin-Priority: -10' | sudo tee /etc/apt/preferences.d/nosnap.pref
```

## 3. 配置vim

```shell
echo 'set nocompatible
syntax on
set showmode
set showcmd
set encoding=utf-8
set t_Co=256
filetype indent on
set autoindent
set tabstop=2
set shiftwidth=2
set expandtab
set softtabstop=2
set number
set textwidth=120
set nowrap
set wrapmargin=2
set scrolloff=5
set sidescrolloff=15
set laststatus=2
set ruler
set showmatch
set hlsearch
set incsearch
set ignorecase
set smartcase
set nobackup
set noswapfile
set noerrorbells
set wildmenu
set wildmode=longest:list,full' | tee ~/.vimrc
```

## 4. 配置环境变量

配置PATH环境如下:

```shell
echo '
MANPATH=/usr/local/texlive/2024/texmf-dist/doc/man
INFOPATH=/usr/local/texlive/2024/texmf-dist/doc/info
TEXLIVE_HOME=/usr/local/texlive/2024/bin/x86_64-linux

JAVA_HOME=~/.softwares/java/current
GRADLE_HOME=~/.softwares/gradle/current
MAVEN_HOME=~/.softwares/maven/current
JMETER_HOME=~/.softwares/jmeter
VISUALVM_HOME=~/.softwares/visualvm

PATH=$PATH:$TEXLIVE_HOME:$JAVA_HOME/bin:$GRADLE_HOME/bin:$MAVEN_HOME/bin:$JMETER_HOME/bin:$VISUALVM_HOME/bin

export PATH MANPATH INFOPATH TEXLIVE_HOME JAVA_HOME GRADLE_HOME GRADLE_USER_HOME MAVEN_HOME' | tee -a ~/.profile
```

## 5. 安装基础开发包

1. 安装基础环境

    ```shell
    sudo apt install git vim zsh ibus-rime openssh-server curl build-essential cmake ninja-build tcl tk tcl-dev tk-dev gnome-tweaks gnome-shell-extensions 
    ```

2. 安装llvm和clang

    ```shell
    sudo apt install clang-format clang-tidy clang-tools clang clangd libc++-dev libc++1 libc++abi-dev libc++abi1 libclang-dev libclang1 liblldb-dev libllvm-ocaml-dev libomp-dev libomp5 lld lldb llvm-dev llvm-runtime llvm python3-clang
    ```

3. 安装并配置KVM

    ```shell
    sudo apt install qemu-system-x86 virt-manager libvirt-daemon-system virtinst libvirt-clients bridge-utils
    ```

    ```shell
    sudo systemctl enable --now libvirtd && 
    sudo systemctl start libvirtd && 
    sudo usermod -aG kvm $USER && 
    sudo usermod -aG libvirt $USER
    ```

4. 安装并配置wireshark

    ```shell
    sudo apt install wireshark
    ```

    ```shell
    sudo chgrp wireshark /usr/bin/dumpcap && 
    sudo chmod 4755 /usr/bin/dumpcap && 
    sudo gpasswd -a ${USER} wireshark
    ```

## 6. 配置桥接网络

添加配置文件

```shell
sudo vim /etc/netplan/01-network-bridge-config.yaml
```

```yaml
network:
  ethernets:
    enp6s0:
      dhcp4: false
      dhcp6: false
  bridges:
    bridge0:
      interfaces: [enp6s0]
      dhcp4: false
      addresses: [192.168.1.10/24]
      macaddress: 0A:E0:AF:B2:01:35
      routes:
        - to: default
          via: 192.168.1.1
          metric: 100
      nameservers:
        addresses: [223.5.5.5,8.8.4.4,223.6.6.6]
      parameters:
        stp: false
      dhcp6: false
  version: 2
```

重启网络

```shell
sudo netplan apply
```

## 6. 配置ssh和git

1. 配置ssh

    ```shell
    ssh-keygen -t rsa -b4096 -C "comrade.lijing@gmail.com" && ssh-add ~/.ssh/id_rsa && cat ~/.ssh/id_rsa.pub
    ```

2. 配置git

    ```shell
    git config --global user.email "comrade.lijing@gmail.com" && git config --global user.name "Comrade Li"
    ```

## 7. 安装并配置oh-my-zsh

1. 安装oh-my-zsh

    ```shell
    git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
    ```

2. 安装插件

    ```shell
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
    ```

    ```shell
    git clone https://github.com/zsh-users/zsh-autosuggestions.git ~/.oh-my-zsh/plugins/zsh-autosuggestions
    ```

3. 默认配置

    ```shell
    cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
    ```

4. 更换默认sh为zsh

    ```shell
    chsh -s $(which zsh)
    ```

5. 更新配置

    ```shell
    sed -i 's/plugins=(git)/plugins=(\n  git\n  zsh-autosuggestions\n  zsh-syntax-highlighting\n)\nsource ~\/.profile/g' ~/.zshrc
    ```

## 8. 安装oh-my-rime输入法

```shell
git clone https://github.com/Mintimate/oh-my-rime.git ~/.config/ibus/rime
```

## 9. 安装字体

1. 拉取配置

    ```shell
    git clone git@github.com:comrade-li/documents.git ~/Projects/documents
    ```

2. 配置字体

    ```shell
    cd ~/Projects/documents/dev-environment/fonts && 
    sudo tar -vxJf sarasa-ui-sc.tar.xz -C /usr/share/fonts/truetype && 
    sudo tar -vxJf jetbrains-mono.tar.xz -C /usr/share/fonts/truetype && 
    sudo tar -vxJf intel-one-mono.tar.xz -C /usr/share/fonts/truetype && 
    sudo tar -vxJf lxgw-wenkai-mono.tar.xz -C /usr/share/fonts/truetype && 
    sudo tar -vxJf source-code-pro.tar.xz -C /usr/share/fonts/truetype && 
    sudo tar -vxJf sf-mono.tar.xz -C /usr/share/fonts/truetype && 
    sudo tar -vxJf inter.tar.xz -C /usr/share/fonts/truetype && 
    sudo tar -vxJf courier-prime.tar.xz -C /usr/share/fonts/truetype && 
    fc-cache -fv && sudo fc-cache -fsv && fc-cache -fsv
    ```

## 10. JetBrains idea的chrome-sandbox权限问题

```shell
sudo chown root:root ~/.softwares/idea-IU/jbr/lib/chrome-sandbox && sudo chmod 4755 ~/.softwares/idea-IU/jbr/lib/chrome-sandbox
```

## 11. Debian 12卸载无用软件

```shell
sudo apt remove gnome-2048 gnome-contacts gnome-weather gnome-clocks gnome-maps aisleriot gnome-calendar gnome-video-effects gnome-chess simple-scan five-or-more four-in-a-row hitori gnome-chess im-config gnome-klotski lightsoff gnome-mahjongg gnome-mines gnome-music gnome-nibbles quadrapassel iagno rhythmbox gnome-robots shotwell gnome-sound-recorder gnome-sudoku swell-foop tali gnome-taquin gnome-tetravex gnome-text-editor transmission-gtk evolution synaptic totem gnome-software libreoffice-*
```
