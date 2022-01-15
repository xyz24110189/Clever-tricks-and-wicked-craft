network awareness in windows获取windows网络状态
===================================
获取windows网络状态有三种方式：


* InternetGetConnectedState(wininet.dll)直接获取


* GetAdaptersAddresses(iphlpapi.dll)枚举网络适配器


* Network Awareness

 + NLA in Windows XP

 + NLM in Vista or windows 7

## 源代码 source code##


[**netaware** - https://raw.githubusercontent.com/codepongo/utocode/master/windows/netaware.cpp](https://raw.githubusercontent.com/codepongo/utocode/master/windows/netaware.cpp)

* with iphlp - webkit的GetAdaptersAddresses方式的实现 webkit implementation
* with nlm  - NLM的同步实现 NLM synchronous implementation
* with nlm event  - NLM的异步实现 NLM asynchronous implementation
* with wininet - InternetGetConnectedState的实现 InternetGetConnectedState implementation
* with winsock - NLA的同步实现 NLA synchronous implementation
* wait for change with winsock - NLA的异步实现 NLA asynchronous implementation
* GetAdaptersAddresses方式的更准确实现可参考[firefox的源码](http://mxr.mozilla.org/mozilla1.9.2/source/netwerk/system/win32/nsNotifyAddrListener.cpp?raw=1)
 + 如果确定不了网络是否连接（如API失败等情况），则返回已连接状态
 + 通过获取AdapaterAddress AdapaterInfo和IPAddressTable进行综合判断
 + 通过NotifyAddrChange(http://msdn.microsoft.com/en-us/library/windows/desktop/aa366329.aspx)异步通知
伪代码如下：
<pre data-language="python">
def CheckAdapterAddress():
	addresses = GetAdaptersAddresses()
	for a in addresses:
		if a.OperStatus == IfOperStatusUp 
		and a.IfType != IF_TYPE_SOFTWARE_LOOPBACK
		and a.FirstUnicastAddress->Address.lpSockaddr != '192.168.0.1':
			return True;
	return False

def CheckAdaptersInfo():
	adapters = GetAdaptersInfo()
	for a in adapters:
		if a.DhcpEnabled:
			if a.DhcpServer.IpAddress == '255.255.255.255':
				return True;
			else:
				for ip in a.IpAddressList:
					if ip == '0.0.0.0':
						return True;
		return False;

def CheckIPAddrTable():
	tables = GetIpAddrTable()
	for t in tables:
		if t.dwAddr != 0 && t.dwAddr !=  0x0100007F:
			return True
	return False
</pre>
## reference 参考 ##
* [chromium source code](http://src.chromium.org/viewvc/chrome/trunk/src/net/base/network_change_notifier_win.cc?revision=264545)
* [Network Awareness](http://msdn.microsoft.com/en-us/library/ms697501.aspx)
* [NLA in Windows XP](http://msdn.microsoft.com/en-us/library/ms700657.aspx)
* [NLM in windowx XP later](http://msdn.microsoft.com/en-us/library/ee264321.aspx)
* [GetAdaptersAddresses in firefox](http://mxr.mozilla.org/mozilla1.9.2/source/netwerk/system/win32/nsNotifyAddrListener.cpp)
* [GetAdaptersAddresses in webkit](http://trac.webkit.org/browser/trunk/Source/WebCore/platform/network/win/NetworkStateNotifierWin.cpp?order=name)
* [How to automatic get Network change state](http://litao.me/post/2012-04-23-How-to-Get-Network-Change-automation.html)
* [source code in How to automatic get Network change state](https://gist.github.com/eahydra/3747803)
* [source code in How to automatic get Network change state](https://gist.github.com/eahydra/3747800)
* [GetAdaptersAddresses](http://msdn.microsoft.com/en-us/library/windows/desktop/aa365915.aspx)


