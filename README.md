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
            enp0s3:
            dhcp4: no
            addresses:
                - 172.16.10.1/24
            enp0s8:
            dhcp4: yes
        version: 2

        - PC2
    ```bash
        sudo nano /etc/netplan/50-cloud-init.yaml


    ```bash
        network:
        ethernets:
            enp0s3:
            dhcp4: no
            addresses:
                - 172.16.10.2/24
            enp0s8:
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

![openVpn](https://github.com/KAOUTARBAH/VPN-OpenVPN/tree/main/images/OpenVpn.png)

