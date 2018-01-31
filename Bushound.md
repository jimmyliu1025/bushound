#Bus Hound


USB運作模式
-------

&nbsp;&nbsp;&nbsp;&nbsp;位於PC上的USB裝置稱為 USB Host，上面可外接數個裝置(USB Device)，而Host會主導整個USB結構的通訊。
方法為<font color="red">輪流詢問<font color="black">所有Device以檢查是否有裝置需要傳送資料，所有Device都須等待Host的命令，Host同意後Device才可開始傳送資料。

<a href="https://imgbb.com/"><img src="https://image.ibb.co/htvmJm/1.png" alt="1" border="0"></a>

USB Device結構
-------

&nbsp;&nbsp;&nbsp;&nbsp;Device的結構大約分為**模式(Configurations)**、**介面(Interfaces)**、**設定值(Settings)**和**端點(Endpoints)**。

<a href="https://ibb.co/eoMAdm"><img src="https://preview.ibb.co/fMxiym/2.png" alt="2" border="0"></a>

<a href="https://imgbb.com/"><img src="https://image.ibb.co/ka3bjR/3.png" alt="3" border="0"></a>



•	模式(Configurations)：
  可使USB扮演不同的角色，但<font color="red">一次只能使用一種模式<font color="black">，不可能同時有兩種模式並行的運作狀態。(ex常見的應用是將 USB 裝置切換至 Firmware 的更新模式，做韌體升級，此時該硬體的一般模式將沒有作用。)
<br><br>
•	介面(Interfaces)：
  一個 USB 裝置可能有很<font color="red">多個輸入或輸出介面<font color="black">，舉例來說，若是有一個帶麥克風的耳機，就會有兩個介面，分別為『聲音 Input』和『聲音 Output』所使用。也因此作業系統須<font color="red">使用不同的裝置驅動程式來管理<font color="black">這兩個介面。
<br><br>
•	端點(Endpoint) ：
  <font color="red">可以被視為資料的來源或目標<font color="black">，USB Device初始化完後，剩下的就是使用 Endpoint 與 USB Device 溝通，以<font color="red">實現傳送接收的功能<font color="black">，而多個終端點集合成一介面。<br>
&nbsp;&nbsp;&nbsp;&nbsp;每一個 USB 裝置保留其終端點位址0 (Endpoint 0)作為規劃用，透過終端點位址 0，USB系統軟體從裝置取得USB裝置描述元，這些描述元提供必備的資訊用來識別裝置，指出終端點的數目及其功能。藉由此方式，系統軟體偵測到裝置的型態或種類，並且決定該如何存取此裝置。
<font color="red">除Endpoint 0外，其餘皆只能單向傳輸。<font color="black">


端點(Endpoint) 傳輸型態
-----

###傳輸型態分為四種：

•	控制型(Control)
  通常用來控制USB Device所使用，<font color="red">只能用在endpoint 0上。<font color="black">用於**配置設備**、**獲取設備資訊**、**發送命令**或者**獲取設備的狀態報告**。
<br><br>
•	巨量型(Bulk)
  大量且安全的傳輸，通常應用於不可失真且大資料量的通訊，如：隨身碟。USB協定中，並不保證Bulk會有一定的頻寬，所以若是當下 USB bus <font color="red">頻寬不夠，資料會被分批進行傳輸<font color="black">(為非週期性或無傳輸速率限制之裝置，若有錯誤就重傳)。
<br><br>
•	中斷型(Interrupt)
  Host的輪詢機制會對所有USB Device請求Interrupt傳輸，若此時USB Device有需要Interrupt傳輸，便可開始以<font color="red">固定的速率<font color="black">傳送少量資料，通常應用在鍵盤、滑鼠此種HID上。(USB 不支援硬體中斷，那些必須靠中斷驅動的 USB 裝置必須被週期性的詢問，以便得知是否有資料需要傳送) 
<br><br>
•	同步型(Isochronous)
  大量且穩定持續但不保證安全的傳輸，也就是<font color="red">不會檢查資料<font color="black">是否完整通過，但<font color="red">保證恆定的資料流程<font color="black">(ex畫面和聲音同步，少傳畫面沒關係)，多用於資料獲取。可應用串流影音等可容許資料失真的地方。

