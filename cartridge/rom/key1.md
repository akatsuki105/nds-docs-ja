# KEY1暗号化

KEY1の暗号化はゲームコード(またはファームウェアのIDコード)にのみ依存し、ランダムな要素は含まれていません。KEY1暗号化コマンドがランダムに見えるのは、<unencrypted>コマンドにランダムな値が含まれているため、暗号化結果がランダムに見えるだけです。

KEY1暗号化はKEY1暗号化されたゲームカートコマンド(セキュアエリアのロードなど)に使用されます。また、セキュアエリアの最初の2KBの復号化、ファームウェアの復号化、ゲームカート/ファームウェアヘッダの暗号化された値の復号化にも使用されます。

KEY1暗号には、Blowfish暗号を使用しています。

## Initial Encryption Values

後述の計算式は、NDS/DSi の BIOS から4168バイトのキーテーブル(`KeyBuf`)をコピーした場合のみ使用できます。

```
  NDS.ARM7 BIOS: 00000030h..00001077h (values 99 D5 20 5F ..) Blowfish/NDS-mode
  DSi.ARM9 BIOS: FFFF99A0h..FFFFA9E7h (values 99 D5 20 5F ..) ""
  DSi.TCM Copy:  01FFC894h..01FFD8DBh (values 99 D5 20 5F ..) ""
  DSi.ARM7 BIOS: 0000C6D0h..0000D717h (values 59 AA 56 8E ..) Blowfish/DSi-mode
  DSi.RAM Copy:  03FFC654h..03FFD69Bh (values 59 AA 56 8E ..) ""
  DSi.Debug:     (stored in launcher) (values 69 63 52 05 ..) Blowfish/DSi-debug
```

The DSi BIOS sections are disabled after booting, but the RAM/TCM copies can be dumped (eg. with some complex main memory hardware mods, or via unlaunch exploit). The DSi.Debug key is stored in launcher, and it’s used when SCFG_OP is nonzero (as so on debugging on hardware).

## `encrypt_64bit(u32* ptr)`

```c
  // KeyBuf は u32[1042]
  u32 y=ptr[0];
  u32 x=ptr[1];
  
  for (u32 i = 0; i <= 15; i++) {
    u32 z = KeyBuf[i] ^ x;

    x =  KeyBuf[ 18 +        ((z >> 24) & 0xFF)];
    x += KeyBuf[(18 + 256) + ((z >> 16) & 0xFF)];
    x ^= KeyBuf[(18 + 512) + ((z >>  8) & 0xFF)];
    x += KeyBuf[(18 + 768) + ((z >>  0) & 0xFF)];

    x ^= y;
    y =  z;
  }

  ptr[0] = x ^ KeyBuf[0x10];
  ptr[1] = y ^ KeyBuf[0x11];
```

## `decrypt_64bit(u32* ptr)`

```c
  // KeyBuf は u32[1042]
  u32 y=ptr[0];
  u32 x=ptr[1];
  
  for (u32 i = 17; i >= 2; i--) {
    u32 z = KeyBuf[i] ^ x;

    x =  KeyBuf[ 18 +        ((z >> 24) & 0xFF)];
    x += KeyBuf[(18 + 256) + ((z >> 16) & 0xFF)];
    x ^= KeyBuf[(18 + 512) + ((z >>  8) & 0xFF)];
    x += KeyBuf[(18 + 768) + ((z >>  0) & 0xFF)];

    x ^= y;
    y =  z;
  }

  ptr[0] = x ^ KeyBuf[0x1];
  ptr[1] = y ^ KeyBuf[0x0];
```

## `apply_keycode(u32 modulo)`

```c
  // keycode は 後述の init_keycode で得られる u32[3]
  encrypt_64bit(&keycode[1]);
  encrypt_64bit(&keycode[0]);
  for (u32 i = 0; i <= 17; i++) {
    KeyBuf[i] = KeyBuf[i] ^ bswap_32bit(keycode[i % modulo]);
  }

  u32 scratch[2] = {0, 0}; 
  for (u32 i = 0; i <= 1040; i+=2) {
    encrypt_64bit(scratch);
    KeyBuf[i]   = scratch[1];
    KeyBuf[i+1] = scratch[0];
  }
```

## `init_keycode(idcode, level, modulo, key)`

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

復号されたセキュアエリアの最初の8バイトのIDフィールドはASCII文字列で`"encryObj"`であるべきで、もしそれが一致すれば、最初の8バイトは`0xE7FFFDEFF`で埋められ、そうでなければ2KB全体がその値で埋められます。

