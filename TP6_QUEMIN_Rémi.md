# tp6-admin-systeme-RemiQuemin
### Exercice 1. Disques et partitions

#### 1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM
...

#### 2. Vérifiez que ce nouveau disque dur est bien détecté par le système
j'ai fait `fdisk -l`.

#### 3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)
J'ai d'abord fait fdisk /deb/sdb puis j'ai mis : p, 1, puis entrée puis +2G pour choisir 2Go.

J'ai refait les mêmes commande pour la seconde partition NTFS mais j'ai mis +3G et ensuite une fois créee on appuie sur t et on choisi la seconde partition le numéro 7 (NTFS).

#### 4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)
J'ai fait la commande mkfs.ext4 /dev/sdb1 pour la première partition puis la commande mkfs.ntfs /dev/sdb2 pour la deuxième partition.

#### 5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?
La commande df -T n'affiche pas le disque sdb car il n'est pas encore monté

#### 6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)
nano /etc/fstab

#### 7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration
J'ai utilisé la commande `mount -a`.
#### 8. Montez votre clé USB dans la VM

#### 9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox.

### Exercice 2. Partitionnement LVM
Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler les disques et les partitions.
#### 1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab.
J'ai utilisé la commande `umount /data` et `umount /win` pour démonter les systèmes de fichiers montés dans /data et /win.
#### 2. Supprimez les deux partitions du disque, et créez une patition unique de type LVM
 La création d’une partition LVM n’est pas indispensable, mais vivement recommandée quand on utilise LVM sur un disque entier. En effet, elle permet d’indiquer à d’autres OS ou logiciels de gestion de disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données sur ce disque.
 Attention à ne pas supprimer la partition système !

J'ai utilisé la commande `fdisk /dev/sdb`puis d pour supprimer les deux partitions.  

#### 3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay.
``` 
Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8E
Changed type of partition 'Linux' to 'Linux LVM'.
```

```
root@serveur:/home/serveur# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
root@serveur:/home/serveur# pvdisplay
*  "/dev/sdb1" is a new physical volume of "2,00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               2,00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               pHUkqM-CoFg-jFfw-O66Z-8wTX-QQWB-TQUnvm
  ```
  
#### 4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.
```
root@serveur:/home/serveur# vgcreate volume1 /dev/sdb1
  Volume group "volume1" successfully created
root@serveur:/home/serveur# vgdisplay
  --- Volume group ---
  VG Name               volume1
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <2,00 GiB
  PE Size               4,00 MiB
  Total PE              511
  Alloc PE / Size       0 / 0
  Free  PE / Size       511 / <2,00 GiB
  VG UUID               OivH5Q-v3Ie-hcaj-ZbRm-qIEg-Viq6-pdr0jf
  ```
#### 5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.
```
root@serveur:/home/serveur# lvcreate -l 100%FREE -n 1vData volume1
  Logical volume "1vData" created.
  ```
#### 6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.
`fdisk /dev/mapper/volume1-1vData` puis `mkfs.ext4 /dev/mapper/volume1-1vData` puis rajouté dans le fstab la ligne `/dev/mapper/volume1-1vData /data ext4 defaults 0 0`.

#### 7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.
Done.

#### 8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes
`vgextend volume1 /dev/sdc1`.

#### 9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.
`lvextend -l 100%FREE /dev/volume1/1vData` puis `resize2fs /dev/volume1/1vData`.
