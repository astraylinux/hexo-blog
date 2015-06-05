title: 统计文件行数的shell
date: 2012-02-21 19:37:00
categories:
- Linux
tags:
- Bash
- tools
---

Program: Count every cpp file and h file is line in a path ,and add them ,output it
History: Build by AstrayLinux in 2011/10/17
<!--more-->

```bash
#!/bin/bash


if [ $# -ge 1 ] ; then 	
	i=0;
	s=0;
	t=0;
	if [ -d $1 ] ; then
		path=$1
		shift
	else
		path="./"
	fi
		 
	while [ "$i" != "$#" ]
	do
		str="\"*.$1\":  " 
		echo "======================= *.$1 ========================="
		find $path -type f -name "*.$1" -exec wc -l {} \;
		t=`find $path -type f -name "*.$1" -exec cat {} \; |wc -l`
		echo "====================== *.$1 SUM ======================"
		printf "%11s %10d\n" $str $t
		s=$(( $s+$t ))
		shift
	done
		
	echo "===================== ALL SUM ======================="
	printf "%11s %10d\n" "The all:" $s
else
	echo "" 
	echo "countSrcLine [path] [filetype ...]"
	echo "	If path is empty ,it will be ./"
	echo "	Filetype is a postfix like cpp,log. etc."
	echo "	EXAMPLE: "
	echo  "		countSrcLine ~/workspace/ cpp h"	
	echo ""
fi

exit 0
```

结果如下
![](http://hi.csdn.net/attachment/201202/21/0_1329824384Lwv4.gif)
