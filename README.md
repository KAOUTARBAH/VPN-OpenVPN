# VPN-OpenVPN

## Réalise cet atelier jusqu'à avoir une communication sécurisée entre les 2 PC.

# Objectifs :

✅ Installer OpenVPN  
✅ Créer une clé de chiffrement  
✅ Mettre en place un VPN à clé de chiffrement unique

# Prérequis

Tu as besoin de 2 PC ou VM sous VirtualBox.  
Pour chacun :

- Une distribution Linux, comme Ubuntu  
- 1 carte réseau en mode **Réseau interne**, avec l'adresse **172.16.10.1** pour **PC1** et **172.16.10.2** pour **PC2**  
- 1 carte réseau permettant d'aller sur internet, en mode **NAT** par exemple  

    - PC1
    ```bash
        sudo nano /etc/netplan/50-cloud-init.yaml


    ```bash
        network:
        ethernets:
            enp0s8:
            dhcp4: no
            addresses:
                - 172.16.10.1/24
            enp0s3:
            dhcp4: yes
        version: 2

        - PC2
    ```bash
        sudo nano /etc/netplan/50-cloud-init.yaml


    ```bash
        network:
        ethernets:
            enp0s8:
            dhcp4: no
            addresses:
                - 172.16.10.2/24
            enp0s3:
            dhcp4: yes
        version: 2

- Applique la configuration avec la commande :
    ```bash
        sudo netplan apply

- Un compte **wilder1** sur **PC1**, et **wilder2** sur **PC2**  
  *(Ne pas oublier de donner les droits sudo à chacun de ces utilisateurs avec :)*  
    ```bash
    usermod -aG sudo <user>

- PC1 est le client et PC2 est le serveur, la connexion sécurisée se fera donc de PC1 vers PC2.


## Configuration initiale des PC
- Sur **PC1** et **PC2**, faire les actions suivantes.

### MAJ des paquets :
    ```bash
    sudo apt update && sudo apt upgrade -y

- Installation d'OpenVPN :
    ```bash
    sudo apt install openvpn

- Installation de SSH :
    ```bash
    sudo apt install openssh-server


### Création d'une clé sécurisée

- Si les machines sont des VM, il faut **désactiver ou supprimer les cartes NAT** et ne garder que les **cartes réseaux du réseau interne**.
- Dans la suite de cet atelier, tu vas **créer une clé de chiffrement**, qui sera partagée entre le **client** et le **serveur**.

- Sur **PC1**, place-toi dans le répertoire :
    ```bash
    cd /etc/openvpn/client
    sudo openvpn --genkey secret ma-cle_vpn.key

- Tu peux vérifier le contenu de la clé avec la commande cat :
    ```bash
    sudo cat ma_cle_vpn.key 

![openVpn](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/OpenVpn.png)


## Envoi de la clé sur le serveur
### Méthode : Copier la clé de PC1 vers PC2

1. **Copie de la clé de PC1 vers PC2**  
   Sur PC1, utilisez la commande suivante pour copier le fichier `static-OpenVPN.key` vers PC2. Cette commande utilise `scp` et nécessite des privilèges administratifs (`sudo`) :
    ```bash
    sudo scp ma_cle_vpn.key wilder2@172.16.10.2:~

![copie-cle](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/copie-cle.png)

2. Connexion en SSH sur PC2
Ensuite, établissez une connexion SSH vers PC2 avec la commande suivante :
    ```bash
    ssh wilder2@172.16.10.2

![connSSH](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/connSSH.png)
### 3. Copie de la clé vers le répertoire approprié

Une fois connecté en SSH sur PC2, copiez la clé du répertoire personnel (`~`) vers le répertoire `/etc/openvpn/server/`. Cette opération nécessite des privilèges administratifs. Utilisez la commande suivante :

    ```bash
    sudo cp ~/ma_cle_vpn.key /etc/openvpn/server/

![copie ok](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/copie-ok.png)


## Vérification du ping de PC1 vers PC2 avec Wireshark

1. **Effectuer un ping depuis PC1 vers PC2 :**

   Ouvre un terminal sur **PC1** et envoie un ping vers **PC2** (adresse IP `172.16.10.2`) :

   ```bash
   ping 172.16.10.2

2. **Le protocole est bien l'ICMP **
Lance Wireshark avec la commande suivante :
    ```bash
    sudo wireshark

Sélectionne l'interface réseau qui est utilisée pour la communication entre PC1 et PC2 (généralement enp0s8 et, wlan0, etc.).


3. **La partie Data du protocole ICMP est visible en clair**
![wireshark](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/wireshark.png)

## Mise en place du VPN

### Sur PC1 :

- **Place-toi dans le dossier où tu as placé la clé.**  
- Assure-toi que le fichier de clé `ma_cle_vpn.key` est bien dans le dossier approprié. Tu peux utiliser la commande `cd` pour naviguer jusqu'à ce dossier. Par exemple :

   ```bash
    sudo openvpn --dev tun --remote 172.16.10.2 --ifconfig 10.10.5.1 10.10.5.2 --cipher AES-256-CBC --secret ma_cle_vpn.key

![clientOpenVpn](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/clientOpenVpn.png)

### Sur PC2 :
- **Place toi dans le dossier où tu as mis la clé.**
- Exécute la commande 

    ```bash
    sudo openvpn --dev tun --ifconfig 10.10.5.2 10.10.5.1 --cipher AES-256-CBC --secret ma_cle_vpn.key

![serverOpenVpn](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/serverOpenVpn.png)

- La communication sécurisée est établit lorsque sur les 2 machines il y a le message Initialization Sequence Completed.


## Test d'une communication sécurisée

### 1. Lancer un ping de PC1 vers PC2
- Ouvrir un terminal ou l’invite de commande sur **PC1**.
- Exécuter la commande suivante pour tester la connectivité :
  ```bash
  ping 10.10.5.2

![clt icmp OpenVpn](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/clt-icmp-vpn.png)

### 1. Lancer un ping de PC2 vers PC1
    ```bash
    ping 10.10.5.1

![server icmp OpenVpn](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/server-icmp-vpn.png)

- Capturer le trafic OpenVPN
- Ouvre Wireshark sur PC1 ou PC2.

Sélectionne l’interface réseau utilisée par OpenVPN :

- Si VPN actif → Choisis tun0 (trafic déjà déchiffré).
- Si tu veux voir le chiffrement brut → Sélectionne eth0 ou wlan0 (interface réseau physique).
![wireshark](https://github.com/KAOUTARBAH/VPN-OpenVPN/blob/main/images/wireshark.png)