<a href="https://imgbb.com/"><img src="https://image.ibb.co/eBTFdm/4.png" alt="4" border="0"></a>


USB 規範指出最多可以有<font color="red">90%的 USB 頻寬（Bus Bandwidth）用於週期性（**中斷與即時**）的傳輸<font color="black">，而<font color="red">控制型傳輸在每個訊框(Frame)中保留 10%，剩餘的頻寬則分配給巨量型傳輸<font color="black">，因為**巨量型傳輸**對於頻寬沒有特別需求，在每個訊框(Frame)中它們被歸入最低優先權。


USB 資料流的傳送與訊框模組
----
&nbsp;&nbsp;&nbsp;&nbsp;在 USB 軟體系統的角度，<font color="red">USB 硬體終端點（Endpoint）便視成管道（Pipe）<font color="black">使用，當用戶端應用程式要求 USB 管道傳送或接收資料時，程式首先透過<font color="blue">系統介面函式（API）<font color="black">告知裝置驅動程式所需要的動作。經由<font color="blue">USB 匯流排驅動程式 (USB Bus Driver)<font color="red">會分解需求資料量成數個處理動作(Transactions)<font color="black">，並且將<font color="red">依裝置需求的特性，將它們填入適當的訊框（Frame）中<font color="black">。<font color="blue">USB 主機控制器驅動程式（USB Host Controller Driver）<font color="black">將處理動作排序成一系列的處理次序表（Transactions List），一張處理次序表定義一串將會在 1ms 訊框中被執行的處理動作。

>>><a href="https://imgbb.com/"><img src="https://image.ibb.co/dVfNPR/5.png" alt="5" border="0"></a>
>>><a href="https://imgbb.com/"><img src="https://image.ibb.co/eSrwjR/6.png" alt="6" border="0"></a>


&nbsp;&nbsp;&nbsp;&nbsp;裝置A與B都有一個即 時（Isochronous）終端點，在每個訊框(Frame)中系統都保留固定與相當大的頻寬。而裝置C則有一中斷的終端點，將以固定的頻率每兩個訊框詢問(Polling)一次，系統將為它每兩個訊框保留一小部份的空 間。當訊框不含C裝置詢問(Poll)所造成多餘的頻寬將分配給巨量（Bulk）傳輸與其他目的使用。

USB資料連結
----
<a href="https://ibb.co/drHDW6"><img src="https://preview.ibb.co/kKp5dm/12.gif" alt="12" border="0"></a>
<a href="https://imgbb.com/"><img src="https://image.ibb.co/mVORjR/7.png" alt="7" border="0"></a>

Transaction指USB資料的傳輸，大部分的傳輸包含了三種封包，<font color="red">封包是組成USB傳輸的最小單位<font color="black">。 一個 Transaction <font color="red">通常由三個封包組成<font color="black">，但依傳輸型態而定，一個 Transaction 可能包含一個、兩個、三個封包。

<a href="https://ibb.co/eu4JW6"><img src="https://preview.ibb.co/jJ9kB6/8.png" alt="8" border="0"></a>



<a href="https://ibb.co/kVzSr6"><img src="https://preview.ibb.co/h9NQdm/9.png" alt="9" border="0"></a>

<a href="https://ibb.co/g15BJm"><img src="https://preview.ibb.co/muJnr6/10.png" alt="10" border="0"></a>

<a href="https://ibb.co/gzw1Jm"><img src="https://preview.ibb.co/m7ZXPR/11.png" alt="11" border="0"></a>




Bus Hound 解讀
----
###基本參數<br>
<a href="https://imgbb.com/"><img src="https://image.ibb.co/mdbwjR/12.png" alt="12" border="0"></a><br>

