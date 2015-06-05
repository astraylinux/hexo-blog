title: 批量重命名
date: 2012-02-21 19:39:00
categories:
- Linux
tags:
- Bash
- Linux
---

命令用法： allrename 新的名称 新的后缀名 旧文件名1 旧文件名2 
文件一般是同类格式才会一起重命名，比如: allrename picture jpg *.jpg
<!--more-->

```bash
#!/bin/bash

#this script is use to rename many files together
#format :  allrename newfilename extendname filename1 filename2 .....
#warn : if you use it like this ,[ allrename * ] it will work ,but will have a bad consequence(result)

#build by ocean , 2011-11-11

if [ $# -le 2 ]
then 
	echo "wrong parms!"
	exit 111
fi

newname=$1
extend=$2
oldname=$(echo $3|cut -f 1 -d -)

if [ $newname = $oldname ]
then
	echo "old name is \"$oldname\""
	echo "new name is \"$newname\""
	echo "	name unchange!"
	exit 112
fi

shift
shift

echo $0

i=1

while [ $# -gt 0 ]
do	
	mv $1 $newname-$i.$extend
	printf "%-20s %-10s %-20s\n" $1  "---->"  $newname-$i.$extend
	i=$((i+1))
	shift
done
```

