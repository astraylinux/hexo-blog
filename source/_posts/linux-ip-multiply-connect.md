title: ip批量连接测试
date: 2012-02-21 19:47:00
categories:
- Linux
tags:
- Bash
- Linux
- server
---

Program: scan the ip from $1 to $2 ,output the ip which your can connect
History: build by AstrayLinux in 2011/10/19


```bash
#!/bin/bash
PATH=/home/ocean/bin/script:/home/ocean/bin:/home/ocean/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
export PATH

net=""
net=`echo $1 | cut -d '.' -f 1`
net=${net}.`echo $1 | cut -d '.' -f 2`
net=${net}.`echo $1 | cut -d '.' -f 3`

ips=`echo $1 | cut -d '.' -f 4`
ipb=`echo $2 | cut -d '.' -f 4`
if [ "$net" == "" ] || [ "$ipb" == "" ] || [ "$ips" == "" ];then 
	echo "The parmers wrong!"	
	exit 1
fi

i=1	
while [ $ips -le $ipb ]
do 
	ping -c 1 -w 0.5 -i 0.5  ${net}.${ips} &> /dev/null && result=0 || result=1
	if [ $result == 0 ];then
		str=`ping -c 1 -w 0.5 -i 0.5  ${net}.${ips} | awk 'NR==2 {print $7}'`
		printf "%3s : " $i
		echo "Server ${net}.${ips} is up " $str 
		i=$(( $i+1 ))
	fi
	ips=$(( $ips+1 ))
done

exit 0
```

测试的两个IP必须在同一网段内