Class(群組代碼):&nbsp;&nbsp;&nbsp;&nbsp;			代表interface Class 0x08代表mess storage
<br>
SubClass(次群組代碼): &nbsp;&nbsp;&nbsp;&nbsp;		主群組代碼之細分，0x06代表SCSI
<br>
Protocol(介面協定):&nbsp;&nbsp;&nbsp;&nbsp; 		0x50 代表Bulk-Only Transfer
<br>
CMD(命令數):&nbsp;&nbsp;&nbsp;&nbsp;				從1開始遞增
<br>
Phase(命令中的階段數):&nbsp;&nbsp;&nbsp;&nbsp; 	一個命令通常由多個階段組成
<br>
Ofs(階段中的偏移量): &nbsp;&nbsp;&nbsp;&nbsp;		從0開始
<br>
Rep(命令重複發布數):&nbsp;&nbsp;&nbsp;&nbsp;		重複數

<a href="https://ibb.co/cyUhPR"><img src="https://preview.ibb.co/dGdLB6/13.png" alt="13" border="0"></a>



###CMD 1:Host 對 Address 0 發送 GetDescriptor(Device Descriptor) 的請求<br>

<a href="https://ibb.co/fuMXr6"><img src="https://preview.ibb.co/hhO1jR/1.png" alt="1" border="0"></a>

<a href="https://imgbb.com/"><img src="https://image.ibb.co/ehXoW6/15.png" alt="15" border="0"></a>


bmRequest Type: &nbsp;&nbsp;&nbsp;&nbsp;(Byte 0)
<br>
bit 7: &nbsp;&nbsp;&nbsp;&nbsp;Request Direction 			1 – Device to Host (IN)
<br>
bit 6-5: &nbsp;&nbsp;&nbsp;&nbsp;Type of Request			0 - Standard
<br>
bit 4-0: &nbsp;&nbsp;&nbsp;&nbsp;Recipient					0 - Device
<br>
bRequest: &nbsp;&nbsp;&nbsp;&nbsp;(Byte 1) 				0x06 Get Descriptor
<br>
wValue: &nbsp;&nbsp;&nbsp;&nbsp;(Byte 2-3)				Descriptor Index  01表示 Device Descriptor
<br>
wIndex: &nbsp;&nbsp;&nbsp;&nbsp;(Byte 4-5)	代表語言ID 
<br>
wLength: &nbsp;&nbsp;&nbsp;&nbsp;(Byte 6-7)
代表Device要回傳多少Data回來(可小於)


###CMD 1 Phase 2:<br>
<a href="https://ibb.co/gwOsPR"><img src="https://preview.ibb.co/mxJRjR/3.png" alt="3" border="0"></a>

Byte 0						&nbsp;&nbsp;&nbsp;&nbsp;Descriptor長度，0x12個Byte
<br>
Byte 1						&nbsp;&nbsp;&nbsp;&nbsp;Descriptor種類，01代表Device Descriptor
<br>
Byte 2-3						&nbsp;&nbsp;&nbsp;&nbsp;USB版本，0200，USB2.0
<br>
Byte 4						&nbsp;&nbsp;&nbsp;&nbsp;裝置類別，0表Device的Class是在介面描述元中定義
<br>
Byte 5						&nbsp;&nbsp;&nbsp;&nbsp;表Sub Class類別，0同上
<br>
Byte 6						&nbsp;&nbsp;&nbsp;&nbsp;表 Protocal，0同上 (bulk-only 上三項必0)
<br>
Byte 7						&nbsp;&nbsp;&nbsp;&nbsp;表 bMaxPacketSize，0x40為EP 0，packet大小
<br>
Byte 8-9						&nbsp;&nbsp;&nbsp;&nbsp;表 vid，13fe為Kingston Technology Company Inc.
<br>
Byte 10-11						&nbsp;&nbsp;&nbsp;&nbsp;表 PID，4200，product ID
<br>
Byte 12-13						&nbsp;&nbsp;&nbsp;&nbsp;表 Deive序號，0001
<br>
Byte 14						&nbsp;&nbsp;&nbsp;&nbsp;表 Manufacturer號，01
<br>
Byte 15						&nbsp;&nbsp;&nbsp;&nbsp;表 Product號，02
<br>
Byte 16						&nbsp;&nbsp;&nbsp;&nbsp;表 SN號，03
<br>
Byte 17						&nbsp;&nbsp;&nbsp;&nbsp;Number of possible configration，01
<br>

