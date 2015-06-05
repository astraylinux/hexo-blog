title: 遇到多线程问题，关于MyDownload
date: 2012-12-10 18:51:00
categories:
- program
tags:
- program
- problem
- VC 
---

最近在的安卓驱动安装程序要用到Windows下的C++下载程序。在网上找到了一个下载类MyDownload，MyDownload里面有多线程下载的部分，一开始开三个线程，偶尔会出现崩溃，后面加大线程很快就崩溃，明显是线程处理有问题。下面是下载线程
<!--more--->

```cpp
UINT CHttpGet::ThreadDownLoad(void* pParam)
{
	CHttpSect *pInfo=(CHttpSect*)pParam;
	SOCKET hSocket;

	if(pInfo->bProxyMode){	
		hSocket=ConnectHttpProxy(pInfo->szProxyAddr,pInfo->nProxyPort);
	}
	else{
		hSocket=ConnectHttpNonProxy(pInfo->szHostAddr,pInfo->nHostPort);
	}
	if(hSocket == INVALID_SOCKET) return 1;


	// 计算临时文件大小，为了断点续传
	DWORD nFileSize=myfile.GetFileSizeByName(pInfo->szDesFilename);
	DWORD nSectSize=(pInfo->nEnd)-(pInfo->nStart);

	rdownloaded+=nFileSize;

	// 此段已下载完毕.
	if(nFileSize==nSectSize){
		//mj
		printf("文件下载成功！下载结束！\n");                //这里可以设置写信息
		//mj

		TRACE("文件已下载完毕!\n");                                     
		CHttpGet::m_nCount++;  // 计数.
		return 0;
	}

    FILE *fpwrite=myfile.GetFilePointer(pInfo->szDesFilename);
	if(!fpwrite) return 1;

    // 设置下载范围.
	SendHttpHeader(hSocket,pInfo->szHostAddr,pInfo->szHttpAddr,
		      pInfo->szHttpFilename,pInfo->nStart+nFileSize);
	
	// 设置文件写指针起始位置，断点续传
	fseek(fpwrite,nFileSize,SEEK_SET);

	DWORD nLen; 
	DWORD nSumLen=0; 
	char szBuffer[1024];

	while(1)
	{
		if(nSumLen>=nSectSize-nFileSize) break;
		nLen=recv(hSocket,szBuffer,sizeof(szBuffer),0);
		
		//原子操作，不用同步。
		rdownloaded += nLen;
		
		if (nLen == SOCKET_ERROR){
			TRACE("Read error!\n");
			fclose(fpwrite);
			return 1;
		}

  		if(nLen==0) break;
		nSumLen +=nLen;
		TRACE("%d\n",nLen);

		// 把数据写入文件.		
		fwrite(szBuffer,nLen,1,fpwrite);
	}

	fclose(fpwrite);      // 关闭写文件.
	closesocket(hSocket); // 关闭套接字.
	CHttpGet::m_nCount++; // 计数.
	return 0;
}
```

这段代码看来没什么问题，线程开到50的时候崩溃问题就很明显了。跟了几次都是CString 释放出错，往回找发现是在SendHttpHeader里有问题，代码：


```cpp
BOOL CHttpGet::SendHttpHeader(SOCKET hSocket,CString strHostAddr,CString strHttpAddr,CString strHttpFilename,DWORD nPos)
{
	// 进行下载. 
	static CString sTemp;
	char cTmpBuffer[1024];

	// Line1: 请求的路径,版本.
	sTemp.Format(L"GET %s%s HTTP/1.1\r\n",strHttpAddr,strHttpFilename);
	if(!SocketSend(hSocket,sTemp)) return FALSE;

	// Line2:主机.
	sTemp.Format(L"Host: %s\r\n",strHostAddr);
	if(!SocketSend(hSocket,sTemp)) return FALSE;

	// Line3:接收的数据类型.
	sTemp.Format(L"Accept: \r\n");
	if(!SocketSend(hSocket,sTemp)) return FALSE;
	
	// Line4:参考地址.
    sTemp.Format(L"Referer: %s\r\n",strHttpAddr); 
	if(!SocketSend(hSocket,sTemp)) return FALSE;
		
	// Line5:浏览器类型.
	sTemp.Format(L"User-Agent: Mozilla/4.0 \
		(compatible; MSIE 5.0; Windows NT; DigExt; DTS Agent;)\r\n");

	if(!SocketSend(hSocket,sTemp)) return FALSE;

	// 续传. Range 是要下载的数据范围，对续传很重要.
	sTemp.Format(L"Range: bytes=%d-\r\n",nPos);
	if(!SocketSend(hSocket,sTemp)) return FALSE;
	
	// LastLine: 空行.
	sTemp.Format(L"\r\n");
	if(!SocketSend(hSocket,sTemp)) return FALSE;

	// 取得http头.
	int i=GetHttpHeader(hSocket,cTmpBuffer);
	if(!i)
	{
		TRACE(L"获取HTTP头出错!\n");
		return 0;
	}
	
	// 如果取得的http头含有404等字样，则表示连接出问题.
	sTemp=cTmpBuffer;
	if(sTemp.Find(L"404")!=-1) return FALSE;

	// 得到待下载文件的大小.
	m_nFileLength=GetFileLength(cTmpBuffer);

	// 因为TRACE()函数最大的字符串长度为255.
    //TRACE(CString(cTmpBuffer).GetBuffer(200));
	
	return TRUE;
}
```

看了许久才发开头那个sTemp是static，有点不解了，作者为什么要在这里加一个static，为了效率？
总之，去掉这个static崩溃问题就没再出现了。写多线程的时候不能只看函数有没有同步问题，还要看看所调用到的函数有没有调用到全局或static变量，有的话还是保护起来，不然一不注意，出现这种问题，实在不容易跟。


