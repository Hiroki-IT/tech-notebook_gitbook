# 組み込み機器

## 01. 組み込み機器

### 組み込みシステムとは

組み込み機器（限定的な用途向けに特定の機能を果たす事を目的とした機器）を制御するシステムのこと．

※パソコンは、汎用機器（汎用的な用途向けに多様な機能を果たす事を目的とした機器）に分類される．

![組み込みシステム](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/組み込みシステム.png)

<br>

### マイクロコンピュータとは

CPUとして、『マイクロプロセッサ』を用いたコンピュータのこと．

![マイコン](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/マイコン.png)

<br>

### センサによるアナログ情報の入力

外部のアナログ情報を計測し、マイコンに転送する．

**＊具体例＊：温度センサ、加速度センサ、照度センサ、…**

![p119-2](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/p119-2.png)

<br>

### A/D変換器によるアナログ情報からデジタル情報への変換

![AD変換](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/AD変換.png)

<br>

### D/A変換器によるデジタル情報からアナログ情報への変換

![DA変換](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/DA変換.png)

<br>

### Actuater

入力されたエネルギーもしくはコンピュータが出力した電気信号を物理的運動に変換する装置のこと．

![p119-3](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/p119-3.png)

<br>

### 組み込みシステムの制御方式の種類

#### ・シーケンス制御

  決められた順序や条件に従って、制御の各段階を進めていく制御方式．

  **＊具体例＊**

  洗濯機

![p120-1](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/p120-1.png)

#### ・フィードバック制御

  その時の状況を定期的に計測し、目標値との差分を基に、出力を調節する制御方式．

  **＊具体例＊**

  エアコン

![p120-2](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/p120-2.png)

<br>

## 02. 入出力機器


### キーボードからポインティングデバイス

#### ・ジョイスティック

  <br>


### 読み取り機器

#### ・イメージスキャナ

#### ・Optical Character Reader

  紙上の文字を文字データとして読み取る機器．

#### ・Optical Mark Reader

  マークシートの塗り潰し位置を読み取る機器．

![ORM](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ORM.png)

#### ・キャプチャカード

#### ・デジタルカメラ

<br>


### ディスプレイ 

#### ・CRTディスプレイ

#### ・液晶ディスプレイ

  電圧の有無によって液晶分子を制御．外部からの光によって画面を表示させる．

![液晶分子](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/液晶分子.png)

![液晶ディスプレイ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/液晶ディスプレイ.jpg)

#### ・有機ELディスプレイ

  有機化合物に電圧を加えることによって発光させ、画面を表示させる．

![有機ELディスプレイ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/有機ELディスプレイ.jpg)

#### ・プラズマディスプレイ

![プラズマディスプレイ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/プラズマディスプレイ.gif)

  2枚のガラスの間に、封入された希ガスに電圧をかけると放電し、紫外線が出る．そして、この紫外線が蛍光体を発光させることによって画面を表示する．  液晶ディスプレイとのシェア差が大きくなり、2014年に世界的に生産が終了された．

![パナソニック製プラズマディスプレイ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/パナソニック製プラズマディスプレイ.jpg)

#### ・LEDディスプレイ

![LEDディスプレイ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/LEDディスプレイ.jpg)

2018年1月に開催された「CES 2018」でサムスンが発表した“マイクロLEDテレビ”「The Wall」は、従来の「液晶」や「有機EL」とは異なる新たな表示方式を採用したテレビとして、大きな話題となった．

<br>


### プリンタ

#### ・ドットインパクトプリンタ

![ドットインパクトプリンタ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ドットインパクトプリンタ.jpg)

![ドットインパクトプリンタの仕組み](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ドットインパクトプリンタの仕組み.jpg)

#### ・インクジェットプリンタ

![インクジェットプリンタ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/インクジェットプリンタ.jpg)

![インクジェットプリンタの仕組み](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/インクジェットプリンタの仕組み.jpg)

#### ・レーザプリンタ

![レーザプリンタ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/レーザプリンタ.jpg)

![レーザプリンタの仕組み](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/レーザプリンタの仕組み.jpg)

#### ・プリンタの解像度

  １インチ当たりのドット数（dpi）によって、解像度が異なる．復習ではあるが、PC上では、ドット数がどのくらいのビット数を持つかで、解像度が変わる．

![DPI](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/DPI.jpg)

dpiが大きくなるにつれて、解像度は大きくなる．

![DPIの比較](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/DPIの比較.jpg)

#### ・プリンタの印字速度

![CPS と PPM](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/CPS と PPM.jpg)

<br>


## 02-02. 入出力インターフェイス

### Serial interface vs. Parallel interface

シリアルインターフェイスは、情報を1bitずつ転送する方式．パラレルインターフェイスは、複数のbitの情報を同時に転送する方式．パラレルインターフェイスは、同時にデータを送信し、同時に受信しなければならない．配線の形状や長さによって、信号の転送時間は異なる．動作クロックが速ければ速いほど、配線間の同時転送に誤差が生じてしまうため、現代の高スペックパソコンには向いていない．

![パラレルインターフェイスは配線の長さが関係してくる](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/パラレルインターフェイスは配線の長さが関係してくる.png)

![シリアルvs パラレル の違い](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/シリアルvs パラレル の違い.jpeg)

<br>

### Serial interface が用いられている例

#### ・USB（Universal Serial Bus）

![usbケーブル](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/usbインターフェイス.png)

#### ・IEEE1394

ビデオカメラとの接続に用いられるインターフェイス

![ieeeケーブル](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ieeeインターフェイス.jpg)

<br>

### Parallel interface が用いられている例

#### ・IDE（Integrated Drive Electronics）

ハードディスクとの接続に用いられるインターフェイス．

![ideケーブル](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ideインターフェイス.jpg)

#### ・SCSI（Small Computer System Interface）

ハードディスク、CD-ROM、イメージスキャナなど、様々な周辺機器をデイジーチェーンするために用いるインターフェイス．

![scsiケーブル](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/scsiインターフェイス.jpg)

<br>

### 無線インターフェイス

#### ・IrDA（infrared Data Assoiciation）
  赤外線を使って、無線通信を行うためのインターフェイス．

![irDAインターフェイス](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/irDAインターフェイス.jpg)

#### ・Bluetooth
  2.4GHzの電波を使って無線通信を行うためのインターフェイス．