###CMD 2 Phase 1: Host 發送 GetDescriptor(Config) 的請求<br>
<a href="https://ibb.co/itOnr6"><img src="https://preview.ibb.co/iGXfB6/4.png" alt="4" border="0"></a><br>
Byte 3						&nbsp;&nbsp;&nbsp;&nbsp;Descriptor Index  02表示 Configuration Descriptor
<br>

###CMD 2 Phase 2:<br>
<a href="https://ibb.co/itOnr6"><img src="https://preview.ibb.co/iGXfB6/4.png" alt="4" border="0"></a><br>
Byte 0						&nbsp;&nbsp;&nbsp;&nbsp;Descriptor長度，0x09個Byte<br>
Byte 1						&nbsp;&nbsp;&nbsp;&nbsp;Descriptor種類，02代表Configuration Descriptor
<br>
Byte 2						&nbsp;&nbsp;&nbsp;&nbsp;wTtLen_L，0x20(該配置返回的數據( itf,ep )總長)<br>
Byte 3						&nbsp;&nbsp;&nbsp;&nbsp;wTotalLength_H，0x00 <br>
Byte 4						&nbsp;&nbsp;&nbsp;&nbsp;表 interface數量，0x01<br>
Byte 5						&nbsp;&nbsp;&nbsp;&nbsp;表此 Configuration 的編號，0x01<br>
Byte 6						&nbsp;&nbsp;&nbsp;&nbsp;表此 config 的String Descriptor 編號，0x00表沒有<br>
Byte 7						&nbsp;&nbsp;&nbsp;&nbsp;表此配置的屬性，(ex. Bit6=1，代表從Usb Bus 供電)<br>
Byte 8						&nbsp;&nbsp;&nbsp;&nbsp;表所用電流，單位為2mA，0x64，200mA<br>

###CMD 3 Phase 1: Host 發送 GetDescriptor(Config) 的請求<br>
<a href="https://ibb.co/haEtW6"><img src="https://preview.ibb.co/mTRU4R/5.png" alt="5" border="0"></a><br>
###CMD 3 Phase 2: <br>
第一組為 Configuration，共 9 Byte，而 0x02 代表 組態描述元(Configuration Descriptor)<br>
第二組為 Interface ，共 9 Byte，而 0x04 代表 介面描述元(Interface Descriptor)<br>
第三組為 Endpoint ，共 7 Byte，而 0x05 代表 端點描述元(Endpoint Descriptor)<br>
第四組為 Endpoint ，共 7 Byte，而 0x05 代表 端點描述元(Endpoint Descriptor)<br><br><br>

第二組：09 04 00 00 02 08 06 50 00<br>
Byte 1						&nbsp;&nbsp;&nbsp;&nbsp;種類，04代表 Interface descriptor <br>
Byte 2						&nbsp;&nbsp;&nbsp;&nbsp;0x00，Interface Number<br>
Byte 3&nbsp;&nbsp;&nbsp;&nbsp;						0x00，Altemate Setting，代表交替設定<br>
Byte 4&nbsp;&nbsp;&nbsp;&nbsp;						0x02，代表此介面所使用的Endpoint數 <br>
Byte 5&nbsp;&nbsp;&nbsp;&nbsp;						表Interface Class(群組代碼)，0x08代表 mess storage<br>
Byte 6&nbsp;&nbsp;&nbsp;&nbsp;						表SubClass code(次群組代碼)， 0x06代表SCSI<br>
Byte 7&nbsp;&nbsp;&nbsp;&nbsp;						 interface Protocol(介面協定)，0x50代表Bulk-Only<br> 
Byte 8&nbsp;&nbsp;&nbsp;&nbsp;						表Interface 的 String Descriptor，0x00 代表沒有<br>

第三組：07 05 81 02 00 02 00 <br>
Byte 1&nbsp;&nbsp;&nbsp;&nbsp;						種類，05代表 Endpoint descriptor <br>
Byte 2	&nbsp;&nbsp;&nbsp;&nbsp;					EndPoint Address，0x81，10000001，(b7:1 in，0001 EP1)<br>
Byte 3&nbsp;&nbsp;&nbsp;&nbsp;						Attribute(端點屬性)，0x02，代表Bulk 類<br>
Byte 4	&nbsp;&nbsp;&nbsp;&nbsp;					0x00，wMaxPacketSize_L: 最大傳輸量<br> 
Byte 5&nbsp;&nbsp;&nbsp;&nbsp;						0x02，wMaxPacketSize_H : 0x200 = 512 為 HighSpeed<br>
Byte 6						host 間隔多少時間來 polling device，0x00表不用<br><br>
第四組：07 05 02 02 00 02 00<br>
Byte 2	&nbsp;&nbsp;&nbsp;&nbsp;					EndPoint Address，0x02，00000010，(b7:0 out, 0010 EP2)<br><br><br>

