---
layout: post
title:  "[SuperSocket으로 서버 만들기] 1-2.에코서버를 만들어보자"
subtitle:   "1-2.서버의 기본 틀 만들기"
categories: develop
tags: csharp
---

## 서버의 기본 틀 만들기

일단 프로젝트부터 만들어보자.  
필자는 .net framework 4.5.2 를 이용하여 프로젝트를 만들었다.

다음으로는 SuperSocket를 설치 하여보자.
필자는 Nuget을 이용하여 다운로드 하였다.

Nuget 이용방법은 [해당글](https://docs.microsoft.com/ko-kr/nuget/quickstart/install-and-use-a-package-in-visual-studio)을 참고하면된다.

설치가 끝났으면 기본 틀을 잡아보자.

```csharp
//ReceiveFilter
public class MyReceiveFilter : FixedHeaderReceiveFilter<BinaryRequestInfo>
{
        public MyReceiveFilter()
        : base(6) // 해더의 길이
    {

    }
}

//AppServer
public class MyAppServer : AppServer<MyAppSession, BinaryRequestInfo>
{
    public MyAppServer() : base(new DefaultReceiveFilterFactory<MyReceiveFilter, BinaryRequestInfo>())
    {

    }
}

//AppSession
public class MyAppSession : AppSession<MyAppSession, BinaryRequestInfo>
{

}
```

위 코드가 SuperSocket의 기본적인 구조이다.  
한번 코드를 살펴보자
보면 1-1강에서 말 한것과 다른 점이 있다.  

1-1 강에서 말한 4가지의 기초 구조중 3가지만 구현되어 있다.   
이가 가능한 이유는 SuperSocket은 기본적으로 몇가지 종류의 ReceiveFilter와 RequestInfo를 제공하고 있다.

제공하는 RequestInfo과 ReceiveFilter 는 [여기](http://docs.supersocket.net/v1-6/en-US/The-Built-in-Common-Format-Protocol-Implementation-Templates)서 볼 수 있다.

필자는 RequestInfo으로는 BinaryRequestInfo 을 사용하였고
ReceiveFilter는 FixedHeaderReceiveFilter를 약간 수정하여 사용할것이다.

그러면 ReceiveFilter를 만들어보자.

```csharp
//MyReceiveFilter
public class MyReceiveFilter : FixedHeaderReceiveFilter<BinaryRequestInfo>
{
    public MyReceiveFilter()
        : base(6)
    {

    }

    protected override int GetBodyLengthFromHeader(byte[] header, int offset, int length)
    {
        return (int)header[offset + 4] + (int)header[offset + 5] * 256 - 6;
    }

    protected override BinaryRequestInfo ResolveRequestInfo(ArraySegment<byte> header, byte[] bodyBuffer, int offset, int length)
    {
        var byteTmp = bodyBuffer.CloneRange(offset, length);

        return new BinaryRequestInfo(BitConverter.ToUInt32(header.Array, 0).ToString(), MyReceiveFilter.Combine(header.Array, byteTmp));
    }

    public static byte[] Combine(byte[] first, byte[] second)
    {
        byte[] ret = new byte[first.Length + second.Length];
        Buffer.BlockCopy(first, 0, ret, 0, first.Length);
        Buffer.BlockCopy(second, 0, ret, first.Length, second.Length);
        return ret;
    }
}
```

코드를 설명해보자면 

GetBodyLengthFromHeader의 return 에서 6을 빼는것을 볼 수 있다.  
SuperSocket은 기본적으로 길이를 확인할때 전채에서 해더의 길이를 뺀 값을 넘겨줘야 한다.  
그래서 6을 빼고 return 한 것이다. 

나머지 부분은 받은 byte[]를 나중에 echo 할때 편하게 하기 위하여 header와 body를 연결했다.  

다음으로 넘어가서 ```MyAppServer``` 의 내부 코드를 작성하겠다.

```csharp
//MyAppServer
public class MyAppServer : AppServer<MyAppSession, BinaryRequestInfo>
{
    public MyAppServer() : base(new DefaultReceiveFilterFactory<MyReceiveFilter, BinaryRequestInfo>())
    {
        this.NewSessionConnected += new SessionHandler<MyAppSession>(MyServer_NewSessionConnected);
        this.SessionClosed += new SessionHandler<MyAppSession, CloseReason>(MyServer_SessionClosed);
        this.NewRequestReceived += new RequestHandler<MyAppSession, BinaryRequestInfo>(MyServer_NewRequestReceived);
    }

    private void MyServer_NewRequestReceived(MyAppSession session, BinaryRequestInfo requestinfo)
    {
        Console.WriteLine("MyServer_NewRequestReceived");

        session.Send(requestinfo.Body,0, requestinfo.Body.Length);
    }

    private void MyServer_SessionClosed(MyAppSession session, CloseReason value)
    {
        Console.WriteLine("MyServer_UserClosed");
    }

    private void MyServer_NewSessionConnected(MyAppSession session)
    {
        Console.WriteLine("MyServer_NewUserConnected");
    }
}

```

일단 생성자에서 이벤트를 연결해준다.  
코딩하는 사람에 따라서 MyAppServer 를 만드는 부분 에서 연결하는 경우도 있다.  

이벤트에 대하여 설명하자면  
```MyServer_NewUserConnected``` 은 Session의 접속시  
```MyServer_SessionClosed``` 은 Session의 접속종료  
```MyServer_NewRequestReceived``` 는 메시지를 받았을때 호출된다.

서버는 echo 서버 이므로 메시지를 받았을때 ```requestinfo.Body``를 전송하는 형식으로 구현하였다.  

서버의 구현은 끝났으니까 서버를 돌려보자.

```csharp
class Program
{
    static void Main(string[] args)
    {
        MyAppServer server = new MyAppServer();

        server.Setup(new RootConfig(),new ServerConfig()
        {
            Port = 3000,
            Ip = "Any",
        });

        server.Start();

        while (Console.ReadLine() != "q")
        {

        }
    }
}
```

Main 부분은 서버를 만들고 ```ServerConfig``` 을 이용하여 포트를 지정한다.  
```ServerConfig``` 은 포트 말고도 다양한 것을 지정할 수 있다.  
전송 버퍼크기, 서버이름, 전송큐 사이즈 등등 을 지정 가능하다.  

서버를 만들었으면 한번 클라이언트도 만들어보자.

클라이언트는 SuperSocket.ClientEngine.Core, [CGDbuffer](https://github.com/CGLabs/)의 c#버전을 이용하여 만들 것 이다.  

SuperSocket.ClientEngine.Core은 Nuget에서 받을 수 있다.

```csharp
private static AsyncTcpSession tcpSession;
static void Main(string[] args)
{
    tcpSession = new AsyncTcpSession();

    tcpSession.Connected += tcpSession_Connected;
    tcpSession.Closed += tcpSession_Closed;
    tcpSession.DataReceived += tcpSession_DataReceived;
    tcpSession.Error += tcpSession_Error;
    tcpSession.Connect(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 3000));
    var tmpData = "";
    while (tmpData != "q")
    {
        tmpData = Console.ReadLine();

        var buffer = new CGD.buffer(50);

        buffer.append<int>(123);
        buffer.append<short>(0);
        buffer.append<string>(tmpData);
        buffer.set_front<short>(buffer.Count, 4);

        tcpSession.Send(buffer);
    }
}
```

AsyncTcpSession을 이용하여 서버와 통신하며.  
메시지를 키보드로 입력하여 전송 가능 하도록 만들었다.  

그러면 이번에는 각종 이벤트를 선언해보자.  

```csharp
private static void tcpSession_Error(object sender, ErrorEventArgs e)
{
    Console.WriteLine("tcpSession_Error");
}

private static void tcpSession_DataReceived(object sender, DataEventArgs e)
{
    AsyncTcpSession session = sender as AsyncTcpSession;

    byte[] tmpBuffer = e.Data;
    var buffer = new CGD.buffer(e.Data, 0, e.Length);

    int bufferType = (int)buffer.extract_uint();
    ushort bufferLength  = (ushort)buffer.extract_short();
    string bufferData = buffer.extract_string();
        
    Console.WriteLine("tcpSession_DataReceived");
    Console.WriteLine("---------------------------------");
    Console.WriteLine("Type   : " + bufferType);
    Console.WriteLine("Length : " + bufferLength);
    Console.WriteLine("Data   : " + bufferData);
    Console.WriteLine("---------------------------------");
}

private static void tcpSession_Closed(object sender, EventArgs e)
{
    Console.WriteLine("tcpSession_Closed");
}

private static void tcpSession_Connected(object sender, EventArgs e)
{
    AsyncTcpSession session = sender as AsyncTcpSession;

    Console.WriteLine("tcpSession_Connected");

    var buffer = new CGD.buffer(50);

    buffer.append<int>(123);
    buffer.append<short>(0);
    buffer.append<string>("전송한 데이터");
    buffer.set_front<short>(buffer.Count, 4);

    session.Send(buffer);
}
```

tcpSession_Connected 에서는 서버와의 연결시 데이터를 전송한다.  
tcpSession_DataReceived 는 서버에서 전송한 데이터를 받는다.  
tcpSession_Closed 는 서버에서 연결을 종료하면 호출된다.
tcpSession_Error 는 소켓에서 오류가 발생하였을 시 호출된다.

이러면 클라이언트도 완성이다!

이런식으로 서버와 통신을 하는 것을 볼 수 있다.
![결과이미지](/assets/img/csharp/SuperSocket/EchoTest.PNG)

다음 강과는 채팅서버를 만들어 볼 것이다.

본 강좌의 [소스코드](https://github.com/Hot-key/SuperSocketServer)는 깃허브에서 확인 가능하다.

