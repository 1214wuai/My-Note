# My-Note
LTE、NR
# 发送短信
## radio.log
搜索
ImsSmsDispacher|ImsSmsDispatcher|onSendSmsResult |SMSDispatcher: SMS retry|SMSDispatcher: sendText|sendSms|sendSms: return fail|< SEND_SMS|GsmSMSDispatcher

搜isVolteEnabled，可以看到上报的IMS状态

ImsSmsDispatcher [0]: isAvailable: up=true, reg= false, cap= false
reg=false，代表断开了IMS

出现了RILJ：代表UE的IMS断开了，尝试从CS发

失败：messageRef=0
ImsSmsDispatcher [0]: onSendSmsResult token=29 messageRef=0 status=2 reason=16 networkReasonCode=-1


sms 发送，主要搜SmsDispatchersms

# 接收短信
## radio.log
搜索**GsmInboundSmsHandler | SMS receiver**

sms 结束，主要搜InboundSmsHandle

# Android.log
IMS_SMS

# WMS错误
## case：05178476
rp_cause = WMS_RP_CAUSE_PROTOCOL_ERROR
tp_cause = WMS_TP_CAUSE_SC_SYS_FAILURE

## case：05203399
rp_cause = WMS_RP_CAUSE_UNKNOWN_SUBSCRIBER
tp_cause = 0

## case：05211050
rp_cause = WMS_RP_CAUSE_SMS_TRANSFER_REJECTEDtp_cause = WMS_TP_CAUSE_DESTINATION_SME_BARRED
