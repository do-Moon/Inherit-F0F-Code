# F0F継承化

## F0F継承とは？
**F0F継承**とは、F0Fコード内では分岐不可なアドレス(継承先)に分岐するということです。  
そのおかげでコードの拡張も簡単にでき、関数の定義からコールバック関数までも実現できます。  
ARMで関数と言えば曖昧ですが、分岐がしやすくなったという事実は変わりません。  
C/Goのようにmain()関数をF0Fとし、F0Fで関数を呼び出し(継承先で処理された値など)まとめて処理するといったことが可能です。  
また、一度書いたコードを使いまわししたい場合にも継承は便利です^^。  
では、サンプルコードのF0F継承コードを解説します。  
サンプルコードを見て作り方を覚えましょう。  

## F0F継承コードの解説
まずは、サンプルコードから見て頂きます。
処理内容は以下の通り。
```arm
E59F0024  ldr r0, [pc, #0x24]
EA000001  b  #0xc
E580100C  str r1, [r0, #0xc]
E12FFF1E  bx lr
E1A0C00F  mov r12, pc
E5901000  ldr r1, [r0]
E3510000  cmp r1, #0
0A000001  beq #0xc
E12FFF10  bx path_data
EAFFFFF7  b #0xffffffe4
E12FFF1E  bx lr
01E81000  .word 0x01E81000

.path_data
  E3A01001  mov r1, #1
  E28CC00C  add 12, 12, #0xc
  E1A0F00C  mov pc, 12

.build_data
  .word 0x00000001
```

### 解説
全てを詳しく解説すると長くなるので簡単に書いていきます。  

ldr r0, [pc, #0x24]  
[pc, #0x24]の演算結果をr0に格納(.word 0x01E81000)  

b #0xc  
mov r12, pcからスタート  

str r1, [r0, #0xc]  
.path_dataで処理されたr1の値を[r0, #0xc]先に格納(.word 0x01E8100C)  

bx lr  
lrに格納されているアドレスへと分岐する(プログラム終了)  

mov r12, pc  
pcレジスタの値をr12に格納(これにより分岐先からF0Fへ帰還できる)  

ldr r1, [r0]  
[[r0]](01E81000)の値をr1に格納(.word 0x00000000)  

cmp r1, #0  
r1と#0を比較する(これにより、もし01E81000の値が0と↓)  

beq #0xc  
(等しければ、bx lrに分岐し安全にプログラムを終了する)  

bx path_data  
.path_data(01E81000)へ分岐する  

b #0xffffffe4  
.path_dataで処理された値を01E8100Cに書き込むためにstr r1, [r0, #0xc]へ分岐  

bx lr  
プログラム終了  

.word 0x01E81000  
ldr r0, [pc, #0x24]のpc+0x24した値  

#### .path_data(01E81000)からの処理
mov r1, #1  
#1をr1に格納  

add 12, 12, #0xc  
mov r12, pcで格納されたr12をF0F内に記述したb #0xffffffe4までの距離(#0xc)をr12に格納  
**<span style="color: red">※ここが重要</span>**  
**<span style="color: red">F0Fに戻るために利用するr12の値はmov r12, pcのアドレスからb #0xffffffe4までの距離なので、  
mov r12, pc ~ b #0xffffffe4 までを 32bitを頭に入れておき、0x14に-0x8をする  
この計算結果は0xcになり、実質cmp r1, #0から数えた結果になる</span>**  

mov pc, r12  
r12をpcに格納し、b #0xffffffe4へ帰還(分岐)する  

### これをコード化し、一度ビルド(実行)する
```gw_code
F0F00000 00000030
E59F0024 EA000001
E580100C E12FFF1E
E1A0C00F E5901000
E3510000 0A000001
E12FFF10 EAFFFFF7
E12FFF1E 01E81000
E1E81000 0000000C
E3A01001 E28CC00C
E1A0F00C 00000000
```

#### build結果
.build_data  
.word 0x00000001(01E8100Cに1が書き込まれていれば成功)    

難しく感じて慣れれば余裕なので考えて楽しんでF0F継承コードを組み立てていってください！
