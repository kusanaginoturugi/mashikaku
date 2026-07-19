# mashikaku

## VIA on Linux

このキーボードは VIA 公式データベースに入っていないため、usevia.app では
`mashikaku.json` を Design タブから読み込ませる必要がある。

### 手順

1. usevia.app を開く。
2. Settings で `Show Design tab` を有効にする。
3. Design タブで `Load Draft Definition` から `mashikaku.json` を読み込む。
4. Configure タブに戻り、`MASHIKAKU` に接続する。

### Arch Linux / hidraw permission

VIA が WebHID 経由で `mashikaku` を認識しても、次のようなエラーが出る場合がある。

```text
VIA は mashikaku の V3 定義を見つけられませんでした
mashikaku は VIA 対応キーボードとして応答していないようです
```

この場合、`/dev/hidraw*` の権限不足で Chrome/Chromium が VIA 用 HID に触れて
いない可能性が高い。

一時確認:

```sh
chmod a+rw /dev/hidrawX
```

`hidrawX` は `Ratata mashikaku` に対応するものを指定する。

対象確認:

```sh
for d in /sys/class/hidraw/hidraw*; do
  echo "$d"
  udevadm info -q property -p "$d/device" | rg 'HID_NAME|HID_ID'
done
```

恒久対応:

```sh
printf '%s\n' 'KERNEL=="hidraw*", ATTRS{idVendor}=="fef4", ATTRS{idProduct}=="00f4", MODE="0666", TAG+="uaccess"' > /etc/udev/rules.d/99-mashikaku.rules
udevadm control --reload-rules
udevadm trigger
```

その後、USB を抜き差しする。

### BOOTSEL

RP2040 の `BOOTSEL` はファームウェア書き込みモードに入るためのボタン。
`BOOTSEL` を押しながら USB 接続すると `RPI-RP2` としてマウントできる。

このドライブには現在の firmware は表示されない。見えるのは通常これだけ。

```text
INDEX.HTM
INFO_UF2.TXT
```

firmware を焼くときは、このドライブに `mashikaku_via.uf2` をコピーする。

```sh
cp mashikaku_via.uf2 /mnt/
sync
umount /mnt
```

通常起動時に見える USB ID:

```text
fef4:00f4 Ratata mashikaku
```
