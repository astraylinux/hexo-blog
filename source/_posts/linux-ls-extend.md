title: 将文件和文件夹分开的ls
date: 2011-11-28 10:21:00
categories:
- Linux
tags:
- Linux
- Bash
- C++
---

以前常常在找让ls单独列出文件或文件夹的方法，基本上都是通过一些shell处理，不过得到的结果一般都没有格式。学linux编程，想想自己写一个好了。
可执行文件下载地址：[点击打开链接](http://download.csdn.net/detail/astraylinux/3851414)
用C++组织的，自己觉得组织的不好，应该有很多改进的地方，贴出来求建议和批评!
不知道是计算的方法不好，还是C++效率比较低，程序的效率比默认ls要差不少。有空用Ｃ实现看看。
总共三个类，
ListFile　主要功能都在这
Util　　　一些辅助函数，大部分计算的函数
FileNode 文件节点信息

<!--more-->

```cpp
//================================ FileNode ==================================
/*
 * FileNode.h
 *
 *  Created on: 2011-11-18
 *      Author: Astray
 */

#ifndef FILENODE_H_
#define FILENODE_H_
#include <iostream>
#include <sys/stat.h>

using namespace std;
class FileNode
{
public:
    FileNode(string n);
    FileNode();
    virtual ~FileNode();

    struct stat statbuf;
    string sName;
    int iType;
};
```

```cpp
#endif /* FILENODE_H_ */

/*
 * FileNode.cpp
 *
 *  Created on: 2011-11-18
 *      Author: Astray
 */

#include "FileNode.h"

FileNode::FileNode(string n)
{
    sName = n;
}

FileNode::FileNode()
{

}

FileNode::~FileNode()
{
}
```

```cpp
//===============================　 Util 　================================
#ifndef PRINTF_TYPEFL_H
#define PRINTF_TYPEFL_H

#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <vector>
#include <list>
#include "FileNode.h"
//Color ForeGround
#define P_BLACK 	"\e[30m"
#define P_RED 		"\e[31m"
#define P_GREEN 	"\e[32m"
#define P_YELLOW 	"\e[33m"
#define P_BLUE 		"\e[34m"
#define P_PURPURE 	"\e[35m"
#define P_DGREEN 	"\e[36m"
#define P_WHITE 	"\e[37m"

//Color BackGround
#define G_BLACK 	"\e[40m"
#define G_DRED 		"\e[41m"
#define G_GREEN 	"\e[42m"
#define G_YELLOW 	"\e[43m"
#define G_BLUE	 	"\e[44m"
#define G_PURPURE 	"\e[45m"
#define G_DGREEN 	"\e[46m"
#define G_WHITE 	"\e[47m"

//Figure
#define F_CLOSE		"\e[0m"
#define F_HIGHLIGHT "\e[1m"
#define F_UNDERLINE "\e[4m"
#define F_GLINT		"\e[5m"
#define F_CONTRARY  "\e[7m"
#define F_BLANKOFF 	"\e[8m"

//Operate
#define O_CLEAR		"\e[2J"
#define O_CLTOEND	"\e[K"
#define O_SEVE		"\e[s"
#define O_RECOVE	"\e[u"
#define O_HIDECURSOR	"\e[?25l"
#define O_SHOWCURSOR	"\e[?25h"

//Utility Function
class Util
{
public:
	//get the length each column
	static int getmax(std::vector<int>&, int, int, int);

	//get the output width,sum must smaller than screen width
	static int getsum(std::vector<int>&, int, int, std::vector<int>&);
	static int setFileStat(const char*,FileNode&);

	//insert files into the list ,sort by ascii
	static int insertFile(std::list<FileNode>&,FileNode);
	static void setColor(list<FileNode>::iterator);

	//if there are multi-directories arguments ,print each directory name
	static void printDirLine(string ,char*);
	static void printHelp();
};
#endif
```

```cpp
#include "Util.h"
#include "ListFile.h"

int Util::getmax(vector<int>& array, int length, int gap, int col)
{
	int max = -1;
	for (int i = col; i < length; i += gap)
	{
		if (array[i] > max)
			max = array[i];
	}
	return max;
}

int Util::getsum(vector<int>& array, int length, int gap, vector<int>& out)
{
	int sum = 0;
	for (int i = 0; i < gap; i++)
	{
		out.push_back((getmax(array, length, gap, i) + 2));
		sum += out[i];
	}
	return sum;
}

int Util::setFileStat(const char *name, FileNode& fn)
{
	mode_t modes;
	lstat(name, &(fn.statbuf));
	modes = fn.statbuf.st_mode;
	//cout<<fn.statbuf.st_mode<<endl;
	if (S_ISREG(modes))
		fn.iType = S_IFREG;
	if (S_ISDIR(modes))
		fn.iType = S_IFDIR;
	if (S_ISLNK(modes))
		fn.iType = S_IFLNK;
	if (S_ISBLK(modes))
		fn.iType = S_IFBLK;
	if (S_ISCHR(modes))
		fn.iType = S_IFCHR;
	if (S_ISFIFO(modes))
		fn.iType = S_IFIFO;
	return 1;
}

int Util::insertFile(list<FileNode>& lf, FileNode fn)
{
	list<FileNode>::iterator iter = lf.begin();
	list<FileNode>::iterator end = lf.end();
	if (iter == end)
	{
		lf .push_back(fn);
		return 0;
	}

	if (fn.sName > (--end)->sName)
	{
		lf.push_back(fn);
		return 0;
	}

	end++;
	for (; iter != end; iter++)
	{
		if (fn.sName == iter->sName)
			break;
		if (fn.sName < iter->sName)
		{
			lf.insert(iter, fn);
			return 0;
		}
	}
	return 0;
}

void Util::setColor(list<FileNode>::iterator iter)
{
	switch (iter->iType)
	{
	case S_IFREG:
		if (((iter->statbuf.st_mode) & S_IRWXU) == 448)
			cout << P_GREEN << F_HIGHLIGHT;
		break;
	case S_IFDIR:
		cout << P_BLUE << F_HIGHLIGHT;
		break;
	case S_IFBLK:
	case S_IFCHR:
	case S_IFIFO:
		cout << P_YELLOW << F_HIGHLIGHT;
		break;
	case S_IFLNK:
		cout << P_DGREEN << F_HIGHLIGHT;
		break;
	}
}

void Util::printDirLine(string dir,char *color)
{
	cout<<color<<F_HIGHLIGHT;
	cout<<dir<<":";
	cout<<F_CLOSE<<endl;
}

void Util::printHelp()
{
	cout << "lf -[adfch] directory1 directory2 ..." << endl;
	cout<<" -a 	--all 		show all of the files include begin with \'.\'"<<endl;
	cout<<" -d 	--dir 		show the directories only."<<endl;
	cout<<" -f 	--file 		show the files except directories."<<endl;
	cout<<" -c 	--count 	count there are how much files and directories."<<endl;
	cout<<" -h 	--help 		show this."<<endl;
}

```

```cpp
//================================== ListFile ========================================
/*
 * ListFile.h
 *
 *  Created on: 2011-11-14
 *      Author: Astray
 */

#ifndef FILELIST_H_
#define FILELIST_H_
//========= C/C++ standard ============
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <iostream>
#include <vector>
#include <list>
//========= Linux header ==============
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/ioctl.h>
#include <dirent.h>
#include <unistd.h>
#include <limits.h>
#include <termios.h>
//========== Project header ==========
#include "FileNode.h"
//====================================================
#define DEFAULT		0
#define SHOWALL 	1
#define SHOWFILE 	2
#define SHOWDIR 	3
using namespace std;

class ListFile
{
public:
	//reset，used when there are multi-directories arguments
	void reset(const char* newdir);

	void setOption(int option);

	//use count function
	void countOn();

	//load directory informations
	int loadDir();

	void printDir();

	void printFile();

	void print();

	//print file list or directories list
	int print(list<FileNode>::iterator begin, list<FileNode>::iterator end,
			int size);

	ListFile(char* dir);

	ListFile();

	virtual ~ListFile();

	string sColor;
private:
	string sDirnow;
	list<FileNode> lDir, lFile;

	int iOption;
	bool bShowCount;
};

#endif /* FILELIST_H_ */
```

```cpp
/*
 * ListFile.cpp
 *
 *  Created on: 2011-11-14
 *      Author: Astray
 */

#include "ListFile.h"
#include "Util.h"
#include <myassist.h>

ListFile::ListFile(char* dir)
{
    iOption = DEFAULT;
    sDirnow = dir;
    sColor = "";
    bShowCount = false;
}

ListFile::ListFile()
{
    iOption = DEFAULT;
    sDirnow = ".";
    sColor = "";
    bShowCount = false;

}

void ListFile::reset(const char *dir)
{
    sDirnow = dir;
    lDir.clear();
    lFile.clear();
}

ListFile::~ListFile()
{
}

void ListFile::setOption(int opt)
{
    iOption = opt;
}

void ListFile::countOn()
{
    bShowCount = true;
}

void ListFile::printDir()
{
    if (lDir.size() == 0)
        return;
    print(lDir.begin(), lDir.end(), lDir.size());
}

void ListFile::printFile()
{
    if (lFile.size() == 0)
        return;
    print(lFile.begin(), lFile.end(), lFile.size());
}

void ListFile::print()
{
    switch (iOption)
    {
    case SHOWDIR:
        printDir();
        break;
    case SHOWFILE:
        printFile();
        break;
    default:
        printDir();
        printFile();
    }

    if (bShowCount)
    {
        cout << sColor << F_HIGHLIGHT << "Directories: " << lDir.size()
        <<"\t\t"<<"Files: " << lFile.size() << F_CLOSE
                << endl;
    }
}


int ListFile::loadDir()
{
    DIR* dPath;
    if (!(dPath = opendir(sDirnow.c_str())))
    {
        cerr << "Can't open the data of " << sDirnow << endl;
        exit(1);
    }
    else
    {
        struct dirent* dEntry;
        string sPath, sName;
        FileNode fNewFile;
        sPath = sDirnow;
        sPath += "/";

        while ((dEntry = readdir(dPath)) != NULL)
        {
            sName = sPath;
            sName += dEntry->d_name;
            Util::setFileStat(sName.c_str(), fNewFile);
            fNewFile.sName = dEntry->d_name;

            if (fNewFile.iType == S_IFDIR)
            {
                if ((iOption == SHOWFILE) || (iOption != SHOWALL
                        && fNewFile.sName.at(0) == '.'))
                    continue;

                Util::insertFile(lDir, fNewFile);
            }
            else
            {
                if ((iOption == SHOWDIR) || (iOption != SHOWALL
                        && fNewFile.sName.at(0) == '.'))
                    continue;
                Util::insertFile(lFile, fNewFile);
            }
        }
        closedir(dPath);
    }
    return 0;
}

int ListFile::print(list<FileNode>::iterator begin,
        list<FileNode>::iterator end, int len)
{
    struct winsize size;
    ioctl(STDIN_FILENO, TIOCGWINSZ, &size);
    int col = size.ws_col, rawwide = 6;
    if (col == 0)
        col = 90;
    int gap = col / rawwide;

    //============= count the wide of dir_name =============
    vector<int> vWide;
    list<FileNode>::iterator iter = begin;

    for (; iter != end; iter++)
    {
        vWide.push_back(iter->sName.length());
    }

    //================ adapt the gap ========================
    vector<int> vWideOfCol;
    int sum;

    do
    {
        vWideOfCol.clear();
        gap = col / rawwide;
        sum = Util::getsum(vWide, len, gap, vWideOfCol);
        rawwide += 1;
        if (len < gap && sum < col)
        {
            gap = len;
            break;
        }
    } while (sum >= col);

    //======== set the format for each column =============
    vector<string> vsOutLen, vsFormat;
    string sOut, sFor;
    char cOut[11];
    for (int i = 0; i < gap; i++)
    {
        sFor = "%-";
        itoa(vWideOfCol[i], cOut);
        sFor += cOut;
        sFor += "s";
        vsFormat.push_back(sFor);
        //        cout << sFor << endl;
    }

    //================== print =======================
    iter = begin;
    int i = 0;
    for (; iter != end; iter++)
    {
        Util::setColor(iter);
        printf(vsFormat[i % gap].c_str(), iter->sName.c_str());
        if ((i % gap) == (gap - 1))
            cout << endl;
        i++;
        cout << F_CLOSE;
    }
    if (i % gap != 0)
        cout << endl;

    return i;
}

```

效果图：
xiaoguotu.jpg![xiaoguotu.jpg](http://hi.csdn.net/attachment/201111/28/0_1322447019Vf46.gif)

