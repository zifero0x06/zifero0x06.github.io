# Astuces Linux

## Créer une clé USB bootable
1. utiliser la commande `lsblk` pour lister les disques actuellement montés
2. utiliser la commande `sudo dd bs=4M if=path/to/input.iso of=/dev/sd<?> conv=fdatasync  status=progress` avec input.iso le fichier copier et \<?> le disque USB sur lequel écrire

Pour un <*.iso> Windows, préférer l'utilisation de Rufus.

## Connaitre son ordinateur

1. Connaitre son CPU : `lscpu`
2. Connaitre son usage RAM : `free -m`
3. Connaitre ses disques : `lsblk`
4. Connaitre ses périphériques PCI : `lspci`
5. Connaitre son BIOS : `dmidecode -t bios`
6. Afficher des informations générales : `dmidecode -t system`

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






