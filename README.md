# Arch Linux 最小インストール
日本語キーボード読み込み

```
loadkeys jp106
```

Wi-Fi接続

```
iwctl station wlan0 connect {SSID}
```

パーティション切り(boot, root, homeを作成)

```
gdisk /dev/sda
```

フォーマット（例）

```
mkfs.fat -F32 /dev/sda1
mkfs.btrfs -f /dev/sda2
mkfs.btrfs -f /dev/sda3
```

マウント（例）

```
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
mount --mkdir /dev/sda3 /mnt/home
```

ベースシステムインストール（カーネルはお好みのものを）

```
pacstrap /mnt base base-devel linux linux-firmware intel-ucode btrfs-progs neovim dosfstools networkmanager fish
```

スワップファイル作成(1GiB)
```
btrfs subvolume create /swap
btrfs filesystem mkswapfile --size 4g --uuid clear /mnt/swap/swapfile
chmod 600 /mnt/swap/swapfile
swapon /mnt/swap/swapfile
```

fstab生成

```
genfstab -U /mnt >> /mnt/etc/fstab
```

pacstrapでインストールしたシステムに入る

```
arch-chroot /mnt
```

locale.gen編集（en_US.UTF-8とja_JP.UTF-8をアンコメント）

```
nvim /etc/locale.gen
```

ロケール生成

```
locale-gen
```

使用言語、キーボードの設定

```
echo LANG=en_US.UTF-8 > /etc/locale.conf
echo KEYMAP=jp106 > /etc/vconsole.conf
```

タイムゾーン設定

```
ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
hwclock --systohc --utc
```

ホストネーム設定

```
echo {hostname} > /etc/hostname
```

NetworkManager有効化
```
systemctl enable NetworkManager
```

systemd-bootをインストール

```
bootctl install
```

/boot/loader/entries/zen.confに以下を追記

```
title Arch Linux (linux)
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=/dev/sda2 rw
```

rootユーザーのパスワードを変更

```
passwd
```

非rootユーザー作成
```
useradd -m -G wheel -s $(which fish) {username}
passwd {username}
```

visudoの設定
```
EDITOR=nvim visudo
```

`# Defaults env_keep += "HOME"`

`# %wheel ALL=(ALL:ALL) NOPASSWD: ALL`

上記２つをアンコメント

`# Defaults env_keep += "EDITOR"`

`# Defaults env_keep += "VISUAL"`

上記２つを追加

`exit`でchrootを抜け、`poweroff`で電源を落としインストールメディアを抜き再度起動
