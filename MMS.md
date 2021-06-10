# 彩信问题分析
[toc]
android.log:
MmsNetworkManager: start new network request|MmsNetworkManager: already available|HTTP: 200 OK|HTTP: IO failure|MmsService: java.net.SocketTimeoutException: timeout|MmsNetworkManager: timed out|MmsNetworkManager: release

android.log:
通过RCS发送的彩信：RCS_TAG
## radio.log
查看建pdn：
trySetupData for APN type|dsm-c|dsm-i
查找**data_call_list**
查看APN建立：
dsm-i|trySetupData for APN type 
## Andriod log

mmsdemo : -------->>>recipients

查找**mmsservice**
[SendRequest@a189fb2 messageId: 0] HTTP: 200 OK

@a189fb2代表同一条彩信，后续有重传根据这个信息来判断

 ## radio.log
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

1. 先找到DRB，不同类型的，彩信和数据使用同一个PDN，PDN建立完成之后，网络侧会回一个RRC Reconfiguration，里边有DRB，
```
//0xB0B0
2020 Oct  9  19:52:14.890  [AA]  0xB0B0  LTE PDCP UL Config
Released RB         -----         |RB |         |Cfg|         |Idx|         -----         |  6|
```
0xB0B3 LTE PDCP UL Cipher Data PDU
sys_fn和sub_fn

![7db8d369c0269fcc3207c9e034437ee7.jpeg](en-resource://database/513:1)

//UL RLC
0xB092 LTE RLC UL AM All PDU

//MAC的grant ok
0xB064 LTE MAC UL Transport Block
使用的无线网络临时标识符（RNTI），Grant、RLC PDU、缓冲区状态报告（BSR）类型和内容
![17344f2a2d6338bba9a3eaf9421f0212.png](en-resource://database/507:1)

//PHY
0xB172 LTE Uplink PKT Build Indication

//PDCCH-PHICH收到上行数据的ACK
0xB16B LTE PDCCH-PHICH Indication Report
PHICH Value--Ack at 625/5 for UL Txn at 625/1
![37e7d5c6c1e5511a9630757d85da9bb0.png](en-resource://database/509:1)

# 下行DL分析

0xB173 LTE PDSCH Stat Indication
![bb7280913c5d515a24e18227c9c22896.png](en-resource://database/508:1)

![07cb4ee684b7e28120ad6e3e49b8c689.png](en-resource://database/506:1)
虽然没有空间复用，但在DL中定义了两层。使用的是发射Deversty，表示DL-RF覆盖较弱
![ef7bc7e914c798aeab0828b8d4c27f18.png](en-resource://database/511:1)
UE没有在连续的子帧中被调度-吞吐量下降的另一个原因
DL传输成功，然而eNB重新传输相同的PDU，指示eNB没有从UE接收UL ACK
![ce8fafd323dca064745d5d76b65248f4.png](en-resource://database/510:1)


//0xB063 LTE MAC DL Transport Block
mac padding--表示eNB传输缓冲区中缺少daa（回程限制）
![701a9ac7898d33e7a930cd68e11ffe07.png](en-resource://database/512:1)
