# キャッシュ

- 4KBのデータキャッシュ と　8KBの命令キャッシュ
- 4wayセットアソシアティブ
- キャッシュラインは1本32バイト
- Read-allocate method (ie. writes are not allocating cache lines)
- Round-robin and Pseudo-random replacement algorithms selectable
- Cache Lockdown, Instruction Prefetch, Data Preload
- Data write-through and write-back modes selectable
