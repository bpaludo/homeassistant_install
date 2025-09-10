# homeassistant_install

no shell do Proxmox:
1. Download & Extract

Download and extract the file on your Proxmox host (or upload the extracted version):

wget https://github.com/home-assistant/operating-system/releases/download/16.0/haos_ova-16.0.qcow2.xz
unxz haos_ova-16.0.qcow2.xz


Criar VM no Proxmox:
em OS: Do not use any media
machine: q35
BIOS: OVMF (UEFI), add EFI disk, selecionar disco, desmarcar pre-enroll keys
SCSI Controller: VirtIO SCSI
selecionar Qemu Agent
desmarcar Add TPM
Bus/Device: SCSI
Disk size: 64 ou 96
Marcar SSD emulation, Discard, IO Thread, Backup
cache: no cache
desmarcar read-only, skip replication
async IO: default IO_uRing
CPU: 2 cores, Type Host
Memoria: 4096, desativar balloning
network: desativar firewall
desmarcar start at boot

No shell do Proxmox:
qm importdisk 100 haos_ova-16.0.qcow2 local-zfs 
(mudar 100 para o numero da VM, talvez seja 100 mesmo, e local-zfs para outro se a instalação do Proxmox foi feita em ext4)

✅ After that:
Go to VM > Hardware
You'll see an "Unused Disk" — click Edit
colocar as mesmas opções que descrevi acima
esse disco vai ser nomeado como SCSI1, mas deveria ser SCSI0, entao:
clique no disco SCSI0, detach e depois apague esse disco

no shell do proxmox:
nano /etc/pve/qemu-server/100.conf (mudar 100 para o numero da VM, talvez seja 100 mesmo)
mudar a linha SCSI1 para:
scsi0: local-zfs:vm-100-disk-2,discard=on,iothread=1,size=96G,ssd=1
Note que mudamos para scsi0 e também alteramos o tamanho do disco para size=96G
Salve com ctrl+x, y, enter

Move this disk to the top of Boot Order (in Options tab in VM)
Set boot order
Go to VM > Options > Boot Order
Ensure your disk (scsi0) is listed first

Marcar start at boot
Iniciar VM


