title: 在终端for循环
date: 2012-02-21 19:23:00
categories:
- Linux
tags:
- Bash
- Linux
---

```bash
for (( i=30;i<37;i++ ))
do
     for (( j=40;j<47;j++))
      do
             printf "\e[$((i))m\e[$((j))m\e[1m%s\e[0m\n" "String!"　//与Ｃ的printf一样
      done
done
```

在终端下写成一行，注意分号位置，done前有一个，其他都可以不用
```bash
for (( i=30;i<37;i++ )) do for ((j=40;j<47;j++)) do printf"\e[$((i))m\e[$((j))m\e[1m%s\e[0m\n" "String!" ;done;done
```