###CMD 4 CMD 5: Host 下 Get String Descriptor  <br>
<a href="https://ibb.co/b8XVB6"><img src="https://preview.ibb.co/gQuz4R/6.png" alt="6" border="0"></a><br>
Byte 2	&nbsp;&nbsp;&nbsp;&nbsp;					index，這裡的 0x00，代表只取LanguageID<br>
Byte 3	&nbsp;&nbsp;&nbsp;&nbsp;					種類，03代表 String descriptor <br>
Byte 4-5&nbsp;&nbsp;&nbsp;&nbsp;						0x00，代表Language ID，表沒有特別的ID <br>

###CMD 5 Phase 2:<br>
Byte 0&nbsp;&nbsp;&nbsp;&nbsp;						Length，0x04 長度為 4<br>
Byte 1	&nbsp;&nbsp;&nbsp;&nbsp;					種類，03代表 String descriptor<br>
Byte 2-3&nbsp;&nbsp;&nbsp;&nbsp;						表Language ID = 0x0409，為 English (United States)<br>

###CMD 6 CMD 7:<br>
<a href="https://ibb.co/ktn8ym"><img src="https://preview.ibb.co/d4q6jR/7.png" alt="7" border="0"></a><br>
###CMD 7 Phase 1:<br>
Byte 2	&nbsp;&nbsp;&nbsp;&nbsp;					index，這裡的 0x03，代表只Product String<br>
Byte 4-5&nbsp;&nbsp;&nbsp;&nbsp;						0x0409，代表Language ID<br>
###CMD 7 Phase 2:<br>
依照ASCII翻譯，為Host端的Device name<br>

###CMD 8:<br>
<a href="https://ibb.co/eS5W16"><img src="https://preview.ibb.co/dnn0uR/8.png" alt="8" border="0"></a><br>
Byte 0	&nbsp;&nbsp;&nbsp;&nbsp;					0x00，bit7 0 – Host to Device (OUT)<br>
Byte 1	&nbsp;&nbsp;&nbsp;&nbsp;					0x09，設定configuration<br>

###CMD 9:<br>
Byte 0	&nbsp;&nbsp;&nbsp;&nbsp;					0x01，bit7 0 – Host to Device (OUT)，Recipient 1 - Endpoint<br>
Byte 1	&nbsp;&nbsp;&nbsp;&nbsp;					0x0b，設定interface<br>

###CMD 10: 確定設備之邏輯單元<br>
 
Byte 0	&nbsp;&nbsp;&nbsp;&nbsp;					0xa1，bit7 1 – Device to Host (IN)，bit5 1 - Request -> Class<br>
Byte 1	&nbsp;&nbsp;&nbsp;&nbsp;					0xfe，field set to 254<br>
###CMD 10 Phase 2:
Byte 0	&nbsp;&nbsp;&nbsp;&nbsp;					0x00，僅有一個邏輯單元<br>

###CMD 11: Host傳送CBW包 (Command Block Wrap)<br>
<a href="https://ibb.co/i81bZR"><img src="https://preview.ibb.co/jQAEM6/9.png" alt="9" border="0"></a><br>
<a href="https://ibb.co/jgMbZR"><img src="https://preview.ibb.co/b66og6/10.jpg" alt="10" border="0"></a><br>

