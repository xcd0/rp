# 試作1

raspberry pi pico相当の基板を作成する。

## ざっくり仕様

* USB-C
* できるだけ幅を狭くする
	* 長くてもよい
	* 不要なGPIOは基板をカットして捨ててもよい
* できるだけ片面実装

## 試作した基板の仕様

* 15mm x 59mm
	* raspberry pi pico は 21mm x 51mm
	* 幅
		* だいぶ狭くできたと思う
		* これ以上狭くするには、
			* USB-Cのコネクタをより狭くできるものを選ぶ
			* 片面実装をやめる
			* 4層基板にする
			* 部品をより小さくする
				* 現状1608(mm)サイズを使用している
				* 0603(mm)サイズにすればさらに狭くできると思われる
			* 現状不要かもしれないコンデンサや抵抗を配置しているので、それを削るのもよいかもしれない。
				* ただそれはどれが不要か判断できない問題がある。
	* 長さ
		* 不要なGPIOの分基板をカットできるかに関しては1cmまではカットできる状態。
		* 部品を小さくする、両面実装にするなどで部品をUSB-Cコネクタ側に寄せれば、もっとカットできる状態にできると思われる
		* RP2040-Zero、RP2040-Tiny をマネして実装すれば部品数削減が可能に思われる
			* RP2040-Zero  
				![](https://www.waveshare.com/img/devkit/RP2040-Zero/RP2040-Zero-details-intro.jpg)  
				* https://www.waveshare.com/rp2040-zero.htm
				* https://www.waveshare.com/wiki/RP2040-Zero
				* https://www.waveshare.com/w/upload/4/4c/RP2040_Zero.pdf
			* RP2040-Tiny  
				![](https://www.waveshare.com/img/devkit/RP2040-Tiny-Kit/RP2040-Tiny-Kit-details-13.jpg)  
				![](https://www.waveshare.com/media/catalog/product/cache/1/image/800x800/9df78eab33525d08d6e5fb8d27136e95/r/p/rp2040-tiny-3.jpg)  

				* https://www.waveshare.com/rp2040-tiny.htm
				* https://www.waveshare.com/w/upload/7/7a/RP2040-Tiny_Schematic.pdf

## 回路図

[ ![回路図]( ./rp2040_slim_v1/slim_20240116_no_vcut.png) ]( ./rp2040_slim_v1/slim_20240116_no_vcut.pdf )

## PCB

![](./rp2040_slim_v1/pcb_ss.png)

## 部品一覧

部品 | 値        | 部品  | 値
-----|-----------|-------|------------
C1	 | 	27 pF	 | 	R1	 | 	5.1 kΩ
C2	 | 	27 pF	 | 	R2	 | 	1 kΩ
C3	 | 	10 uF	 | 	R3	 | 	1 kΩ
C4	 | 	100 nF	 | 	R4	 | 	5.1 kΩ
C5	 | 	10 uF	 | 	R5	 | 	5.1 kΩ
C6	 | 	10 uF	 | 	R6	 | 	5.1 kΩ
C7	 | 	100 nF	 | 	R7	 | 	200 Ω
C8	 | 	100 nF	 | 	R8	 | 	1 Ω
C9	 | 	1 uF	 | 	R9	 | 	27 Ω
C10	 | 	1 uF	 | 	R10	 | 	27 Ω
C11	 | 	100 nF	 | 		 | 	
C12	 | 	1 uF	 | 	U1	 | 	W25Q32JVS   
C13	 | 	100 nF	 | 	U2	 | 	CH213K      
C14	 | 	100 nF	 | 	U3	 | 	AMS1117-3.3 
C15	 | 	100 nF	 | 	U4	 | 	RP2040      
C16	 | 	100 nF	 | 		 | 	
C17	 | 	100 nF	 | 	Y1	 | 	Crystal 12MHz
C18	 | 	100 nF	 | 	J1	 | 	USB-C Connector




