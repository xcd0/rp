# RP2040を直接実装した自作キーボードを作成する

いきなりは難しいので最初にRaspberry Pi Pico相当のものを作成して、動作確認し、  
そのうえで作成する。  
完成は春節前を目標とする。  

## 調査1 資料

公式ドキュメント  
https://datasheets.raspberrypi.com/rp2040/hardware-design-with-rp2040-JP.pdf  
日本語助かる。  

Ki-CADのファイル  
https://datasheets.raspberrypi.com/rp2040/Minimal-KiCAD.zip  
至れり尽くせり助かる。

## 調査2 読解

とりあえず読む。
わからない単語を調べたらメモる。

### 1章

#### P4-5

> ![](./img/p4.png)
> 
> 専用の SPI、DSPI、QSPI インターフェイスを使用して、外部メモリーからコードを直接実行することができます。  
> サイズの小さなキャッシュにより、標準的なアプリケーションのパフォーマンスが向上します。  
> SWD インターフェイスを使用してデバッグを行うことができます。  
> 内蔵 SRAM は、コードやデータを格納するバンク内に配置されます。  
> 専用の AHB バスファブリック接続経由で内蔵 SRAM にアクセスされるため、  
> バスマスターはブロックされることなく個別のバススレーブにアクセスすることができます。  
> DMA バスマスターにより、反復的なデータ転送タスクをプロセッサーからオフロードすることができます。  
> GPIO ピンは、直接起動することも、各種の専用ロジック関数から起動することもできます。  
> 周辺機器専用 IP により、SPI、I2C、UART などの固定機能が提供されます。  
> 柔軟な構成が可能な PIO コントローラーを使用して、さまざまな IO 機能を実行することができます。  
> PHY が内蔵されたシンプルな USB コントローラーにより、ソフトウェアの制御下で FS/LS ホストまたは FS/LS デバイスに接続することができます。  
> 4 つの GPIO で、ADC 入力付きのパッケージピンが共有されます。  
> 2 つの PLL により、USB または ADC の 48MHz 固定クロックと、最大 133MHz の柔軟なシステムクロックが稼働します。  
> 内蔵の電圧レギュレーターによってコア電圧が供給されるため、完成品で供給する必要があるのは IO 電圧だけになります。  

chatgptによる用語の説明。

| 用語 | 説明 |
|------|------|
| SPI (Serial Peripheral Interface) | シリアル通信プロトコルで、マスターとスレーブ間でデータ交換を可能にします。RP2040では、外部デバイスとの高速通信に用いられます。 |
| DSPI (Dual SPI) | SPIの変種で、2つのデータラインを使用して同時にデータを送受信することができ、転送速度が向上します。 |
| QSPI (Quad SPI) | SPIの拡張版で、4つのデータラインを使ってデータ転送を行います。これにより、さらに高速なデータ転送が可能になります。 |
| SWD (Serial Wire Debug) インターフェイス | JTAGの代替として設計されたデバッグインターフェイスで、少ないピン数でフル機能のデバッグが可能です。RP2040では、プログラムのデバッグに使用されます。 |
| 内蔵 SRAM (Static Random Access Memory) | 高速アクセス可能なメモリで、RP2040ではコードやデータの格納に使用されます。低遅延であり、プロセッサの性能向上に貢献します。 |
| AHB (Advanced High-performance Bus) バスファブリック接続 | 高性能なシステムバスの一種で、RP2040では内蔵SRAMへの効率的なアクセスを提供します。バスマスターが個別のバススレーブにアクセスする際にブロックされることがありません。 |
| DMA (Direct Memory Access) バスマスター | CPUの代わりにデータ転送を行うハードウェア。RP2040では、反復的なデータ転送をプロセッサーからオフロードし、効率を高めます。 |
| オフロードする | あるタスクや処理を、主処理装置（例えばCPU）から別の装置（例えばDMA）に移行させること。これにより、主処理装置の負荷を軽減し、全体の効率を向上させます。 |
| 周辺機器専用 IP (Intellectual Property) | 専用のハードウェアモジュールやデザイン。RP2040では、SPI、I2C、UARTなどの固定機能を提供するために用いられます。 |
| PIO (Programmable I/O) コントローラー | ユーザーがプログラム可能なI/Oコントローラー。RP2040では、さまざまなI/O機能をカスタマイズして実行するために使用されます。 |
| PHY (Physical Layer) | 通信ネットワークの物理層を担う部分。RP2040では、USB通信の物理層を内蔵しており、FS/LSホストやデバイスとして機能します。 |
| FS/LS ホスト (Full-Speed/Low-Speed Host) | USBの通信速度規格の一つ。RP2040はFS（フルスピード）とLS（ロースピード）のホストとして機能できます。 |
| FS/LS デバイス (Full-Speed/Low-Speed Device) | USBデバイスとしての動作モード。RP2040では、USBやADC用の固定クロックおよび柔軟なシステムクロックを生成するために使用されます。|

理解した内容。(ざる)  
* SPI/DSPI/QSPI経由で外部のメモリから直接コードを実行できる。
* SWD(Serial Wire Debug)経由で少ないピン数でデバッグできる。

## 第2章 最小設計例
### P6

理解した内容。  
* PCBの厚さは1mm。1.6mmだとUSBの特性インピーダンスに関する問題が発生する可能性がある。(後述)

### P7-8

![](./img/p7_1.png)

理解した内容。  
最小設計は4つの要素からなる。
* 電源
	* 1.1V(コア用) 
		* RP2040の内臓レギュレータによって3.3Vから変換されるので無視してよい。
	* 3.3V(IO用)
		* 外部から与える必要がある。通常USB(5V)から得る。  
		![](./img/p7_2.png)
		* USB_B_Micro
			* VBUSにPCからの5Vが来ている。
			* IDはUSB On the Goのための信号線。GNDに落とすとホスト扱いになる。繋がないとデバイス扱いされる。
			* この回路ではデバイス扱いされるはず。
		* NCP1117-3.3_SOT223
			* 3端子レギュレータ。PCから供給された5VのVBUSから、RP2040のIO用電圧3.3Vを生成する。1Aまで供給できるらしい。
			* NCP1117のデータシートに入力側、出力側の両方に10uFのコンデンサが必要らしい。
		* C1,C4
			* データシート日本語助かる。
			* NCP1117のデータシート https://www.onsemi.jp/download/data-sheet/pdf/ncp1117-d.pdf
			> 外付けけコンデンサ  
			> デバイスが電源から数インチいじょうはなれたば所に設置された場合、  
			> レギュレータを安定させるために入力バイパスコンデンサC_inが必要になる可能性があります。  
			> このコンデンサは、複雑な入力インピーダンスを通じて電力が供給される場合に回路が過敏に反応するのを抑制し、出力過渡応答を大幅に改善させます。  
			> 入力バイパスコンデンサはできるだけハイ船長が短くなるように、レギュレータの入力端子とグランド端子の間に直接取り付ける必要があります。  
			> 10uFのセラミックコンデンサまたはタンタルコンデンサであればほとんどのアプリケーションに対して十分な効果があります。  
			> コンデンサC_outによってレギュレータの周波数保証が行われるため、出力を安定させるにはこのコンデンサの使用は必須です。  
			> 33mΩ ~ 2.2mΩ という制限内の等価直列抵抗(ESR)を持つ、4.7uFの最小容量値が必要です。。
			> ![](./img/p8_NCP1117.png)

* フラッシュストレージ
* 水晶発振器
* IO






* フラッシュストレージ
* 水晶発振器
* IO






