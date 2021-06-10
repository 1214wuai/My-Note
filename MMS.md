# 彩信问题分析
![image](https://user-images.githubusercontent.com/40709975/121476378-2a616e00-c9f9-11eb-881a-e61222096010.png)

## radio.log
查看建pdn/APN：trySetupData for APN type|dsm-c|dsm-i

查找**data_call_list**

## android.log:
MmsNetworkManager: start new network request|MmsNetworkManager: already available|HTTP: 200 OK|HTTP: IO failure|MmsService: java.net.SocketTimeoutException: timeout|MmsNetworkManager: timed out|MmsNetworkManager: release

通过RCS发送的彩信：RCS_TAG

mmsdemo : -------->>>recipients

查找**mmsservice**
[SendRequest@a189fb2 messageId: 0] HTTP: 200 OK

@a189fb2代表同一条彩信，后续有重传根据这个信息来判断


 查找**rilj**
 release ,有可能释放PDN连接
 连上WIFI之后会释放PDN连接
 彩信发送成功：
```
	Line 23053: 10-09 12:48:13.538  1775  2047 D MmsServiceBroker: sendMessage() by com.oneplus.mmsdemo
	Line 23059: 10-09 12:48:13.541  7496  7782 D MmsService: sendMessage messageId: 0
	Line 23071: 10-09 12:48:13.548  7496  7782 D MmsService: Current running=0, current subId=-1, pending=0
	Line 23072: 10-09 12:48:13.548  7496  7782 D MmsService: Add request to running queue for subId 1
	Line 23073: 10-09 12:48:13.549  7496 16833 I MmsService: [SendRequest@14bb0da messageId: 0] Executing...
	Line 23074: 10-09 12:48:13.549  7496 16833 I MmsService: mms config for sub 1: Bundle[{httpSocketTimeout=60000, aliasMinChars=2, smsToMmsTextThreshold=-1, enableSMSDeliveryReports=true, maxMessageTextSize=-1, supportMmsContentDisposition=true, enabledTransID=false, aliasEnabled=false, supportHttpCharsetHeader=false, allowAttachAudio=true, smsToMmsTextLengthThreshold=-1, recipientLimit=2147483647, uaProfTagName=x-wap-profile, aliasMaxChars=48, maxImageHeight=1944, enableMMSDeliveryReports=false, userAgent=, mmsCloseConnection=true, config_cellBroadcastAppLinks=true, maxSubjectLength=40, httpParams=, enableGroupMms=true, emailGatewayNumber=, maxMessageSize=1048576, naiSuffix=, enableMMSReadReports=false, maxImageWidth=2592, uaProfUrl=, enabledMMS=true, enabledNotifyWapMMSC=false, sendMultipartSmsAsSeparateMessages=false, enableMultipartSMS=true}]
	Line 23327: 10-09 12:48:13.609  7496 16833 D MmsService: [SendRequest@14bb0da messageId: 0] MmsNetworkManager: start new network request
	Line 23663: 10-09 12:48:14.104  7496  7541 I MmsService: NetworkCallbackListener.onAvailable: network=109
	Line 23667: 10-09 12:48:14.105  7496 16833 D MmsService: [SendRequest@14bb0da messageId: 0] APN name is fast.t-mobile.com
	Line 23668: 10-09 12:48:14.105  7496 16833 I MmsService: [SendRequest@14bb0da messageId: 0] Loading APN using name fast.t-mobile.com
	Line 23682: 10-09 12:48:14.112  7496 16833 I MmsService: [SendRequest@14bb0da messageId: 0] Using APN [type=default,supl,mms,xcap mmsc=http://mms.msg.eng.t-mobile.com/mms/wapenc name=T-Mobile LTE apn=fast.t-mobile.com bearer_bitmask=0 protocol=IPV6 roaming_protocol=IP authtype=-1]
	Line 23689: 10-09 12:48:14.114  7496 16833 D MmsService: [SendRequest@14bb0da messageId: 0] HTTP: POST http://mms.msg.eng.t-mobile.com[42], PDU size=105350
	Line 23691: 10-09 12:48:14.114  7496 16833 I MmsService: [SendRequest@14bb0da messageId: 0] HTTP: User-Agent=Android-Mms/2.0
	Line 23692: 10-09 12:48:14.114  7496 16833 I MmsService: [SendRequest@14bb0da messageId: 0] HTTP: UaProfUrl=http://www.google.com/oha/rdf/ua-profile-kila.xml, UaProfUrlTagName=x-wap-profile
	Line 23693: 10-09 12:48:14.114  7496 16833 I MmsService: [SendRequest@14bb0da messageId: 0] HTTP: Connection close after request
	Line 24150: 10-09 12:48:15.197  7496 16833 D MmsService: [SendRequest@14bb0da messageId: 0] HTTP: 200 OK
	Line 24151: 10-09 12:48:15.197  7496 16833 D MmsService: [SendRequest@14bb0da messageId: 0] HTTP: response size=68
	Line 24152: 10-09 12:48:15.197  7496 16833 D MmsService: [SendRequest@14bb0da messageId: 0] MmsNetworkManager: release, count=0
	Line 24159: 10-09 12:48:15.199  7496 16833 D MmsService: [SendRequest@14bb0da messageId: 0] persistIfRequired. messageId: 0
	Line 24260: 10-09 12:48:15.249  7496 16833 I MmsService: [SendRequest@14bb0da messageId: 0] processResult: -1, httpStatusCode: success (0)
	Line 24294: 10-09 12:48:15.252  7496 16833 D MmsService: Schedule requests pending on SIM
```
彩信发送失败
```
10-09 12:52:37.100  1775  7485 D MmsServiceBroker: sendMessage() by com.oneplus.mmsdemo
10-09 12:52:37.103  7496  7631 D MmsService: sendMessage messageId: 0
10-09 12:52:37.107  7496  7631 D MmsService: Current running=0, current subId=-1, pending=0
10-09 12:52:37.107  7496  7631 D MmsService: Add request to running queue for subId 1
10-09 12:52:37.107  7496 17357 I MmsService: [SendRequest@6aa3994 messageId: 0] Executing...
10-09 12:52:37.107  7496 17357 I MmsService: mms config for sub 1: Bundle[{httpSocketTimeout=60000, aliasMinChars=2, smsToMmsTextThreshold=-1, enableSMSDeliveryReports=true, maxMessageTextSize=-1, supportMmsContentDisposition=true, enabledTransID=false, aliasEnabled=false, supportHttpCharsetHeader=false, allowAttachAudio=true, smsToMmsTextLengthThreshold=-1, recipientLimit=2147483647, uaProfTagName=x-wap-profile, aliasMaxChars=48, maxImageHeight=1944, enableMMSDeliveryReports=false, userAgent=, mmsCloseConnection=true, config_cellBroadcastAppLinks=true, maxSubjectLength=40, httpParams=, enableGroupMms=true, emailGatewayNumber=, maxMessageSize=1048576, naiSuffix=, enableMMSReadReports=false, maxImageWidth=2592, uaProfUrl=, enabledMMS=true, enabledNotifyWapMMSC=false, sendMultipartSmsAsSeparateMessages=false, enableMultipartSMS=true}]
10-09 12:52:37.146  7496 17357 D MmsService: [SendRequest@6aa3994 messageId: 0] MmsNetworkManager: start new network request
10-09 12:52:37.149  7496  7541 I MmsService: NetworkCallbackListener.onAvailable: network=110
10-09 12:52:37.150  7496 17357 D MmsService: [SendRequest@6aa3994 messageId: 0] APN name is fast.t-mobile.com
10-09 12:52:37.150  7496 17357 I MmsService: [SendRequest@6aa3994 messageId: 0] Loading APN using name fast.t-mobile.com
10-09 12:52:37.157  7496 17357 I MmsService: [SendRequest@6aa3994 messageId: 0] Using APN [type=default,supl,mms,xcap mmsc=http://mms.msg.eng.t-mobile.com/mms/wapenc name=T-Mobile LTE apn=fast.t-mobile.com bearer_bitmask=0 protocol=IPV6 roaming_protocol=IP authtype=-1]
10-09 12:52:37.160  7496 17357 D MmsService: [SendRequest@6aa3994 messageId: 0] HTTP: POST http://mms.msg.eng.t-mobile.com[42], PDU size=105350
10-09 12:52:37.160  7496 17357 I MmsService: [SendRequest@6aa3994 messageId: 0] HTTP: User-Agent=Android-Mms/2.0
10-09 12:52:37.160  7496 17357 I MmsService: [SendRequest@6aa3994 messageId: 0] HTTP: UaProfUrl=http://www.google.com/oha/rdf/ua-profile-kila.xml, UaProfUrlTagName=x-wap-profile
10-09 12:52:37.160  7496 17357 I MmsService: [SendRequest@6aa3994 messageId: 0] HTTP: Connection close after request
10-09 12:53:37.303  7496 17357 E MmsService: [SendRequest@6aa3994 messageId: 0] HTTP: IO failure
```
## radio log
去radio log，查找**data_call_list**，找到对应的时间点，ipv6一般是ims的ip，ipv4v6中的ipv6，就是彩信中心那边的ip，TMO这边彩信和数据域走同一条连接。
## tcpdump
搜索mmse，找到对应的时间点，req和conference
查看tcpdump，搜索**ipv6.addr == xxxxx && ipv6 == xxxx**
统计--->流量图--->点击限制显示过滤--->流类型选择TCP Flows

# 上行分析
DRB:数据承载，SRB:信令承载

## 1.DRB
先找到DRB，不同类型的，彩信和数据使用同一个PDN，PDN建立完成之后，网络侧会回一个RRC Reconfiguration，里边有DRB，
```
2021 Jun  3  05:53:34.631  [09]  0xB0B0  LTE PDCP UL Config
Subscription ID = 2
Version = 1
Number Subpackets = 1
Subpacket[0]
   Subpacket ID = PDCP UL config (0xC1)
   Subpacket Version = 24
   Subpacket Size = 68 bytes
   Subpacket1 Ver24 {
      Reason = Configuration
      Security Config:
         SRB Cipher Algo = None
         SRB Cipher Key Idx = 12
         SRB Integ Algo = None
         SRB Integ Key Idx = 11
         DRB Cipher Algo = None
         DRB Cipher Key Idx = 13
      Number released RB = 0
      Number added/modified RB = 1
      Added/Modified RB
         -------------------
         |RB Cfg Idx|Action|
         -------------------
         |        33|   Add|

      Number active RB = 1
      Active RB
         -------------------------------------------------------------------------------------------------------------
         |          |  |      |   |    |    |SN    |Discard |           |RoHC|          |        |UDC    |UDC |UDC   |
         |          |RB|RB-Cfg|EPS|RB  |RB  |Length|timer   |Compression|Max |          |UDC Cfg |Context|Algo|Header|
         |Action    |ID|Idx   |ID |mode|Type|(bits)|(ms)    |Type       |CID |RoHC Mask |Action  |ID     |Ver |Length|
         -------------------------------------------------------------------------------------------------------------
         |PDCPUL CFG|1 |  33  | 1 | AM |SRB |  5   |INFINITY|   NONE    |n/a |   n/a    |  n/a   |  n/a  |n/a | n/a  |

   }
```
## 2.UL PCCP
0xB0B3 LTE PDCP UL Cipher Data PDU
sys_fn和sub_fn
```
2021 Jun  3  05:53:34.790  [CE]  0xB0B3  LTE PDCP UL Cipher Data PDU
Subscription ID = 2
Version = 1
Num Subpackets = 1
Subpacket[0]
   Subpacket ID = PDCP PDU with Ciphering (0xC3)
   Subpacket Version = 26
   Subpacket Size = 56 bytes
   SRB Ciphering Keys (hex) =  E2 77 7D 62 36 EC E6 ED FA 30 2D AF 7D EB 16 A9
   DRB Ciphering Keys (hex) =  90 7C B8 02 17 A9 0B 84 17 7D 5E A0 0F 2D AE 6B
   SRB Cipher Algo = LTE AES
   DRB Cipher Algo = LTE AES
   Num PDUs = 1
   --------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   |                 |   |    |      |      |     |     |      |      |      |          |     |          |        |els |       |        |   |      |                        |
   |                 |cfg|    |sn    |bearer|valid|pdu  |logged|      |      |count     |     |compressed|        |mini|packet |        |   |      |                        |
   |PDCPUL CIPH DATA |idx|mode|length|id    |pdu  |size |bytes |sys_fn|sub_fn|(hex)     |sn   |pdu       |pdu type|sign|action |checksum|e  |option|log_buffer (hex)        |
   --------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   |PDCPUL CIPH DATA |33 | AM |5 bit |  0   | Yes |  7  |  3   | 596  |  5   |   0x3    |  3  |    No    | DEFAULT|n/a |  n/a  |  n/a   |n/a| n/a  | 03 14 00               |
```


## 3.UL RLC
0xB092 LTE RLC UL AM All PDU
```
2021 Jun  3  05:53:34.790  [CE]  0xB092  LTE RLC UL AM All PDU 
Subscription ID = 2
Version = 1
Number of SubPackets = 1
Subpacket ID = 70
SubPacket - ( RLCUL PDU )
   Version = 3
   Subpacket Size = 64 bytes
   RB Cfg Idx = 33, RB Mode = AM, SN Length = 10 bits
   Reserved = 0
   Enabled PDU Log Packets:
      RLCUL Config (0xB091) = 1
      RLCUL AM ALL PDU (0xB092) = 1
      RLCUL AM CONTROL PDU (0xB093) = 1
      RLCUL AM POLLING PDU (0xB094) = 1
      RLCUL AM SIGNALING PDU (0xB095) = 1
      RLCUL UM DATA PDU (0xB096) = 1
      RLCUL STATISTICS (0xB097) = 1
   VT(A) = 2, VT(S) = 3, PDU Without Poll = 0, Byte Without Poll = 0, Poll SN = 2
   Number of PDUs = 4
   RLCUL PDU[0]
      PDU TYPE = RLCUL CTRL, rb_cfg_idx =  33, ACK_SN =    2, sys_fn = 594, sub_fn = 7, pdu_bytes = 2, cpt = STATUS (0)
      RLCUL CTRL : ACK_SN = 2
      Hex Dump = {  00 08 }
   RLCUL PDU[1]
      PDU TYPE = RLCUL DATA, rb_cfg_idx =  33, SN     =    1, sys_fn = 594, sub_fn = 7, pdu_bytes = 18, RF = 0, P = 1, FI = 00, E = 1
      ---------------------------------------------------------------
      |RLCUL DATA LI|LI   |LI   |LI   |LI   |LI   |LI   |LI   |LI   |
      ---------------------------------------------------------------
      |RLCUL DATA LI|    7|     |     |     |     |     |     |     |

      Hex Dump = {  A4 01 00 70 }
   RLCUL PDU[2]
      PDU TYPE = RLCUL CTRL, rb_cfg_idx =  33, ACK_SN =    3, sys_fn = 596, sub_fn = 0, pdu_bytes = 2, cpt = STATUS (0)
      RLCUL CTRL : ACK_SN = 3
      Hex Dump = {  00 0C }
   RLCUL PDU[3]
      PDU TYPE = RLCUL DATA, rb_cfg_idx =  33, SN     =    2, sys_fn = 596, sub_fn = 5, pdu_bytes = 9, RF = 0, P = 1, FI = 00, E = 0
      Hex Dump = {  A0 02 }
```

## 4.UL MAC
MAC的grant ok
0xB064 LTE MAC UL Transport Block
使用的无线网络临时标识符（RNTI），Grant、RLC PDU、缓冲区状态报告（BSR）类型和内容
```
2021 Jun  3  05:53:34.792  [95]  0xB064  LTE MAC UL Transport Block
Subscription ID = 2
Version = 1
Number of SubPackets = 1
SubPacket ID = 8
SubPacket - ( UL Transport Block Subpacket )
   Version = 2
   Subpacket Size = 208
   Uplink Transport Block V2
      Number of samples = 11
      ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      |Sub|Cell|       |          |      |     |Grant  |RLC  |Padding|                 |               |HDR  |                        |          |     |BSR  |BSR  |BSR  |BSR  |       |BSR LCG 0|BSR LCG 1|BSR LCG 2|BSR LCG 3|       |PH    |Pcmax_c|PH    |Pcmax_c|
      |Id |Id  |HARQ ID|RNTI Type |Sub-FN|SFN  |(bytes)|PDUs |(bytes)|BSR event        |BSR trig       |LEN  |Mac Hdr + CE            |LC ID     |LEN  |LCG 0|LCG 1|LCG 2|LCG 3|PHR Ind|(bytes)  |(bytes)  |(bytes)  |(bytes)  |Pcmax_c|SCell1|SCell1 |SCell2|SCell2 |
      ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      |  2|   0|      1|    C-RNTI|     7|  594|     85|    3|      0|High Data Arrival|          S-BSR|    7| 3D 21 02 21 12 04 23   |     S-BSR|    1|   35|     |     |     |       |     2127|         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         1|    2|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         1|   18|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         4|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      2|    C-RNTI|     6|  595|    333|    1|      0|             None|         No BSR|    1| 04                     |         4|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      3|    C-RNTI|     7|  595|    333|    1|      0|         Periodic|          S-BSR|    3| 3D 04 20               |     S-BSR|    1|   32|     |     |     |       |     1326|         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         4|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      4|    C-RNTI|     8|  595|    333|    1|      0|             None|         No BSR|    1| 04                     |         4|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      5|    C-RNTI|     9|  595|    333|    1|      0|             None|         No BSR|    1| 04                     |         4|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      6|    C-RNTI|     0|  596|    333|    2|      0|High Data Arrival|          S-BSR|    5| 3D 21 02 04 16         |     S-BSR|    1|   22|     |     |     |       |      274|         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         1|    2|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         4|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      7|    C-RNTI|     1|  596|    285|    1|      7|             None|      Pad L-BSR|    8| 3E 24 81 0E 1F 00 00 00|     L-BSR|    3|    0|    0|    0|    0|       |        0|        0|        0|        0|       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         4|  270|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |   Padding|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      0|    C-RNTI|     2|  596|    209|    0|    204|             None|      Pad L-BSR|    5| 3E 1F 00 00 00         |     L-BSR|    3|    0|    0|    0|    0|       |        0|        0|        0|        0|       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |   Padding|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      3|    C-RNTI|     5|  596|    333|    1|    319|High Data Arrival|          S-BSR|    5| 3D 21 09 1F 00         |     S-BSR|    1|    0|     |     |     |       |        0|         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |         1|    9|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |   Padding|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      4|    C-RNTI|     6|  596|    333|    0|    328|             None|      Pad L-BSR|    5| 3E 1F 00 00 00         |     L-BSR|    3|    0|    0|    0|    0|       |        0|        0|        0|        0|       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |   Padding|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |
      |  2|   0|      5|    C-RNTI|     7|  596|    317|    0|    312|             None|      Pad L-BSR|    5| 3E 1F 00 00 00         |     L-BSR|    3|    0|    0|    0|    0|       |        0|        0|        0|        0|       |      |       |      |       |
      |   |    |       |          |      |     |       |     |       |                 |               |     |                        |   Padding|   -1|     |     |     |     |       |         |         |         |         |       |      |       |      |       |


```
## 5.PHY
0xB172 LTE Uplink PKT Build Indication
```
2021 Jun  3  05:53:25.361  [91]  0xB172  LTE Uplink PKT Build Indication
Subscription ID = 1
Version              = 25
Number of Records    = 8
PKT Build Record
   ------------------------------------------------------------------------------------------------
   |   |     |    |      |Transport|          |              |Power   |    |    |       |         |
   |   |Cell |Tx  |Tx    |Block    |EIB       |              |Headroom|HARQ|Tx  |Corrupt|Commit   |
   |#  |Index|Sfn |Sub-fn|Size     |Address   |RNTI Type     |(dB)    |ID  |Type|CRC    |Time     |
   ------------------------------------------------------------------------------------------------
   |  0|    0| 581|     1|      217|0x000024E0|        C_RNTI|     -23|   1| New|     No|  3180978|
   |  1|    0| 581|     9|      217|0x000024E0|        C_RNTI|     -23|   1| DTX|     No|  3334577|
   |  2|    0| 582|     7|      217|0x000024E0|        C_RNTI|     -23|   1| DTX|     No|  3488177|
   |  3|    0| 583|     5|      217|0x000024E0|        C_RNTI|     -23|   1| DTX|     No|  3641777|
   |  4|    0| 669|     1|      217|0x000024E0|        C_RNTI|     -23|   1| New|     No|  3299744|
   |  5|    0| 669|     9|      217|0x000024E0|        C_RNTI|     -23|   1| DTX|     No|  3453344|
   |  6|    0| 670|     7|      217|0x000024E0|        C_RNTI|     -23|   1| DTX|     No|  3606944|
   |  7|    0| 671|     5|      217|0x000024E0|        C_RNTI|     -23|   1| DTX|     No|  3760544|
```
//PDCCH-PHICH收到上行数据的ACK
0xB16B LTE PDCCH-PHICH Indication Report
PHICH Value--Ack at 596/5 for UL Txn at 596/5(B092)
```
2021 Jun  3  05:53:34.788  [12]  0xB16B  LTE PDCCH-PHICH Indication Report
Subscription ID = 2
Version              = 25
Duplex Mode          = FDD
Number of Records    = 25
Info Records
   -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   |   |       |       |      |      |PHICH                              |PDCCH Info                                                                                                         |
   |   |Num    |Num    |PDCCH |PDCCH |     |        |        |     |PHICH|Serv |              |PDCCH  |           |      |          |     |      |     |     |     |     |      |      |Fake |
   |   |PDCCH  |PHICH  |Timing|Timing|Cell |PHICH   |PHICH 1 |PHICH|1    |Cell |              |Payload|Aggregation|Search|SPS Grant |New  |Num DL|S0   |S1   |S2   |S3   |      |      |Pdcch|
   |#  |Results|Results|SFN   |Sub-fn|Index|Included|Included|Value|Value|Index|RNTI Type     |Size   |Level      |Space |Type      |DL Tx|Trblks|Index|Index|Index|Index|Msleep|Usleep|Sf   |
   -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   |  0|      2|      0|   595|     4|     |        |        |     |     |    0|        C_RNTI|     44|       Agg2|    UE|          | true|     1|    0|    0|    0|    0|     0|     0|    0|
   |   |       |       |      |      |     |        |        |     |     |    0|        C_RNTI|     44|       Agg8|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   |  1|      1|      0|   595|     5|     |        |        |     |     |    0|        C_RNTI|     44|       Agg8|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   |  2|      1|      0|   595|     6|     |        |        |     |     |    0|        C_RNTI|     44|       Agg8|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   |  3|      1|      0|   595|     7|     |        |        |     |     |    0|        C_RNTI|     44|       Agg8|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   |  4|      1|      0|   595|     8|     |        |        |     |     |    0|        C_RNTI|     44|       Agg1|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   |  5|      0|      0|   595|     9|     |        |        |     |     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   |  6|      0|      1|   596|     0|    0|     Yes|      No|  ACK|     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   |  7|      1|      1|   596|     1|    0|     Yes|      No|  ACK|     |    0|        C_RNTI|     44|       Agg8|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   |  8|      1|      1|   596|     2|    0|     Yes|      No|  ACK|     |    0|        C_RNTI|     44|       Agg8|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   |  9|      1|      1|   596|     3|    0|     Yes|      No|  ACK|     |    0|        C_RNTI|     44|       Agg2|    UE|          |false|     0|    0|    0|    0|    0|     0|     0|    0|
   | 10|      0|      1|   596|     4|    0|     Yes|      No|  ACK|     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   | 11|      1|      1|   596|     5|    0|     Yes|      No|  ACK|     |    0|        C_RNTI|     64|       Agg8|    UE|          | true|     1|    0|    0|    0|    0|     0|     0|    0|
   | 12|      0|      1|   596|     6|    0|     Yes|      No|  ACK|     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   | 13|      0|      0|   596|     7|     |        |        |     |     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   | 14|      0|      0|   596|     8|     |        |        |     |     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   | 15|      1|      1|   596|     9|    0|     Yes|      No|  ACK|     |    0|        C_RNTI|     44|       Agg8|    UE|          | true|     1|    0|    0|    0|    0|     0|     0|    0|
   | 16|      0|      1|   597|     0|    0|     Yes|      No|  ACK|     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   | 17|      0|      1|   597|     1|    0|     Yes|      No|  ACK|     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
   | 18|      0|      0|   597|     2|     |        |        |     |     |     |              |       |           |      |          |     |      |     |     |     |     |     0|     0|    0|
```
# 下行DL分析

0xB173 LTE PDSCH Stat Indication
Throughput = Total Bytes * 8 / 1000 / Elapsed Time in seconds
Phy layer Tput ~ 108\*8\*2/1000 = 1.728Mbps,其中\*2是因为Num Layers是2
```
2021 Jun  3  05:53:34.906  [B6]  0xB173  LTE PDSCH Stat Indication
Subscription ID = 2
Version      = 25
Num Records  = 11
Records
   ------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   |   |        |     |   |      |Num      |       |        |Transport Blocks                                                                                   |    |    |
   |   |        |     |   |      |Transport|Serving|        |    |  |   |      |         |     |Discarded|           |       |   |   |          |               |    |    |
   |   |Subframe|Frame|Num|Num   |Blocks   |Cell   |HSIC    |HARQ|  |   |CRC   |         |TB   |reTx     |Did        |TB Size|   |Num|Modulation|ACK/NACK       |PMCH|Area|
   |#  |Num     |Num  |RBs|Layers|Present  |Index  |Enabled |ID  |RV|NDI|Result|RNTI Type|Index|Present  |Recombining|(bytes)|MCS|RBs|Type      |Decision       |ID  |ID  |
   ------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   |  0|       5|  580|  5|     2|        1|  PCELL|Disabled|   1| 0|  0|  Pass|       RA|    0|     None|         No|     18|  4|  5|      QPSK|            ACK|    |    |
   |  1|       7|  581| 12|     2|        1|  PCELL|Disabled|   7| 0|  1|  Pass|   Temp-C|    0|     None|         No|     44|  0| 12|      QPSK|            ACK|    |    |
   |  2|       6|  582|  4|     2|        1|  PCELL|Disabled|   7| 0|  0|  Pass|        C|    0|     None|         No|     14|  0|  4|      QPSK|            ACK|    |    |
   |  3|       7|  588|  4|     2|        1|  PCELL|Disabled|   7| 0|  1|  Pass|        C|    0|     None|         No|     14|  0|  4|      QPSK|            ACK|    |    |
   |  4|       1|  592|  4|     2|        1|  PCELL|Disabled|   7| 0|  0|  Pass|        C|    0|     None|         No|     25|  2|  4|      QPSK|            ACK|    |    |
   |  5|       2|  592| 12|     2|        1|  PCELL|Disabled|   6| 0|  1|  Pass|        C|    0|     None|         No|     68|  2| 12|      QPSK|            ACK|    |    |
   |  6|       1|  595|  4|     2|        1|  PCELL|Disabled|   7| 0|  1|  Pass|        C|    0|     None|         No|     25|  2|  4|      QPSK|            ACK|    |    |
   |  7|       4|  595| 16|     2|        1|  PCELL|Disabled|   6| 0|  0|  Pass|        C|    0|     None|         No|     90|  2| 16|      QPSK|            ACK|    |    |
   |  8|       5|  596|  2|     2|        1|  PCELL|Disabled|   7| 0|  0|  Pass|        C|    0|     None|         No|     12|  2|  2|      QPSK|            ACK|    |    |
   |  9|       9|  596|  4|     2|        1|  PCELL|Disabled|   6| 0|  1|  Pass|        C|    0|     None|         No|     25|  2|  4|      QPSK|            ACK|    |    |
   | 10|       3|  605| 12|     2|        1|  PCELL|Disabled|   7| 0|  1|  Pass|        C|    0|     None|         No|    108|  4| 12|      QPSK|            ACK|    |    |
```


虽然没有空间复用，但在DL中定义了两层。使用的是发射Deversty，表示DL-RF覆盖较弱
UE没有在连续的子帧中被调度-吞吐量下降的另一个原因
DL传输成功，然而eNB重新传输相同的PDU，指示eNB没有从UE接收UL ACK



//0xB063 LTE MAC DL Transport Block
mac padding--表示eNB传输缓冲区中缺少daa（回程限制）
```
2021 Jun  3  05:52:51.250  [CA]  0xB063  LTE MAC DL Transport Block
Subscription ID = 1
Version = 1
Number of SubPackets = 1
SubPacket ID = 7
SubPacket - ( DL Transport Block Subpacket )
   Version = 4
   Subpacket Size = 56
   Downlink Transport Block V4
      Number of samples = 3
      ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      |   |    |      |     |          |       |    |    |       |    |     |       |     |                        |               |     |   |     |Absolute|    |      |      |     |   |     |   |        |Relative|
      |Sub|Cell|      |     |          |       |Area|PMCH|DL TBS |RLC |EMBMS|       |HDR  |                        |               |     |BI |Rapid|TA Val  |Hop |RB    |Coding|TBS  |TPC|UL   |CQI|        |TA Val  |
      |Id |Id  |Sub-FN|SFN  |RNTI Type |HARQ ID|ID  |ID  |(bytes)|PDUs|PDUs |Padding|LEN  |Mac Hdr + CE            |LC ID          |LEN  |Val|Val  |(16xTs) |Flag|Assign|Scheme|Index|dB |Delay|Req|T-C-RNTI|(16xTs) |
      ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      |  1|   0|     0|  335|    C-RNTI|      6|    |    |     22|   1|     |     17|    3| 23 02 1F               |              3|    2|   |     |        |    |      |      |     |   |     |   |        |        |
      |   |    |      |     |          |       |    |    |       |    |     |       |     |                        |        Padding|   -1|   |     |        |    |      |      |     |   |     |   |        |        |
      |  1|   0|     7|  335|    C-RNTI|      7|    |    |    121|   1|     |     26|    3| 24 5C 1F               |              4|   92|   |     |        |    |      |      |     |   |     |   |        |        |
      |   |    |      |     |          |       |    |    |       |    |     |       |     |                        |        Padding|   -1|   |     |        |    |      |      |     |   |     |   |        |        |
      |  1|   0|     1|  339|    C-RNTI|      7|    |    |     15|   1|     |     10|    3| 23 02 1F               |              3|    2|   |     |        |    |      |      |     |   |     |   |        |        |
      |   |    |      |     |          |       |    |    |       |    |     |       |     |                        |        Padding|   -1|   |     |        |    |      |      |     |   |     |   |        |        |


```
