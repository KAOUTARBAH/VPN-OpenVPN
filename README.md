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

Applique la configuration avec la commande :

```bash
    sudo netplan apply

- Un compte **wilder1** sur **PC1**, et **wilder2** sur **PC2**  
  *(Ne pas oublier de donner les droits sudo à chacun de ces utilisateurs avec :)*  
  ```bash
  usermod -aG sudo <user>

 - PC1 est le client et PC2 est le serveur, la connexion sécurisée se fera donc de PC1 vers PC2.


## Configuration initiale des PC

Sur **PC1** et **PC2**, faire les actions suivantes.

### MAJ des paquets :

```bash
sudo apt update && sudo apt upgrade -y

- Installation d'OpenVPN :
```bash
sudo apt install openvpn

- Installation de SSH :
```bash
sudo apt install openssh-server



# 1. Installer iptables-persistent

Comme mentionné précédemment, tu peux installer le paquet **iptables-persistent**, qui est un moyen simple de rendre les règles iptables persistantes.

### Installer iptables-persistent :
Si le répertoire `/etc/iptables/` n'existe pas, il se peut que tu n'aies pas installé **iptables-persistent**. Ce paquet permet de sauvegarder et de recharger les règles iptables au démarrage.

Exécute cette commande pour l'installer :

```bash
sudo apt update
sudo apt install iptables-persistent

### Sauvegarder les règles iptables :
Une fois iptables-persistent installé, il te demandera automatiquement si tu souhaites sauvegarder tes règles actuelles lorsque tu l'installes. Accepte cette demande.

- Si tu n'as pas eu cette demande, tu peux enregistrer manuellement les règles avec :
```bash
sudo netfilter-persistent save


# 4. Configuration de NAT (si nécessaire) :

Tu dois configurer le **NAT** pour que les machines du réseau local puissent accéder à Internet via l'interface **enp0s8**. Pour cela, tu peux utiliser **iptables** pour configurer la translation d'adresses réseau (NAT).

### Voici une méthode pour configurer cela :

#### Activer le routage IP :
Ouvre un terminal et exécute cette commande pour activer le routage IP :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
# Configuration de NAT et vérification

Pour que ce changement soit permanent, ajoute ou modifie la ligne suivante dans `/etc/sysctl.conf` :

```bash
net.ipv4.ip_forward = 1

# Configurer le NAT avec iptables :

Utilise **iptables** pour configurer le NAT afin de permettre à ton réseau local d’accéder à Internet via **enp0s8**.

Exécute cette commande pour ajouter une règle NAT :

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE

Cela configure le NAT pour masquer l'adresse IP du réseau local avec l'adresse IP de l'interface **enp0s8** lorsque tu accèdes à Internet.

### Sauvegarder les règles iptables :

Pour que les règles **iptables** persistent après un redémarrage, tu peux les enregistrer avec la commande suivante :

```bash
sudo iptables-save > /etc/iptables/rules.v4

Cela va créer le répertoire /etc/iptables/ et y sauvegarder tes règles dans un fichier rules.v4.
```bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4 > /dev/null

# 5. Vérification :

Après avoir configuré le routage et le NAT, essaie de tester la connexion à Internet depuis la machine qui utilise **enp0s3** :

1. Vérifie l'adresse IP de la machine avec :

   ```bash
   ip a
2. Utilise **ping** pour tester la connectivité :

```bash
ping 8.8.8.8
ping google.com


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

![vpn](https://github.com/KAOUTARBAH/Atelier--GPO/blob/main/images/OpenVpn.png)

