# Astuces Linux

## Créer une clé USB bootable
1. utiliser la commande `lsblk` pour lister les disques actuellement montés
2. utiliser la commande `sudo dd bs=4M if=path/to/input.iso of=/dev/sd<?> conv=fdatasync  status=progress` avec input.iso le fichier copier et \<?> le disque USB sur lequel écrire

Pour un <*.iso> Windows, préférer l'utilisation de Rufus.

## Connaitre son ordinateur

1. Connaitre son CPU : `lscpu`
2. Connaitre son usage RAM : `free -m`
3. Connaitre ses caractéristiques de RAM : `sudo dmidecode -t memory`
4. Connaitre ses disques : `lsblk`
5. Connaitre ses périphériques PCI : `lspci`
6. Connaitre son BIOS : `dmidecode -t bios`
7. Afficher des informations générales : `dmidecode -t system`

## Compiler du code C/C++ directement en assembleur

```bash
$ cat >hello.c <<EOF

#include <stdio.h>
int main(void) {
    printf("Hello, world!\n");
    return 0;
}
EOF

$ gcc -S hello.c -o hello.s
```

## Gérer ses connections avec Network Manager

Pour désactiver IPv6 puis pour assigner une IPv4 statique à un terminal linux en passant la CLI de Network Manager :

```bash
sudo nmcli connection show

sudo nmcli connection modify enp0s3 ipv6.method "disabled"
sudo nmcli connection up enp0s3 

sudo nmcli connection modify enp0s3 ipv4.addresses 192.168.2.100/24
sudo nmcli con mod enp0s3 ipv4.gateway 192.168.2.1
sudo nmcli con mod enp0s3 ipv4.method manual
sudo nmcli con mod enp0s3 ipv4.dns "1.1.1.1"
sudo nmcli con up enp0s3
```

Pour vérifier les changements apportés dans le fichier de configuration : 

```bash
sudo nano etc/sysconfig/network-scripts ifcfg-enp0s3
```

A savoir que :
- 'con' est un alias de 'connection'
- 'mod' est un alias de 'modify"

## Modifier le nom d'hôte d'une machine

Pour modifier le nom d'hôte ou nom de machine Fedora de manière permanente :

```bash
sudo hostnamectl set-hostname --pretty "nouveau_nom"
sudo hostnamectl set-hostname --transient "nouveau_nom"
sudo hostnamectl set-hostname --static "nouveau_nom"
```

Puis redémarrer la machine.

## Démarrer un serveur Nginx sous Fedora

Après avoir ajuster les règles du firewall 

```bash
sudo systemctl status nginx
sudo systemctl enable nginx.service
sudo systemctl start nginx.service
```

## Ajouter une interface Xfce4 à Fedora server


```bash
dnf -y group install "Xfce Desktop"
```

Après installation :

```bash
echo "/usr/bin/startxfce4" >> ~/.xinitrc
startx
```

## Sous Windows/VirtualBox avec Powershell/cmd, utiliser VboxManage pour implémenter une carte réseau au-delà de la quatrième

Ajouter des interfaces réseaux (NIC ou Adapter) supplémentaires (plus de 4 car impossible au-delà avec la GUI) en ligne de commande (CLI) sur VirtualBox :

```shell
VBoxManage modifyvm --nic<5-8> none|null|nat|bridged|intnet|hostonly|generic|natnetwork
VBoxManage modifyvm --nictype<5-8> Am79C970A|Am79C973|82540EM|82543GC|82545EM|virtio
VBoxManage modifyvm --macaddress<5-8> auto|<mac>
VBoxManage modifyvm --cableconnected<5-8> on|off
```

Puis, en fonction des besoins :

```shell
VBoxManage modifyvm --bridgeadapter<5-8> none|<devicename>
VBoxManage modifyvm --hostonlyadapter<5-8> none|<devicename>
VBoxManage modifyvm --intnet<5-8> <network name>
VBoxManage modifyvm --natnet<5-8> <network>|default
VBoxManage modifyvm --nat-network<5-8> <network name>
```

## Installer Exegol

Sous Fedora, installer les éléments prérequis puis Docker puis Exegol :

```bash
sudo dnf install git
sudo dnf install python3
sudo dnf install python3-pip
curl -fsSL "https://get.docker.com/" | sh
sudo usermod -aG docker $(id -u -n)
newgrp docker
sudo systemctl enable docker
sudo systemctl start docker
sudo docker run hello-world

python3 -m pip install pipx
pipx install exegol
pipx ensurepath
echo "alias exegol='sudo -E $(which exegol)'" >> ~/.bash_aliases
source ~/.bashrc
exegol install
```

Installer l'autocompletion :

```bash
sudo dnf install python3-argcomplete
pip3 install --user argcomplete
pipx install argcomplete
sudo dnf update && sudo dnf install bash-completion
```
A ce point, il devrait y avoir un fichier /etc/bash_completion.d/exegol existant.
Sinon, créer un fichier vide 'exegol' puis :

```bash
register-python-argcomplete --no-defaults exegol | sudo tee etc/bash_completion.d/exegol > /dev/null
```

# Liste d'outils indispensables :

```bash
sudo dnf install openvpn
sudo dnf install filezilla
sudo dnf install openrgb
sudo dnf install vlc
sudo dnf install bash-completion
sudo dnf install curl
sudo dnf install neofetch
sudo dnf install timeshift
sudo dnf install gcc-c++
sudo dnf install gcc
sudo dnf install python3
sudo dnf install keepassxc
sudo dnf install yakuake
sudo dnf install konversation
sudo dnf install gparted
```

```bash
sudo dnf config-manager --enable fedora-cisco-openh264
```

Éditeur graphique [yEd](https://www.yworks.com/products/yed/download#download)

Suite bureautique [ONLYOFFICE](https://www.onlyoffice.com/fr/download-desktop.aspx#desktop)

Oracle [VirtualBox](https://www.virtualbox.org/wiki/Linux_Downloads)

Pour finaliser l'installation de VirtualBox :

```bash
sudo usermod -a -G vboxusers $USER
sudo dnf install kernel-devel
sudo /sbin/vboxconfig
```


# Pour réparer l'affichage aperçu (document preview) de l'éditeur KDE Kate lors de l'édition markdown :

Activer le plugin "Aperçu de document" en allant dans Configuration > Configurer Kate... > Modules externes >cocher la case "Aperçu de document"

```bash
sudo dnf install markdownpart
```

# Sous Fedora, pour installer le navigateur Brave :

```bash
sudo dnf update
sudo dnf install dnf-plugins-core
sudo dnf config-manager --add-repo https://brave-browser-rpm-release.s3.brave.com/brave-browser.repo
sudo rpm --import https://brave-browser-rpm-release.s3.brave.com/brave-core.asc
sudo dnf install brave-browser
```

