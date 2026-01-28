# Proxmox - Configuration du stockage LVM local-mpx

## Description

Ce guide explique comment supprimer le stockage `local-lvm` par défaut de Proxmox et créer un nouveau stockage de type Directory nommé `local-mpx` qui utilise 100% de l'espace LVM disponible. Ce stockage pourra héberger les backups, ISO, templates, et disques de VMs/CTs.

## Avertissement

**Cette procédure supprime définitivement le stockage `local-lvm` et toutes les données qu'il contient.**

- Sauvegardez toutes vos VMs et conteneurs avant de continuer
- Arrêtez toutes les VMs/CTs utilisant `local-lvm`
- Cette opération est irréversible

## Prérequis

- Accès root à votre serveur Proxmox
- Aucune VM/CT critique sur `local-lvm` (ou backups effectués)

## Installation

### 1. Vérifier l'état actuel du stockage

```bash
# Voir les volumes logiques existants
lvs

# Voir l'espace disponible dans le volume group
vgs

# Lister les VMs et CTs
qm list
pct list
```

### 2. Supprimer local-lvm de Proxmox

```bash
# Retirer le stockage de la configuration Proxmox
pvesm remove local-lvm
```

### 3. Supprimer le thin pool LVM

```bash
# Supprimer le thin pool (libère l'espace)
lvremove /dev/pve/data

# Confirmer avec 'y' lorsque demandé
```

### 4. Créer local-mpx avec 100% de l'espace libre

```bash
# Vérifier l'espace libéré
vgs

# Créer le logical volume avec tout l'espace disponible
lvcreate -l 100%FREE -n local-mpx pve

# Formater en ext4
mkfs.ext4 /dev/pve/local-mpx

# Créer le point de montage
mkdir -p /mnt/local-mpx

# Monter le volume
mount /dev/pve/local-mpx /mnt/local-mpx

# Rendre le montage permanent au redémarrage
echo "/dev/pve/local-mpx /mnt/local-mpx ext4 defaults 0 2"
```
Vous pouvez modifier `local-mpx` par le nom que vous souhaitez
