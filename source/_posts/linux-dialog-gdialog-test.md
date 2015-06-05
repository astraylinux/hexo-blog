title: dialog及gdialog命令测试
date: 2011-12-04 20:32:00
categories:
- Linux
tags:
- Bash
- test
---

简单明了的列表，copy也方便。
```bash
#!/bin/bash
#test the gDialog
height=24
width=80
text="text"
filename="/home/ocean/.bashrc"

gdialog --title "testbox" --textbox "$filename" $(($height*4)) $width 
gdialog --title "checklist" --checklist "$text" $height $width   2   "1" "aaaa" "on"   "2" "bbbb" "on" # list_height [tag text status]
gdialog --title "infobox" --infobox "$text =========" $height $width
gdialog --title "inputbox" --inputbox  "$text" $height $width "initial string" 
gdialog --title "menu" --menu "$text" $height $width 2 "1" "aaaa" "2" "bbbb"    #menu_height [tag item]
gdialog --title "msgbox" --msgbox "$text========" $height $width 
gdialog --title "radiolist" --radiolist "$text" $height $width 2 "1" "aaaa" "on" "2" "bbbb" "off"
gdialog --title "yesno" --yesno "$text" $height $widch
```

![](http://hi.csdn.net/attachment/201112/4/0_1323002276i5LM.gif)
