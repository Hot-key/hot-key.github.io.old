---
layout: post
title:  "[Sever]SuperSocket 기반의 서버 라이브러리"
subtitle:   "SuperSocket 기반의 서버 라이브러리"
categories: project
tags: server
---

# NCS-Networking-with-CSharp
### https://github.com/Hot-key/NCS-Networking-with-CSharp

c# SuperSocket를 이용하여 만든 소켓서버

기존의 SuperSocket처럼 다양한 커스터마이징이 가능하여 다양한 곳에 사용이 가능하다.

# 기본 사용법
## 서버 열기

NCS 를 이용하여 서버를 만드는 것은 매우 간단하다..

먼저 NcsUser<T> 를 상속하는 클레스를 만든다.

```csharp
public class NewUser : NcsUser<NewUser>
{
    // user data
}
```
이후 해당 코드를 실행하면 서버는 실행된다.

```csharp
var server = new NcsMain<NewUser>(new ServerConfig()
{
    Port = 65535,
    Ip = "Any",
});
```

## 메시지 받기

NCS는 메시지를 받기위하여 복작한 작업이 필요 없다.

먼저 NcsModule<T> 를 상속받는 클레스를 하나 만든다.

클레스명은 상관 없다.
```csharp
public class Test : NcsModule<NewUser>
{
}
```

만들었으면 생성자를 만들어 준다.

```csharp
public class Test : NcsModule<NewUser>
{
    public Test()
    {
    }
}
```

이제 메시지를 받을 준비는 끝났다.

```csharp
public class Test : NcsModule<NewUser>
{
    public Test()
    {
        NewRequestReceived = (user, info) =>
        {
            Console.WriteLine("NewRequestReceived");
        };
    }
}
```

NCS 로 전달된 모든 메시지는 NewRequestReceived로 전달된다.


NewRequestReceived내부에서 switch 문을 통하여 메시지를 분류할 수 있다.

하지만 이런 구조는 NewRequestReceived 내부의 복잡도를 높이고 코드 분석을 어렵게 만든다.

그래서 만들어진 것이 packet[Data] 이다.

이를 이해하기 위해서는 일단 NCS 의 기본 패킷구조를 이해하여야 하지만 일단 사용하여 보자.

```csharp
public class Test : NcsModule<NewUser>
{
    public Test()
    {
        packet[(ushort)1] = (user, info) =>
        {
            Console.WriteLine("Packet 1 Received");
        };
    }
}
```

이러면 패킷 내부의 타입이 1 인 패킷을 검출할 수 있다.

## 패킷구조

NCS는 CGD.buffer 을 약간 수정한 패킷을 사용한다..

NCS는 해더와 본문이 있는 구조를 가지고 있다.

NCS의 해더는 4바이트로 길이를 표시하고 2 바이트로 메시지 분류를 위한 타입을 표시한다.

여기서 2바이트의 메시지 타입은 packet[Data] 의 분류에 사용한다.

## 패킷 구조 변경
위에서 말한 해더는 NcsOption 을 이용하여 사용자가 원하는 값으로 변경이 가능하다.

한번 2바이트로 길이를 표시하고 2 바이트로 메시지 분류를 하는 NcsOption 을 만들어 보자.

```csharp
var option = new NcsOption
(
    buffer => buffer.get_front_short(2), // 2 바이트 떨어진 곳에서 short를 가져온다. 
    (header, offset, length) => // 본문의 길이를 계산한다.
    {
        return (int) header[offset] +
               (int) header[offset + 1] * 256;
    },
    4 // 해더의 총 길이이다. 길이 2바이트 메시지 타입 2바이트
);
```
이러면 2바이트로 길이를 표시하고 2 바이트로 메시지 분류를 하는 NcsOption이 만들어졌다.

한번 이를 서버에 적용해보자.

```csharp
var server = new NcsMain<NewUser>(new ServerConfig()
{
    Port = 65535,
    Ip = "Any",
}, option); // 위에서 만든 option 이다
```
자 이러면 이 서버는 위에서 만든 option 에 따라 패킷을 분석한다.