dCBWSignature:	&nbsp;&nbsp;&nbsp;&nbsp;				CBW的簽名標誌 43 42 53 55h<br>
dCBWTag:&nbsp;&nbsp;&nbsp;&nbsp;						由host生成之標誌，在CSW裡面需要回填這個欄位<br>
dCBWDataTransferLength: 	&nbsp;&nbsp;&nbsp;&nbsp;		表示接下來host想slave回發36個資料0x24<br>
bmCBWFlags:	&nbsp;&nbsp;&nbsp;&nbsp;				0x80 bit7 1 – Device to Host (IN)<br>
bCBWLU:	&nbsp;&nbsp;&nbsp;&nbsp;					其值是0x00，這裡只有一個Lun<br>
bCBWCBLength:	&nbsp;&nbsp;&nbsp;&nbsp;				其值是0x06，也就是說這裡接下來是6個資料<br>
CBWCB:&nbsp;&nbsp;&nbsp;&nbsp;
SCSI命令就是Inquiry : (第一个是0x12就是SCSI協議裡面定義的Inquiry命令) 12 00 00 00 24 00<br>

###CMD 12: Slave端回覆36筆資料<br>
<a href="https://ibb.co/j6ciim"><img src="https://preview.ibb.co/epFEpR/37.png" alt="37" border="0"></a><br /><a target='_blank' href='https://zh-tw.imgbb.com/'>free photo hosting</a><br />
<a href="https://ibb.co/kQHB16"><img src="https://preview.ibb.co/hL64M6/12.png" alt="12" border="0"></a><br>
RBM:		&nbsp;&nbsp;&nbsp;&nbsp;					裝置是否支援removable media : 1 -> is removable<br>
Response data format:	&nbsp;&nbsp;&nbsp;&nbsp;			回應資料支援的規格( complies to this version of SCSI )<br>
Additional length:&nbsp;&nbsp;&nbsp;&nbsp;				在標準查詢之後，尚餘多少可用資料。<br>
RelAdr:		&nbsp;&nbsp;&nbsp;&nbsp;				是否支援相關位址<br>
WBus32:		&nbsp;&nbsp;&nbsp;&nbsp;				是否支援32 bit Wide SCSI<br>
WBus16:		&nbsp;&nbsp;&nbsp;&nbsp;				是否支援16 bit Wide SCSI<br>
Sync:		&nbsp;&nbsp;&nbsp;&nbsp;					是否支援同步資料傳輸<br>
Linked:	&nbsp;&nbsp;&nbsp;&nbsp;					是否支援指令連結<br>
CmdQue:	&nbsp;&nbsp;&nbsp;&nbsp;					是否支援指令佇列<br>
SftRe:	&nbsp;&nbsp;&nbsp;&nbsp;						是否支援軟體重置<br>

###CMD 13: 傳送CSW回Host (Command State Wrap)<br>
<a href="https://ibb.co/jXnTg6"><img src="https://preview.ibb.co/cLgX8m/13.png" alt="13" border="0"></a><br>
<a href="https://imgbb.com/"><img src="https://image.ibb.co/k0GJER/14.png" alt="14" border="0"></a>
<br>

dCSWSignature:	&nbsp;&nbsp;&nbsp;&nbsp;				CSW的簽名標誌 53 42 53 55<br>
dCSWTag:&nbsp;&nbsp;&nbsp;&nbsp;						回填標誌給在CBW<br>
dCSWDataResidue: &nbsp;&nbsp;&nbsp;&nbsp;				還需傳送之數據<br>
bCSWStatus:	&nbsp;&nbsp;&nbsp;&nbsp;				指示命令的執行狀態，如正確即返回0<br>

###CMD 17: <br>
<a href="https://ibb.co/itSauR"><img src="https://preview.ibb.co/d8xx8m/15.png" alt="15" border="0"></a><br>
dCBWSignature:	&nbsp;&nbsp;&nbsp;&nbsp;				CBW的簽名標誌 43 42 53 55h<br>
dCBWTag:	&nbsp;&nbsp;&nbsp;&nbsp;					由host生成之標誌，在CSW裡面需要回填這個欄位<br>
dCBWDataTransferLength: &nbsp;&nbsp;&nbsp;&nbsp;			表示接下來host想slave回發252個資料0xfc<br>
bmCBWFlags:		&nbsp;&nbsp;&nbsp;&nbsp;			0x80 bit7 1 – Device to Host (IN)<br>
bCBWLU:		&nbsp;&nbsp;&nbsp;&nbsp;				其值是0x00，這裡只有一個Lun<br>
bCBWCBLength:	&nbsp;&nbsp;&nbsp;&nbsp;				其值是0x0a，命令長度是10個資料<br>
CBWCB: 		&nbsp;&nbsp;&nbsp;&nbsp;	SCSI命令是: 23 00 00 00 00 00 00 00 fc 00 (0x23 SCSI Read Capacity命令)<br>

