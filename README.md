# Este guia é uma fork traduzida do guia feito por korvahannu com o intuito de ajudar os usuarios brasileiros de arch-linux
# Link do guia original... https://github.com/korvahannu/arch-nvidia-drivers-installation-guide

# Guia de instalação dos drivers da NVIDIA no Arch Linux

![Arch Linux Logo](https://archlinux.org/static/logos/archlinux-logo-dark-90dpi.ebdee92a15b3.png)

## Links importantes

- [Arch Linux Homepage](https://archlinux.org/ "Página inicial, site do Arch Linux")
- [Arch Linux Wiki](https://wiki.archlinux.org/ "Arch Wiki")

## Pré-requisitos padrões

Este é um rapido tutorial de como você pode instalar os drivers proprietários da NVIDIA para o Arch Linux. Por favor, caso você esteja usando qualquer outro kernal que não seja o kernel padrão do linux, como o linux-lts, você deverá fazer mudanças de acordo. Todos os comandos marcados `assim` devem ser rodados no terminal. **Não reinicie nem desligue o computador antes de terminar todos os passos abaixo!**

## Passo 1: Instalar os pacotes necessários e habilitar a multilib

1. Atualize o sistema:
   `sudo pacman -Syu`
2. Instale os pacotes necessários:
   `sudo pacman -S base-devel linux-headers git nano --needed`
3. Instale o ajudante AUR, yay:
   - `cd ~`
   - `git clone https://aur.archlinux.org/yay.git`
   - `cd yay`
   - `makepkg -si`
4. Habilite o repositório multilib:
   - `sudo nano /etc/pacman.conf`
   - Descomente as linhas a seguir removendo o caractere "#" no começo da linha
     - **[multilib]**
     - **Include = /etc/pacman.d/mirrorlist**
   - Salve o arquivo com _CTRL+S_ e feche o nano com _CTRL+X_
5. Rode `yay -Syu` para atualizar a database de pacotes

## Passo 2: Instalando os pacotes dos drivers

1. Primeiro encontre sua [NVIDIA GPU nesta lista aqui](https://nouveau.freedesktop.org/CodeNames.html). Alternativamente, você pode tambem olhar na [Gentoo wiki](https://wiki.gentoo.org/wiki/NVIDIA#Feature_support).
2. Cheque que pacote de drivers você precisa instalar na lista abaixo:

| Nome do Driver                                      | Kernel                      | Base driver       | OpenGL             | OpenGL (multilib)        |
| ------------------------------------------------ | --------------------------- | ----------------- | ------------------ | ------------------------ |
| Maxwell (NV110) series ou mais novo                 | linux or linux-lts          | nvidia            | nvidia-utils       | lib32-nvidia-utils       |
| Maxwell (NV110) series ou mais novo                 | qualquer a não ser linux e linux-lts | nvidia-dkms       | nvidia-utils       | lib32-nvidia-utils       |
| Kepler (NVE0) series                             | qualquer                         | nvidia-470xx-dkms | nvidia-470xx-utils | lib32-nvidia-470xx-utils |
| GeForce 400/500/600 series cards [NVCx and NVDx] | qualquer                         | nvidia-390xx-dkms | nvidia-390xx-utils | lib32-nvidia-390xx-utils |
| Tesla (NV50/G80-90-GT2XX)                        | qualquer                         | nvidia-340xx-dkms | nvidia-340xx-utils | lib32-nvidia-340xx-utils |

3. Instale os pacotes corretos de Base driver, OpenGL e OpenGL (multilib)
   - Exemplo: `yay -S nvidia-470xx-dkms nvidia-470xx-utils lib32-nvidia-470xx-utils`
4. Instale o nvidia-settings com: `yay -S nvidia-settings`

## Passo 3: Habilite a configuração DRM kernel mode

Neste passo, por favor, siga as seguintes partes: _Configurando o parâmetro do Kernel_, _Adicionando Early Loading dos módulos NVIDIA_, e _Adicionando o Hook do Pacman_.

### Configurando o parâmetro do Kernel:

Conforme o bootloader que você usa, siga a opção A ou B. Depois disso, continue para _Adicionando Early Loading dos módulos NVIDIA_

#### Opção A) Para usuários de GRUB

1. Edite o arquivo de configuração do GRUB:

   - `sudo nano /etc/default/grub`
   - Encontre a linha com **GRUB_CMDLINE_LINUX_DEFAULT**
   - Adicione dentro dos parênteses da linha: **nvidia-drm.modeset=1**
     - Exemplo: **GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia-drm.modeset=1"**
   - Salve o arquivo com _CTRL+S_ e feche o nano com _CTRL+X_
2. Atualize as configurações do GRUB: `sudo grub-mkconfig -o /boot/grub/grub.cfg`

#### Opção B) Para usuáriios de systemd-boot

1. Navegue até o diretório de entries do bootloader: `cd /boot/loader/entries/`
2. Edite o arquivo **.conf** apropriado para a boot entry do Arch Linux
   - `sudo nano <filename>.conf`
3. Adicione **nvidia-drm.modeset=1** para a linha contendo **options**
4. Salve o arquivo com _CTRL+S_ e feche o nano com _CTRL+X_

### Adicionando os Módulos NVIDIA no Early Loading:

1. Edite o arquivo de configurações **mkinitcpio**:
   - `sudo nano /etc/mkinitcpio.conf`
   - Encontre a linha que diz **MODULES=()**
   - Atualize a linha para: **MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)**
   - Encontre a linha que diz **HOOKS=()**
   - Na linha contendo **HOOKS=()**, encontre a palavra **kms** dentro dos parênteses e a remova
   - Salve o arquivo com _CTRL+S_ e feche o nano com _CTRL+X_
2. Regenere a initramfs com: `sudo mkinitcpio -P`

### Adicionando o Hook do Pacman:

1. Baixe o arquivo **nvidia.hook** deste repositório
   - `cd ~`
   - `wget https://raw.githubusercontent.com/korvahannu/arch-nvidia-drivers-installation-guide/main/nvidia.hook`
2. Abra o arquivo com seu editor de preferência.
   - `nano nvidia.hook`
3. Encontre a linha que diz **Target=nvidia**.
4. Troque a palavra **nvidia** com o base driver que você instalou, exemplo: **nvidia-470xx-dkms** (Edit do tradutor, esta linha implica que este passo é obrigatório, mas não é, se você instalou o pacote "nvidia" pule este passo)
   - A linha editada deve ficar mais ou menos assim: **Target=nvidia-470xx-dkms**
5. Salve o arquivo com _CTRL+S_ e feche o nano com _CTRL+X_
6. Mova o arquivo **/etc/pacman.d/hooks/** usando: `sudo mv ./nvidia.hook /etc/pacman.d/hooks/`

## Passo 4: Reinicie e aproveite!

Agora você pode reiniciar de forma segura e aproveitar seus drivers NVIDIA proprietários. Se você tiver algum problema, lembre-se de checar a Wiki do Arch Linux ou os Forums para respostas de erros e perguntas comuns