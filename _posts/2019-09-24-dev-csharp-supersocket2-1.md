---
layout: post
title:  "[SuperSocket으로 서버 만들기] 2-1.채팅서버를 만들어보자"
subtitle:   "2-1.서버 구조 구상하기"
categories: develop
tags: csharp
---

## 서버 구조 구상하기
먼저 채팅서버를 위한 구조를 구상해보자.

메시지를 구조를 구상하자.

필자는 버퍼관리를 위하여 [CGDbuffer(github))](https://github.com/CGLabs/CGDbuffer) 를 약간 병경하여 이용할 것이다.  
기능등은 동일하니 CGDbuffer 를 이용하여 실습이 가능 할 것이다.

필자가 사용할 패킷의 구조는 
4바이트의 길이와 2바이트의 타입 즉 6바이트의 해더를 가지고 해더 이후로는 메시지를 담고있다.  

![패킷 구조](/assets/img/dev/csharp/SuperSocket/ChatPacket.png)

한번 저 패킷 구조를 받을 수 있는 ReceiveFilter 를 만들기 전에 이번에는 RequestInfo를 만들어보자.

```RequestInfo<T>``` 를 상속받는 구조로 만들면 된다.

```csharp
public class MyRequestInfo : RequestInfo<MyRequestInfo>
{
    public new dynamic Key { get; private set; }

    public new CGD.NcsBuffer Body { get; private set; }

    public MyRequestInfo(byte[] body, int length)
    {
        Body = new CGD.NcsBuffer(body, 0, body.Length);
        Key = Body.get_front_ushort(2);
    }
}
```

Key와 Body를 선언하고 초기화 시 버퍼를 넣어줄 수 있도록 한다.

다음은 ReceiveFilter를 만들어보자.

```csharp
public class MyReceiveFilter : FixedHeaderReceiveFilter<MyRequestInfo>
{
    public MyReceiveFilter() : base(6)
    {

    }

    protected override int GetBodyLengthFromHeader(byte[] header, int offset, int length)
    {
        return (int)header[offset] +
               (int)header[offset + 1] * 256 +
               (int)header[offset + 2] * 65535 +
               (int)header[offset + 3] * 16777216 - 6; 
    }

    protected override MyRequestInfo ResolveRequestInfo(ArraySegment<byte> header, byte[] bodyBuffer, int offset, int length)
    {
        var byteTmp = bodyBuffer.CloneRange(offset, length);

        return new MyRequestInfo(MyReceiveFilter.Combine(header.Array, byteTmp), length);
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

ReceiveFilter는 1-2 에서 설명한 것과 비슷한 구조 이다.  
다른점이 있다면 MyRequestInfo를 사용한다는것 정도이다.  


```csharp
public class MyAppServer : AppServer<MyAppSession, MyRequestInfo>
{
    public MyAppServer() : base(new DefaultReceiveFilterFactory<MyReceiveFilter, MyRequestInfo>())
    {
        this.NewSessionConnected += MyServer_NewSessionConnected;
        this.SessionClosed += MyServer_SessionClosed;
        this.NewRequestReceived += MyServer_NewRequestReceived;
    }

    private void MyServer_NewRequestReceived(MyAppSession session, MyRequestInfo requestInfo)
    {
    }

    private void MyServer_SessionClosed(MyAppSession session, CloseReason value)
    {
    }

    private void MyServer_NewSessionConnected(MyAppSession session)
    {
    }
}

public class MyAppSession : AppSession<MyAppSession, MyRequestInfo>
{

}
```

다른 부분도 이전 강좌와 다른 부분은 거의 없다.  


다음으로는 통신에 사용할 프로토콜을 만들어보자.  
간단하게 우리가 만들 서버에서 사용할 통신규약이라고 생각하면 된다.

필자는 Enum 을 이용하여 구현 하였다.  

```csharp
public enum ChatProtocol : ushort
{
    RoomCreate = 0x0001, // 방 만들기
    RoomRemove = 0x0002, // 방 지우기

    RoomConnection = 0x1001, // 방 연결시
    RoomDisconnection = 0x1002, // 방 접속종료

    RoomUserList = 0x2001, // 모든 유저 정보
    RoomUserCount = 0x2002, // 방의 유저 수
    RoomUserInfo = 0x2003, // 특정 유저의 정보

    UseRegister = 0x3001, // 회원가입
    UserLogin = 0x3002, // 로그인

    UserNameChange = 0x3011, // 이름변경

    RoomSendMessage = 0x4001, // 메시지 전송
    RoomRemoveMessage = 0x4002, // 메시지 지우기
    RoomEditMessage = 0x4003, // 메시지 수정
}
```

Enum 등을 이용하여 만들면 관리하기가 더욱 편하다.  

그러면 서버 구조는 이정도로 끝내고 다음강에는 클라이언트 구조를 잡아 보겠다.