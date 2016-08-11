title: 如何通过shell监控CDN是否正常
date: 2015-06-09 18:28:59 星期二 
tags: [shell,CDN]
categories: [linux]

---

# 写在前面 #
先说说基本的思路，然后再贴上详细的代码

<!-- more -->


```bash
#!/bin/bash 
# 
#script_name:chk_cdn.sh 
#check cdn index.html 
#Domain name resolution 
# 
#last update 20130320 by dongnan 
#bbs# http://bbs.ywwd.net/ 
#blog# http://dngood.blog.51cto.com 
# 
#tail error.log  
#curl: (6) name lookup timed out 
#variables  
#check_time=2        #检查间隔时间,2s 
check_count=8        #故障后检查次数,8次 
fault_count=4        #故障次数大于(4次),则认为不可用 
www_url="http://www.test.com"
domain=www.test.com 
sh_dir=/root/sh/ 
crondir=${sh_dir}crontab 
error_log=${crondir}/log/cdn_error.log 
flag_file=${crondir}/log/cdn.flag 
echo=/bin/echo 
curl=/usr/bin/curl 
options='--connect-timeout 3 -IL -A "check_cdn_www"'
source ${sh_dir}/CONFIG 
#main 
test -e "${crondir}/log" || mkdir -p "${crondir}/log" 
#true 
if $curl $options $www_url > /dev/null 2>&1 ;then 
#flag 
    if [ -f $flag_file ];then 
        #sms 
        #for mobile in $MOBILES; do 
            #$echo "cdn_www ok" | /usr/local/bin/gammu --sendsms TEXT "$mobile" -unicode 
            #$echo "cdn_www_index.html ok" 
        $done 
        #email 
        for mail in $MAILS;do 
            $echo "$domain ok" | mail -s "cdn ok" $mail 
        done 
        #flag 
        test -e "$flag_file" && rm -f "$flag_file" 
    fi 
#false 
else 
#flag真退出脚本 
    test -e $flag_file && exit 0 
check_failed=0
    # 
    for((i=1;i<="$check_count";i++));do 
check_date=$(date '+ %F %T') 
        #curl 值取反 
        if ! $curl $options $www_url > /dev/null 2>&1;then 
            #变量加1 
            ((check_failed++)) 
            #error.log 
            $echo "$(/bin/date +'%F %T') $www_url $check_failed fault" >> "$error_log" 
            /usr/bin/dig "$domain" >> "$error_log" 
            # 
            sleep 1 
        fi 
    done 
    # 
    if [ "$check_failed" -gt "$fault_count" ];then 
         #sms 
         #for mobile in $MOBILES;do 
             #$echo "www_cdn_index.html error" | /usr/local/bin/gammu --sendsms TEXT "$mobile" -unicode 
             #/bin/date +'%F %T' && $echo "www_cdn_index.html error" 
         #done 
         #email 
         for mail in $MAILS;do 
             $echo "$domain error" | mail -s "cdn error" $mail 
         done 
         #flag 
         $echo "cdn_www_index.html error" > "$flag_file" 
         #log 
    fi 
fi 
```