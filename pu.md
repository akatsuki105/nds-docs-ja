# 保護ユニット(PU)

デフォルトでは次のように設定されています。

Region Name | Address | Size | Cache | WBuf | Code | Data 
-- | -- | -- | -- | -- | -- | -- 
Background     | 0x0000_0000 | 4GB   | -  | -  | -   | -
I/O and VRAM   | 0x0400_0000 | 64MB  | -  | -  | R/W | R/W
メインメモリ     | 0x0200_0000 | 4MB   | On | On | R/W | R/W
ARM7-dedicated | 0x027C_0000 | 256KB | -  | -  | -   | -
GBA Slot       | 0x0800_0000 | 128MB | -  | -  | -   | R/W
DTCM           | 0x027C_0000 | 16KB  | -  | -  | -   | R/W
ITCM           | 0x0100_0000 | 32KB  | -  | -  | R/W | R/W
BIOS           | 0xFFFF_0000 | 32KB  | On | -  | R   | R
Shared Work    | 0x027F_F000 | 4KB   | -  | -  | -   | R/W

Notes: In Nintendo's hardware-debugger, Main Memory is expanded to 8MB (for that reason, some addresses are at 27NN000h instead 23NN000h) (some of the extra memory is reserved for the debugger, some can be used for game development). 

Region 2 and 7 are not understood? GBA Slot should be max 32MB+64KB, rounded up to 64MB, no idea why it is 128MB? DTCM and ITCM do not use Cache and Write-Buffer because TCM is fast. 

Above settings do not allow to access Shared Memory at 37F8000h? Do not use cache/wbuf for I/O, doing so might suppress writes, and/or might read outdated values.

The main purpose of the Protection Unit is debugging, a major problem with GBA programs have been faulty accesses to memory address 00000000h and up (due to \[base+offset\] addressing with uninitialized (zero) base values). 

This problem has been fixed in the NDS, for the ARM9 processor at least, still there are various leaks: For example, the 64MB I/O and VRAM area contains only ca. 660KB valid addresses, and the ARM7 probably doesn't have a Protection Unit at all. Alltogether, the protection is better than in GBA, but it's still pretty crude compared with software debugging tools.

Region address/size are unified (same for code and data), however, cachabilty and access rights are non-unified (and may be separately defined for code and data).

