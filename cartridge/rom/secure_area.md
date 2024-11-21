# セキュアエリア

セキュアエリアはROMデータの`0x4000..7FFF`にあり、通常のプログラムコードやデータを格納することができますが、ARM9のブートコードにのみ使用でき、ARM7のブートコード、アイコン/タイトル、ファイルシステム、その他のデータには使用できません。

## サイズ

セキュアエリアは、ARM9ブートコードROMのソースアドレス(`=src`)が`0x4000..7FFF`内にある場合に存在し、その場合、(BIOSがKEY1暗号化コマンドを使って)srcを起点に`1000h`ずつアライメントされた`7FFFh`アドレスまでの4KB分割でロードされます。

したがって、ヘッダのARM9ブートコードのサイズエントリに関係なく、セキュアエリアのサイズは`8000h-src`となります。

Note: The BIOS silently skips any NDS9 bootcode at src<4000h.

Cartridges with src>=8000h do not have a secure area.

## セキュアエリアID

セキュアエリアの最初の8バイトにはセキュアエリアIDが含まれます。IDは必須であり（BIOSブートコードによって検証される）、ID値はブートプロセス中に変更されます。

```
  Value                Expl.
  "encryObj"           raw ID before encryption (raw ROM-image)
  (encrypted)          encrypted ID after encryption (encrypted ROM-image)
  "encryObj"           raw ID after decryption (verified by BIOS boot code)
  E7FFDEFFh,E7FFDEFFh  destroyed ID (overwritten by BIOS after verify)
```

- 復号化されたIDが一致する場合: BIOS は最初の8バイトを `E7FFDEFFh,E7FFDEFFh` で上書きします。つまり、ID部分だけが破壊され、ARM9のブートコードは無事です
- IDが一致しない場合: BIOSによって、最初の 2KB が E7FFDEFFh 値で上書き(破壊)されます。つまり、ARM9のブートコードも壊れます。

## セキュアエリアの先頭2KBについて

セキュアエリアの最初の2KB（存在する場合）はKEY1で暗号化されています。市販のゲームでは、この2KBの領域に以下のようなデータが格納されています。

(複合化された状態です)

```
  000h..007h  セキュアエリアID (see above)
  008h..00Dh  固定値 (FFh,DEh,FFh,E7h,FFh,DEh)
  00Eh..00Fh  この後の 0x7E0 バイトの CRC16 [010h..7FFh]
  010h..7FDh  Unknown/random values, mixed with some THUMB SWI calls
  7FEh..7FFh  固定値 (00h,00h)
```

このうち、最初の8バイトのIDだけが検証されます。

BIOSも(現在の)ファームウェアのバージョンも`008h.7FFh`のデータを検証していないため、`008h.7FFh`の`0x7F8`バイトは通常のプログラムコード/データにも使用される可能性があります。

## Avoiding Secure Area Encryption

WLAN files are reportedly same format as cartridges, but without Secure Area, so games with Secure Area cannot be booted via WLAN. No$gba can encrypt and decrypt Secure Areas only if the NDS BIOS-images are present. And, Nintendo’s devkit doesn’t seem to support Secure Area encryption of unreleased games.

So, unencrypted cartridges are more flexible in use. Ways to avoid encryption (which still work on real hardware) are:

1) Set NDS9 ROM offset to 4000h, and leave the first 800h bytes of the Secure Area 00h-filled, which can be (and will be) safely destroyed during loading; due to the missing “encryObj” ID; that method is used by Nintendo’s devkit.

2) Set NDS9 ROM offset to 8000h or higher (cartridge has no Secure Area at all).

3) Set NDS9 ROM offset, RAM address, and size to zero, set NDS7 ROM offset to 200h, and point both NDS9 and NDS7 entrypoints to the loaded NDS7 region. That method avoids waste of unused memory at 200h..3FFFh, and it should be compatible with the NDS console, however, it is not comaptible with commercial cartridges - which do silently redirect address below 4000h to “addr=8000h+(addr AND 1FFh)”. Still, it should work with inofficial flashcards, which do not do that redirection. No$gba emulates the redirection for regular official cartridges, but it disables redirection for homebrew carts if NDS7 rom offset<8000h, and NDS7 size>0.

[One possible problem: Newer “anti-passme” firmware versions reportedly check that the entrypoint isn’t set to 80000C0h, that firmwares might also reject NDS9 entrypoints within the NDS7 bootcode region?]


