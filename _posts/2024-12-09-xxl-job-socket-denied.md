---
layout: post
title: XXL-JOB > Socket Exception - Permission denied
---

# env

jdk17, spring-boot 3.2.x, xxl-job-core 2.4.1

# story

admin 启动后 executor 正常注册，但是执行任务的指令传不到 executor。

admin 报错信息如下：

```shell
ERROR c.x.job.core.util.XxlJobRemotingUtil - Permission denied: getsockopt
java.net.SocketException: Permission denied: getsockopt
    at java.base/sun.nio.ch.Net.pollConnect(Native Method)
    at java.base/sun.nio.ch.Net.pollConnectNow(Net.java:672)
    at java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)
    at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)
    at java.base/java.net.Socket.connect(Socket.java:633)
    at java.base/sun.net.NetworkClient.doConnect(NetworkClient.java:178)
    at java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:533)
    at java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:638)
    at java.base/sun.net.www.http.HttpClient.<init>(HttpClient.java:281)
    at java.base/sun.net.www.http.HttpClient.New(HttpClient.java:386)
    at java.base/sun.net.www.http.HttpClient.New(HttpClient.java:408)
    at java.base/sun.net.www.protocol.http.HttpURLConnection.getNewHttpClient(HttpURLConnection.java:1309)
    at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1242)
    at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1128)
    at java.base/sun.net.www.protocol.http.HttpURLConnection.connect(HttpURLConnection.java:1057)
    at com.xxl.job.core.util.XxlJobRemotingUtil.postBody(XxlJobRemotingUtil.java:99)
    at com.xxl.job.core.biz.client.ExecutorBizClient.run(ExecutorBizClient.java:43)
    at com.xxl.job.admin.core.trigger.XxlJobTrigger.runExecutor(XxlJobTrigger.java:211)
    at com.xxl.job.admin.core.trigger.XxlJobTrigger.processTrigger(XxlJobTrigger.java:164)
    at com.xxl.job.admin.core.trigger.XxlJobTrigger.trigger(XxlJobTrigger.java:89)
```

排查后发现是因为开了 VPN，关闭 VPN 后调度正常。

# Native method

查看 [openjdk - Net.c](https://github.com/openjdk/jdk/blob/master/src/java.base/windows/native/libnio/ch/Net.c)

```c
JNIEXPORT jboolean JNICALL
Java_sun_nio_ch_Net_pollConnect(JNIEnv* env, jclass this, jobject fdo, jlong timeout)
{
    int optError = 0;
    int result;
    int n = sizeof(int);
    jint fd = fdval(env, fdo);
    fd_set wr, ex;
    struct timeval t;

    FD_ZERO(&wr);
    FD_ZERO(&ex);
    FD_SET((u_int)fd, &wr);
    FD_SET((u_int)fd, &ex);

    if (timeout >= 0) {
        t.tv_sec = (long)(timeout / 1000);
        t.tv_usec = (timeout % 1000) * 1000;
    }

    result = select(fd+1, 0, &wr, &ex, (timeout >= 0) ? &t : NULL);

    if (result == SOCKET_ERROR) {
        NET_ThrowNew(env, WSAGetLastError(), "select");
        return JNI_FALSE;
    } else if (result == 0) {
        return JNI_FALSE;
    } else {
        // connection established if writable and no error to check
        if (FD_ISSET(fd, &wr) && !FD_ISSET(fd, &ex)) {
            return JNI_TRUE;
        }
        result = getsockopt((SOCKET)fd,
                            SOL_SOCKET,
                            SO_ERROR,
                            (char *)&optError,
                            &n);
        if (result == SOCKET_ERROR) {
            int lastError = WSAGetLastError();
            if (lastError != WSAEINPROGRESS) {
                NET_ThrowNew(env, lastError, "getsockopt");
            }
        } else if (optError != NO_ERROR) {
            NET_ThrowNew(env, optError, "getsockopt");
        }
        return JNI_FALSE;
    }
}
```

注意到 getsockopt 在头文件 winsock2.h 中找到该方法。

https://learn.microsoft.com/en-us/windows/win32/api/winsock2/

注意到 NET_ThrowNew 在头文件 net_util.h 中找到该方法。

https://github.com/openjdk/jdk/blob/master/src/java.base/share/native/libnet/net_util.h

https://github.com/openjdk/jdk/blob/master/src/java.base/windows/native/libnet/net_util_md.c

```c
JNIEXPORT void JNICALL
NET_ThrowNew(JNIEnv *env, int errorNum, char *msg)
{
    int i;
    int table_size = sizeof(winsock_errors) /
                     sizeof(winsock_errors[0]);
    // ...
}                     
```

注意到 winsock_errors 在 net_util_md.c 中找到该常量。

```c
static struct {
    int errCode;
    const char *exc;
    const char *errString;
} const winsock_errors[] = {
    { WSAEACCES,                0,      "Permission denied" },
    // ...
```

到这里可以确认是 getsockopt 抛出的异常，往回查 winsock2.h#getsockopt，一无所获。

https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/um/WinSock2.h

```c
#if INCL_WINSOCK_API_PROTOTYPES
WINSOCK_API_LINKAGE
int
WSAAPI
getsockopt(
    _In_ SOCKET s,
    _In_ int level,
    _In_ int optname,
    _Out_writes_bytes_(*optlen) char FAR * optval,
    _Inout_ int FAR * optlen
    );
#endif /* INCL_WINSOCK_API_PROTOTYPES */
```

# conclusion

socket 建立失败，根本原因不明，推测是 VPN 转发了全部流量导致。

admin 127.0.0.1

executor 识别为本机的内网地址 10.200.0.10

VPN 开启后，ping 10.200.0.10 是不通的，返回“一般故障”，关闭后可以 ping 通。

地址是 IPv4 格式，VPN IPv4 的接口跳跃点是 1，其他适配器是自动跳跃点，但是变更这部分配置还是不通。

感觉上可以给 10.200.0.10 配个专门的 route，，，没试成功。

# ref

https://support.veryant.com/phpkb/article.php?id=197#:~:text=SocketException%3A%20Permission%20denied%3A%20connect%22%20exception%20message%20may%20happen%20when,in%20Java%207%20and%20above.

https://blog.csdn.net/huangzhen1991/article/details/81460791

https://freyagrace.cn/index.php/archives/296/

https://www.cnblogs.com/vincentmu/p/15641757.html

https://zhuanlan.zhihu.com/p/71536075

