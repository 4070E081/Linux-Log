# Linux-Log
核心日誌
主要是儲存系統開機時所顯示的訊息,以及一些核心與硬體溝通時的訊息(如:上一個 "insmod" 硬體裝置的模組),核心日誌存放在 /var/log/dmesg 或可使用 #dmesg 指令來看 /var/log/dmesg 的訊息.

# 系統日誌
首先先來看看系統日誌設定檔 /etc/syslog.conf 的內容

[root@benjr ~]# cat /etc/syslog.conf
#Log all kernel messages to the console.
#Logging much else clutters up the screen.
#kern.*       /dev/console
#Log anything (except mail) of level info or higher.
#Don't log private authentication messages!
*.info;mail.none;news.none;authpriv.none;cron.none  /var/log/messages
#The authpriv file has restricted access.
authpriv.*      /var/log/secure
#Log all the mail messages in one place.
mail.*          /var/log/maillog
#Log cron stuff
cron.*          /var/log/cron
#Everybody gets emergency messages
*.emerg       *
#Save news errors of level crit and higher in a special file.
uucp,news.crit      /var/log/spooler
#Save boot messages also to boot.log
local7.*       /var/log/boot.log

#INN

news.=crit     /var/log/news/news.crit
news.=err      /var/log/news/news.err
news.notice    /var/log/news/news.notice
以上的每一列都要包含的格式 facility.level action

## facility
用以代表訊息的來源, local0 至 local7 為使用者自定.系統預定的有
kern :kernel message
user : generic user level message
mail : mail system
daemon : other daemon
authpriv : security/authorization messages
syslog : internal syslog messages
lpr : printing system
news : news system
uucp : UUCP subsystem
authpriv : security/authorization messages
ftp : FTP daemon
cron : clock daemons(atd and crond)
## level(priority)
代表何種等級的錯誤訊息需要登錄,可選用的錯誤等級(由高至低):
“emerg” (0), “alert” (1),”crit” (2), “err” (3), “warning” (4), “notice” (5), “info” (6),”debug” (7).
使用*代表所有的錯誤皆登錄,還有一種為none代表系統停用此一登錄措施.
## action
代表系統日誌檔所存放的位址,除了本機可存放,還可以指定遠端機器來存放日誌檔.
系統可以用不同的方式決定哪些錯誤需要登錄.


 
. : 在指定的錯誤 level 以上將會被紀錄下來.
news 的錯誤 level 在 notice 以上 (包括 warning,err,crit,alert 與 emerg) 將會被紀錄下來.
news.notice
表示所有的錯誤訊息來源在 info 以上的層級都會紀錄.

*.info
.= : 只有這種錯誤 level 需要登錄
這樣的寫法會使系統只紀錄 news 的 crit 錯誤 level.
news.=crit
.! : 除了這個 facility 或 priority 其他皆要登錄
除了 local5 的錯誤 level "info" 其他 level (priority) 皆要登錄.
local5.!info
.* : 所有的 facility 或 priority 皆要登錄
authpriv 所有的錯誤皆要紀錄.
authpriv.*
接下來看看 /etc/syslog.conf 的內容

*.info 表示所有的錯誤訊息來源在 info 以上的層級都會紀錄至 /var/log/message 檔中.
mail.none ; news.none ; authpriv.none ; cron.none 表示 mail,news,authpriv,cron 的訊息將不紀錄至 /var/log/message 檔中.

*.info;mail.none;news.none;authpriv.none;cron.none  /var/log/messages
表示 authpriv , mail , cron 所有的錯誤皆要紀錄到指定的檔案.

authpriv.*      /var/log/secure
mail.*       /var/log/maillog
cron.*       /var/log/cron
除了會紀錄至日誌日誌檔外,還會在螢幕上秀出警告訊息.

*.emerg       *
uucp,news 的錯誤 level 在 crit 將會被紀錄下來

uucp,news.crit      /var/log/spooler
所有使用者自定 local 訊息,都會被記錄下,請參考後面的 自定的登錄措施.

local7.*      /var/log/boot.log
這樣的寫法會使系統只紀錄 news 的 crit,err 這兩個錯誤 level.


 
news.=crit        /var/log/news/news.crit
news.=err         /var/log/news/news.err
news 的錯誤 level 在 notice 以上將會被紀錄下來.

news.notice       /var/log/news/news.notice
# 自定的登錄措施
logger 指令不指使用.

[root@benjr ~]# logger "Testing"
記錄 info 以上錯誤 level 的訊息至 local5
在 /etc/syslog.conf 檔內加入 local5.info /var/log/local5 並重新啟動 syslog.
[root@benjr ~]# service syslog restart
利用logger工具將測試訊息寫入登錄檔.

[root@benjr ~]# logger -p local5.info "Script test for info"
[root@benjr ~]# logger -p local5.debug "Script test for debug"
[root@benjr ~]# logger -p local5.emerg  "Script test for emerg"
若 logger 不指定 -p 的參數,則會使用 user.notice 為預設.

查看登錄檔.