###CMD 18: <br>
Stall PID:	&nbsp;&nbsp;&nbsp;&nbsp;					Packet Identifier無法傳送或接收資料<br>
###CMD 19:<br>
RESTART:	&nbsp;&nbsp;&nbsp;&nbsp;					重覆傳輸一次<br>

###CMD 20:上組數據有問題 此組不正確<br>
<a href="https://ibb.co/bLUG16"><img src="https://preview.ibb.co/hjZig6/16.png" alt="16" border="0"></a><br>
dCSWDataResidue: 				還需傳送之數據 0xfc<br>
bCSWStatus:					指示命令的執行狀態，如不正確即返回1<br>

###CMD 21:傳送0x03 CBW<br>
<a href="https://ibb.co/idytg6"><img src="https://preview.ibb.co/j3qTER/17.png" alt="17" border="0"></a><br>

dCBWSignature:	&nbsp;&nbsp;&nbsp;&nbsp;				CBW的簽名標誌 43 42 53 55h<br>
dCBWDataTransferLength: &nbsp;&nbsp;&nbsp;&nbsp;			表示接下來host想slave回發18個資料0x12<br>
bmCBWFlags:	&nbsp;&nbsp;&nbsp;&nbsp;				0x80 bit7 1 – Device to Host (IN)<br>
bCBWLU:		&nbsp;&nbsp;&nbsp;&nbsp;				其值是0x00，這裡只有一個Lun<br>
bCBWCBLength:	&nbsp;&nbsp;&nbsp;&nbsp;				其值是0x0c，命令長度是12個資料<br>
CBWCB:&nbsp;&nbsp;&nbsp;&nbsp;
SCSI命令是: 03 00 00 00 12 00 00 00 00 00 00 00 00  (0x03 SCSI Request Sense命令，請求設備向主機返回執行結果，及狀態資料)<br>
###CMD 22: <br>
<a href="https://ibb.co/hwgJER"><img src="https://preview.ibb.co/ceLpM6/18.png" alt="18" border="0"></a><br>
Error Code:	&nbsp;&nbsp;&nbsp;&nbsp;					70<br>
Sense Key:&nbsp;&nbsp;&nbsp;&nbsp;						06<br>
Additional Sense Length:&nbsp;&nbsp;&nbsp;&nbsp;			0a <br>
Additional Sense Code:		&nbsp;&nbsp;&nbsp;&nbsp;		29<br>
Additional Sense Code Qualifier:	&nbsp;&nbsp;&nbsp;&nbsp;	00<br>

<a href="https://upload.cc/i/kwi9TJ.png"><img src="https://upload.cc/i/kwi9TJ.png" alt="19" border="0"></a>

對應到: &nbsp;&nbsp;&nbsp;&nbsp;power on, reset, or bus device reset ocurred<br>


###CMD 23: 表示cmd21的指令結束<br>
<a href="https://ibb.co/bRKKom"><img src="https://preview.ibb.co/jQi116/21.png" alt="21" border="0"></a><br>
dCSWDataResidue: 				還需傳送之數據<br>
bCSWStatus:					指示命令的執行狀態，如正確即返回0<br>


###CMD24、25、26: <br>
<a href="https://ibb.co/b3oC8m"><img src="https://preview.ibb.co/dEyZM6/22.png" alt="22" border="0"></a><br>
HOST希望執行0x23指令，(0x23 SCSI Read Format Capacity命令) 23 00 00 00 00 00 00 00 fc 00 
因沒有儲存媒介，因此返回最大格式化容量<br>

###CMD25:<br>
Bit 3:&nbsp;&nbsp;&nbsp;&nbsp; 0x08			容量列表長度為 8byte<br>
Bit 4-7:&nbsp;&nbsp;&nbsp;&nbsp;			sector數				0xd0e600	= 13690368<br>
Bit 9-11:&nbsp;&nbsp;&nbsp;&nbsp;			每sector的byte數		0x200	= 512<br>

