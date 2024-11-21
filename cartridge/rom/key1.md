# KEY1暗号化

KEY1の暗号化はゲームコード(またはファームウェアのIDコード)にのみ依存し、ランダムな要素は含まれていません。KEY1暗号化コマンドがランダムに見えるのは、<unencrypted>コマンドにランダムな値が含まれているため、暗号化結果がランダムに見えるだけです。

KEY1暗号化はKEY1暗号化されたゲームカートコマンド(セキュアエリアのロードなど)に使用されます。また、セキュアエリアの最初の2KBの復号化、ファームウェアの復号化、ゲームカート/ファームウェアヘッダの暗号化された値の復号化にも使用されます。

KEY1暗号には、Blowfish暗号を使用しています。

## Initial Encryption Values

後述の計算式は、NDS/DSi BIOSから1048hバイトのキーテーブルをコピーした場合のみ使用できます。

```
  NDS.ARM7 ROM: 00000030h..00001077h (values 99 D5 20 5F ..) Blowfish/NDS-mode
  DSi.ARM9 ROM: FFFF99A0h..FFFFA9E7h (values 99 D5 20 5F ..) ""
  DSi.TCM Copy: 01FFC894h..01FFD8DBh (values 99 D5 20 5F ..) ""
  DSi.ARM7 ROM: 0000C6D0h..0000D717h (values 59 AA 56 8E ..) Blowfish/DSi-mode
  DSi.RAM Copy: 03FFC654h..03FFD69Bh (values 59 AA 56 8E ..) ""
  DSi.Debug:    (stored in launcher) (values 69 63 52 05 ..) Blowfish/DSi-debug
```

The DSi ROM sections are disabled after booting, but the RAM/TCM copies can be dumped (eg. with some complex main memory hardware mods, or via unlaunch exploit). The DSi.Debug key is stored in launcher, and it’s used when SCFG_OP is nonzero (as so on debugging on hardware).

## encrypt_64bit(ptr) / decrypt_64bit(ptr)

```
  Y=[ptr+0]
  X=[ptr+4]
  FOR I=0 TO 0Fh (encrypt), or FOR I=11h TO 02h (decrypt)
    Z=[keybuf+I*4] XOR X
    X=[keybuf+048h+((Z SHR 24) AND FFh)*4]
    X=[keybuf+448h+((Z SHR 16) AND FFh)*4] + X
    X=[keybuf+848h+((Z SHR  8) AND FFh)*4] XOR X
    X=[keybuf+C48h+((Z SHR  0) AND FFh)*4] + X
    X=Y XOR X
    Y=Z
  NEXT I
  [ptr+0]=X XOR [keybuf+40h] (encrypt), or [ptr+0]=X XOR [keybuf+4h] (decrypt)
  [ptr+4]=Y XOR [keybuf+44h] (encrypt), or [ptr+4]=Y XOR [keybuf+0h] (decrypt)
```

## apply_keycode(modulo)

```
  encrypt_64bit(keycode+4)
  encrypt_64bit(keycode+0)
  [scratch]=0000000000000000h   ;S=0 (64bit)
  FOR I=0 TO 44h STEP 4         ;xor with reversed byte-order (bswap)
    [keybuf+I]=[keybuf+I] XOR bswap_32bit([keycode+(I MOD modulo)])
  NEXT I
  FOR I=0 TO 1040h STEP 8
    encrypt_64bit(scratch)      ;encrypt S (64bit) by keybuf
    [keybuf+I+0]=[scratch+4]    ;write S to keybuf (first upper 32bit)
    [keybuf+I+4]=[scratch+0]    ;write S to keybuf (then lower 32bit)
  NEXT I
```

## init_keycode(idcode,level,modulo,key)

```
  if key=nds then copy [nds_arm7bios+0030h..1077h] to [keybuf+0..1047h]
  if key=dsi then copy [dsi_arm7bios+C6D0h..D717h] to [keybuf+0..1047h]
  [keycode+0]=[idcode]
  [keycode+4]=[idcode]/2
  [keycode+8]=[idcode]*2
  IF level>=1 THEN apply_keycode(modulo) ;first apply (always)
  IF level>=2 THEN apply_keycode(modulo) ;second apply (optional)
  [keycode+4]=[keycode+4]*2
  [keycode+8]=[keycode+8]/2
  IF level>=3 THEN apply_keycode(modulo) ;third apply (optional)
```

## firmware_decryption

```
  init_keycode(firmware_header+08h,1,0Ch,nds) ;idcode (usually "MACP"), level 1
  decrypt_64bit(firmware_header+18h)          ;rominfo
  init_keycode(firmware_header+08h,2,0Ch,nds) ;idcode (usually "MACP"), level 2
  decrypt ARM9 and ARM7 bootcode by decrypt_64bit (each 8 bytes)
  decompress ARM9 and ARM7 bootcode by LZ77 function (swi)
  calc CRC16 on decrypted/decompressed ARM9 bootcode followed by ARM7 bootcode
```

Note: The sizes of the compressed/encrypted bootcode areas are unknown (until they are fully decompressed), one way to solve that problem is to decrypt the next 8 bytes each time when the decompression function requires more data.


## gamecart_decryption

```
  init_keycode(cart_header+0Ch,1,08h,nds)   ;gamecode, level 1, modulo 8
  decrypt_64bit(cart_header+78h)            ;rominfo (secure area disable)
  init_keycode(cart_header+0Ch,2,08h,nds)   ;gamecode, level 2, modulo 8
  encrypt_64bit all NDS KEY1 commands (1st command byte in MSB of 64bit value)
  after loading the secure_area, calculate secure_area crc, then
  decrypt_64bit(secure_area+0)              ;first 8 bytes of secure area
  init_keycode(cart_header+0Ch,3,08h,nds)   ;gamecode, level 3, modulo 8
  decrypt_64bit(secure_area+0..7F8h)        ;each 8 bytes in first 2K of secure
  init_keycode(cart_header+0Ch,1,08h,dsi)   ;gamecode, level 1, modulo 8
  encrypt_64bit all DSi KEY1 commands (1st command byte in MSB of 64bit value)
```

セキュアエリアの復号後、最初の8バイトのIDフィールドは`"encryObj"`であるべきで、もしそれが一致すれば、最初の8バイトは`E7FFFDEFFh`で埋められ、そうでなければ2KB全体がその値で埋められます。

## Gamecart Command Register

Observe that the byte-order of the command register [40001A8h] is reversed. The way how the CPU stores 64bit data in memory (and the way how the “encrypt_64bit” function for KEY1-encrypted commands expects data in memory) is LSB at [addr+0] and MSB at [addr+7]. This value is to be transferred MSB first. However, the DS hardware transfers [40001A8h+0] first, and [40001A8h+7] last. So, the byte order must be reversed when copying the value from memory to the command register.


