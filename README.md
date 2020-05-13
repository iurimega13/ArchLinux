# ArchLinux
##Instalação arch linux
 
### Teclado abnt2
Setar layout br-abnt2 para o teclado
```bash
# loadkeys br-abnt2
```

### Conexão com a internet 
Ethernet:
```bash
# systemctl start dhcpcd
# ping -c3 google.com
```
Wifi:
```bash
# wifi-menu
# ping -c3 www.google.com
```

### Particionamento de disco
__Particionar Disco (BIOS)__

Aconcelhavel dar:
* /swap = 4gb
* /raiz = Todo o restante do HD

Agora vamos usar o cfdisk para para começar a criação das partições.
```bash
# cfdisk /dev/sdX
```
**(:warning: Substitua o X pela letra do seu disco rígido ex: “sda” :warning:)**

A interface do cfdisk é bem simples, basta selecionar o tipo DOS criar uma partição que vai conter o tamanho total do seu HD para a raiz e em seguida tornar essa partição bootable, a partição swap não é necessária, só vem a ser útil se você tiver pouca memória ram.

Acompanhe o processo abaixo:

![](/imgs/bios.gif)

__Particionar Disco (UEFI)__

Aconcelhavel dar:

* /boot = 300mb
* /swap = 4gb
* /raiz = Todo o restante do HD
* /home (Se case quiser deixe 30Gb para raiz e o resto do hd para home)

Agora vamos usar o cfdisk para para começar a criação das partições.
```bash
# cfdisk /dev/sdX
```
**(:warning: Substitua o X pela letra do seu disco rígido ex: “sda” :warning:)**

Faça a mesma coisa de como foi para bios só que não vai ser colocado o tipo da tabela de partiçoes.

### Formatar as partições
**(:warning: Substitua o X pela letra da partição ex: “sda1” :warning:)**
Formatar Root (BIOS):
```bash
# mkfs.ext4 /dev/sdax
```

Formatar Swap (BIOS/UEFI):
```bash
# mkswap /dev/sdax
# swapon /dev/sdax
```
Formatar Boot (UEFI):
```bash
# mkfs.vfat -F32 /dev/sdax
```
Formatar Raiz/home() (UEFI):
```bash
# mkfs.ext4 /dev/sdax
```

### Montando as partições

**(:warning: Substitua o X pela letra da partição ex: “sda1” :warning:)**

Montar Root (BIOS):
```bash
# mount -t ext4 /dev/sdax /mnt
```

Montar Boot (UEFI):
```bash
# mkdir -p /mnt/boot
# mount /dev/sdax /mnt/boot
```

Montar Root (UEFI):
```bash
# mount /dev/sdax /mnt
```

Montar Home(Se houver) (UEFI):
```bash
# mkdir -p /mnt/home
# mount /dev/sdax /mnt/home
```

### Escolhendo mirrors

Escolher a lista de espelhos do brasil:
```bash
# pacman -Sy reflector
# reflector --country Brazil --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```
### Instalar os pacotes bases do sistema

Instalando a base do sistema:
```bash
# pacstrap -i /mnt base base-devel linux linux-firmware nano dhcpcd(Caso for usar internet cabeada)
```

### Configurando o FSTAB

Para configurar fstab (tabela de sistemas de arquivos) execute:
```bash
# genfstab -U -p /mnt >> /mnt/etc/fstab
```

### Entrar no novo sistema

Agora é hora de entrar no diretório root recém-instalado para configurá-lo:
```bash
# arch-chroot /mnt
# loadkeys br-abnt2 (para usar o layout abnt2)
```

### Configurando KEYMAP

A variável KEYMAP é especificada no arquivo /etc/vconsole.conf . Ele define qual layout de teclado, será usado nos consoles virtuais. Execute este comando:
```bash
# echo -e 'KEYMAP="br-abnt2.map.gz"' > /etc/vconsole.conf
```

### Configuração do idioma e fuso horário

Para configurar o idioma do sistema, execute o seguinte comando:
```bash
# nano /etc/locale.gen
```
Procure as linhas e descomente-as:
* pt_BR.UTF-8
* pt_BR ISO-8859-1

Depois execute:
```bash
# locale-gen
# echo LANG=pt_BR.UTF-8 > /etc/locale.conf
# export LANG=pt_BR.UTF-8
```
Para ver todos os fusos horários disponíveis da América:


```bash
# ls /usr/share/zoneinfo/America

```

Agora você pode configurar a sua zona:
```bash
# ln -sf /usr/share/zoneinfo/America/Fortaleza /etc/localtime
```

Vamos agora configurar o relógio do hardware, apenas no caso de termos uma data errada:
```bash
# hwclock --systohc --utc
```

Para conferir se a hora está certa:
```bash
# date
```

### Configurando o repositório
Execute esse comando e descomente as linhas a seguir:
```bash
# nano /etc/pacman.conf
```
* [multilib]
* Include = /etc/pacman.d/mirrorlist

E depois atualize os repositórios:
```bash
# pacman -Sy
```

### Definir hostname

Agora você vai setar o nome que você deseja ter na sua maquina, basta trocar o “hostname” pelo nome que quiser, mas ele não pode conter espaços.
```bash
# echo hostname > /etc/hostname
```

### Configurando a conexão
Ethernet:

```bash
# systemctl enable dhcpcd
```
Wifi:

```bash
# pacman -S wpa_supplicant dialog iw networkmanager
# systemctl enable NetworkManager
```

### Criando usuário
Adicionado Usuario, troque "username" pelo seu nome de usuário:

```bash
# useradd -m -g users -G wheel,storage,power -s /bin/bash username
```

Em seguida, forneça a senha para este novo usuário executando:
```bash
# passwd username
```

Não se esqueça de definir também a senha para o usuário root:
```bash
# passwd
```

Permitir que os usuários no grupo wheel, sejam capazes de executar tarefas administrativas com o sudo:
```bash
# sed -i '/%wheel ALL=(ALL) ALL/s/^#//' /etc/sudoers
```

### Instalar GRUB
Instalar e configurar o GRUB (BIOS):

```bash
# mkinitcpio -p linux
# pacman -S grub
# grub-install /dev/sdX
# pacman -S os-prober (Se você estiver inicializando em dual boot)
# pacman -S intel-ucode (Se você tiver uma CPU Intel, instale o pacote intel-ucode)
# pacman -S amd-ucode (Se você tiver uma CPU amd, instale o pacote amd-ucode)
# grub-mkconfig -o /boot/grub/grub.cfg
```
🔶 Instalar e configurar o boot-loader (UEFI)

```bash
# mkinitcpio -p linux
# pacman -S grub efibootmgr
# grub-install /dev/sdX --efi-directory=/boot
# pacman -S os-prober (Se você estiver inicializando em dual boot)
# pacman -S intel-ucode (Se você tiver uma CPU Intel, instale o pacote intel-ucode)
# pacman -S amd-ucode (Se você tiver uma CPU amd, instale o pacote amd-ucode)
# grub-mkconfig -o /boot/grub/grub.cfg
```

### Desmontar e reiniciar
Desmonte as partições e reinicie para poder ir para o próximo passo, a pós instalação.
```bash
# exit
# umount -R /mnt
# reboot
```