[root@benjr ~]# cat /var/log/local5
Oct 22 17:59:09 localhost root: Script test for info
Oct 22 18:00:44 localhost root: Script test for emerg
在這不會看到 "Script test for debug" 這一段訊息.是因為 debug 的 Level 在 info 之下 (錯誤等級 由低至高 debug,info,notice,warning,err,crit,alert 與 emerg) 所以不會紀錄至登錄檔.

只記錄 info 錯誤 level 的訊息至 local5
在 /etc/syslog.conf 檔內加入 local5.=info /var/log/local5
重新啟動 syslog

[root@benjr ~]# service syslog restart
利用 logger 工具將測試訊息寫入登錄檔.

[root@benjr ~]# logger -p local5.info "Script test for info"
[root@benjr ~]# logger -p local5.debug "Script test for debug"
[root@benjr ~]# logger -p local5.emerg  "Script test for emerg"
查看登錄檔

[root@benjr ~]# cat /var/log/local5
Oct 22 17:59:09 localhost root: Script test for info
在這不會看到"Script test for debug"以及"Script test for emerg"這兩段訊息.是因為我們已經指定只紀錄info 的錯誤level (local5.=info).

啟動 dhcpcd 系統預設的 local0 登入措施
dhcpcd (Dhcp client daemon)使用 syslog 的 local0 登入措施所以必須加入以下的步驟.
在 /etc/syslog.cong 檔內加入 local0.* /var/log/dhcpcd.log 並重新啟動 syslog.
[root@benjr ~]# service syslog restart
使用dhcpcd獲得 IP

[root@benjr ~]# dhcpcd -dn eth0
dhcpcd:MAC address:00:01:18:07:d4:37
查看 /var/log/dhcpcd.log

[root@benjr ~]# cat /var/log/dhcpcd.log
Oct 22 19:28:13 localhost dhcpcd[3739]: broadcasting DHCP_DISCOVER
Oct 22 19:29:13 localhost dhcpcd[3739]: timed out waiting for a valid DHCP server response
# 集中管理日誌檔(允許日誌訊息記載在遠端機器)
遠端機器(允許日誌訊息記載)

[root@benjr ~]# vi /etc/sysconfig/syslog
……….略………………………
SYSLOGD_OPTIONS="-r -m 0"
-m 0 設定每幾分鐘在 log 加上一行 —MARK— ( 0 代表不使用, 1 代表每一分鐘)
-r 允許遠端日誌訊息記載至本機
重新啟動 syslog

[root@benjr ~]# service syslog restart
Shutting down kernel logger:                    [ok]
Shutting down system logger:                   [ok]
Starting kernel logger:                              [ok]
Starting system logger:                             [ok]
本端機器(日誌訊息記載至遠端)

[root@benjr ~]# vi /etc/syslog.conf
……….略………………………
user.*         @remoteside
我們自訂一個 user 的所有訊息皆記載至 @remoteside (@表示機器名稱)這台機器上或可以使用 IP 來指定.重新啟動 syslog

[root@benjr ~]# service syslog restart
Shutting down kernel logger:                    [ok]
Shutting down system logger:                   [ok]
Starting kernel logger:                              [ok]
Starting system logger:                             [ok]
來試一下

[root@benjr ~]# logger -i -t yourname "This is a test"
logger 不指定 facility.level 時預設使用 user.notice,可以看到遠端 /var/log/message 多了一些訊息

[root@remote ~]# tail /var/log/messages
Sep 15 13:56:42 192.26.0.2 yourname[3494]:This is a test
# 日誌檔的循環機制
主要是使用 /etc/cron.daily/logrotate 檔,每日更新日誌檔,其內容如下
cron 的設定請參考 http://benjr.tw/421

[root@benjr ~]# cat /etc/cron.daily/logrotate
#!/bin/sh
/usr/sbin/logratate    /etc/logrotate.conf
logratate會依據 logrotate.conf 的內容來更新日誌檔,/etc/logrotate.conf 的內容如下:

[root@benjr ~]# cat /etc/logrotate.conf
#see "man logrotate" for details
#rotate log files weekly
weekly
#keep 4 weeks worth of backlogs
rotate 4
#create new (empty) log files after rotating old ones
create
#uncomment this if you want your log files compressed
#compress
#RPM packages drop log rotation information into this directory
include /etc/logrotate.d
#no packages own wtmp — we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
    rotate 1
}
#system-specific logs may be also be configured here.
其中 include /etc/logrotate.d 目錄下又包含 apache ftpd junkbuster mailman mars-nwe.log mgetty mysqld named psacct rpm samba sendfax slrnpull snmpd squid syslog tux up2date uucp vgetty vm vsftpd.log zebra

其內容都大致相同,我們就拿 /etc/logrotate.d/syslog 來看看

[root@benjr ~]# cat /etc/logrotate.d/syslog
/var/log/messages /var/log/secure /var/log/maillog /var/log/spooler /var/log/boot.log /var/log/cron {
    sharedscripts
    postrotate
 /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
也因此每個星期, message日誌檔都會被更新為 message1,message2…….