###CMD27、28、29:<br>
<a href="https://ibb.co/f5oeom"><img src="https://preview.ibb.co/hcOC8m/23.png" alt="23" border="0"></a><br>
HOST希望執行0x12指令，(0x12 SCSI Inquiry命令) 12 01 80 00 ff 00<br>
Bit 1:	&nbsp;&nbsp;&nbsp;&nbsp;			EVPD 1,為要傳回特定資料，須與page配合使用，才能讀到正確資料<br>
Bit 2:	&nbsp;&nbsp;&nbsp;&nbsp;			Page code 0x80<br>
Bit 4:	&nbsp;&nbsp;&nbsp;&nbsp;			Allocation length 0xff<br>

###CMD30: 再讀取一次 Inquiry<br>
<a href="https://ibb.co/mM3eom"><img src="https://preview.ibb.co/hNrog6/24.png" alt="24" border="0"></a><br>
<a href="https://ibb.co/mnivTm"><img src="https://preview.ibb.co/dBT6ZR/25.png" alt="25" border="0"></a><br>
###CMD31-34: Read Capacity<br>
<a href="https://ibb.co/k5D9M6"><img src="https://preview.ibb.co/bxOn8m/26.png" alt="26" border="0"></a><br>

Command:<a href="https://ibb.co/dmQOg6"><img src="https://preview.ibb.co/fqtn8m/27.png" alt="27" border="0"></a><br>
Return:<a href="https://ibb.co/jekjom"><img src="https://preview.ibb.co/drWPom/28.png" alt="28" border="0"></a><br>

Last Logical Block Address:&nbsp;&nbsp;&nbsp;&nbsp;			0xe6cfff		15126527<br>
Block length in Bytes(512):&nbsp;&nbsp;&nbsp;&nbsp;			0x200		512<br>

###CMD35-38: Read<br>
<a href="https://ibb.co/k4ZoER"><img src="https://preview.ibb.co/gyezM6/29.png" alt="29" border="0"></a><br>
Command:	<a href="https://ibb.co/it5Og6"><img src="https://preview.ibb.co/cU1w16/30.png" alt="30" border="0"></a><br>
Logical Block Address:	&nbsp;&nbsp;&nbsp;&nbsp;		0x00<br>
Transfer Length:&nbsp;&nbsp;&nbsp;&nbsp;				0x01<br>
Return:	<a href="https://imgbb.com/"><img src="https://image.ibb.co/bTXm16/31.png" alt="31" border="0"></a><br>

##CMD39、40: Mode Sense(6) 1a<br>
<a href="https://ibb.co/f6Spom"><img src="https://preview.ibb.co/he2N8m/32.png" alt="32" border="0"></a><br>
<a href="https://ibb.co/bAsN8m"><img src="https://preview.ibb.co/cKK28m/33.png" alt="33" border="0"></a><br>
Page Code:&nbsp;&nbsp;&nbsp;&nbsp;				0x1c 查詢Informational Exceptions Control<br>
傳輸長度:&nbsp;&nbsp;&nbsp;&nbsp;				0xc0 192byte，<br>

###CMD41-43:<br>
<a href="https://ibb.co/dWUfuR"><img src="https://preview.ibb.co/fwMFTm/34.png" alt="34" border="0"></a><br>
僅回傳Mode Parameter Header 0x03<br>

CMD47-50:&nbsp;&nbsp;&nbsp;&nbsp; Read Capacity<br>
CMD51-54: &nbsp;&nbsp;&nbsp;&nbsp;Read<br>
……重複<br>

###CMD87-89: Test Unit Ready<br>

<a href="https://ibb.co/c8ciER"><img src="https://preview.ibb.co/j4HiER/35.png" alt="35" border="0"></a><br>
<a href="https://ibb.co/iX6AuR"><img src="https://preview.ibb.co/crs5Tm/36.png" alt="36" border="0"></a><br>
回傳&nbsp;&nbsp;&nbsp;&nbsp; ok &nbsp;&nbsp;&nbsp;&nbsp;表示準備就緒<br>

