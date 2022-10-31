# ayakurume
- full rootfs r/w (fakefs), tweak injection etc...


## 注意
間違いがある可能性大、serialでデバッグ推奨(デバイス側のverbose bootだとSpringBoardが起動すると追えなくなるため)

## サポート環境
- iPhone 6s (iPhone8,1/N71AP) 15.7.1

# 使うもの
- [gaster](https://github.com/0x7ff/gaster)
- [libirecovery](https://github.com/libimobiledevice/libirecovery)
- [SSHRD_Script](https://github.com/verygenericname/SSHRD_Script)
- [libusbmuxd](https://github.com/libimobiledevice/libusbmuxd)

- [bootstrap-ssh.tar](https://cdn.discordapp.com/attachments/1017153024768081921/1026161261077090365/bootstrap-ssh.tar)
- [org.swift.libswift_5.0-electra2_iphoneos-arm.deb](https://github.com/coolstar/Odyssey-bootstrap/raw/master/org.swift.libswift_5.0-electra2_iphoneos-arm.deb)
- [com.ex.substitute_2.3.1_iphoneos-arm.deb](https://apt.bingner.com/debs/1443.00/com.ex.substitute_2.3.1_iphoneos-arm.deb)
- [com.saurik.substrate.safemode_0.9.6005_iphoneos-arm.deb](https://apt.bingner.com/debs/1443.00/com.saurik.substrate.safemode_0.9.6005_iphoneos-arm.deb)

# 手順
## sshrdで必要なものをセットアップ
- macos side
```
cd SSHRD_Script/
./sshrd.sh 15.7.1
./sshrd.sh boot
./sshrd.sh ssh
```
- ios side
```
newfs_apfs -A -D -o role=r -v System /dev/disk0s1
mount_apfs /dev/disk0s1s1 /mnt1
mount_apfs /dev/disk0s1s8 /mnt2
mount_apfs /dev/disk0s1s6 /mnt6
cp -a /mnt1/. /mnt2/
umount /mnt1
mkdir /mnt6/{UUID}/binpack
mkdir /mnt2/jbin
```
- macos side
```
scp -P 2222 lightstrap.tar root@localhost:/mnt6/
scp -P 2222 jb.dylib jbloader launchd root@localhost:/mnt2/jbin/
```
- ios side
```
tar -xvf /mnt6/lightstrap.tar -C /mnt6/{UUID}/binpack/
rm /mnt6/lightstrap.tar
```

sshpassなどを使い、`/mnt6/{UUID}/System/Library/Caches/apticket.der`(以下`apticket.der`として使用)をmac側にコピーする

- macos side
```
./img4 -i kernelcache.release.n71 -o kernelcachd -P kc.bpatch -M apticket.der
scp -P {port} kernelcachd root@localhost:/mnt6/{UUID}/System/Library/Caches/com.apple.kernelcaches/kernelcachd
```

- ios side
```
reboot
```

## 初回起動前準備
- macos side
```
./gaster pwn
./gaster decrypt iBSS.n71.RELEASE.im4p iBSS.n71.RELEASE.dec
./gaster decrypt iBoot.n71.RELEASE.im4p iBoot.n71.RELEASE.dec
bspatch iBSS.n71.RELEASE.dec pwniBSS.dec iBSS.n71.RELEASE.patch
bspatch iBoot.n71.RELEASE.dec pwniBoot.dec iBoot.n71.RELEASE.patch
./img4 -i pwniBSS.dec -o iBSS.img4 -M apticket.der -A -T ibss
./img4 -i pwniBoot.dec -o iBoot.img4 -M apticket.der -A -T ibec
```
*iBootのboot-argsは`rd=disk0s1s8 serial=3`に設定されています。(必要に応じて改変ok)

## 初回起動
- macos side
```
./gaster pwn
irecovery -f iBSS.img4
irecovery -f iBoot.img4
```

dropbearの起動を確認後
- macos side
```
iproxy {port} 44
```
```
ssh root@localhost -p {port}
scp -P {port} bootstrap-ssh.tar root@localhost:/var/root 
scp -P {port} org.swift.libswift_5.0-electra2_iphoneos-arm.deb root@localhost:/var/root 
scp -P {port} com.ex.substitute_2.3.1_iphoneos-arm.deb root@localhost:/var/root 
scp -P {port} com.saurik.substrate.safemode_0.9.6005_iphoneos-arm.deb root@localhost:/var/root 
```

- ios side
```
mount -uw /
cd /var/root
tar --preserve-permissions --no-overwrite-dir -xvf bootstrap-ssh.tar -C /
/prep_bootstrap.sh
apt update
apt upgrade -y
apt install org.coolstar.sileo
dpkg -i *.deb
rm *.deb
rm bootstrap-ssh.tar
touch /.installed_ayakurume
reboot
```

## 起動
- macos side
```
./gaster pwn
irecovery -f iBSS.img4
irecovery -f iBoot.img4
```

# credit
- launchd hook: [LinusHenze's fugu](https://github.com/LinusHenze/Fugu)
- jbinit: [tihmstar](https://github.com/tihmstar/jbinit)
- img4lib: [xerub](https://github.com/xerub/img4lib)
- bootstrap: [ProcursusTeam](https://github.com/ProcursusTeam)
- bootstrap: [checkra1n](https://github.com/checkra1n)