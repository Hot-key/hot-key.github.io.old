---
layout: post
title:  "[SuperSocket으로 서버 만들기] 1-1.에코서버를 만들어보자"
subtitle:   "1-1.SuperSocket의 구조"
categories: develop
tags: csharp
---

## SuperSocket의 구조

처음 접하면 어렵다고 생각할 수도 있지만 SuperSocket의 구조는 간단하다.

```csharp
ReceiveFilter<T>
RequestInfo<T>
AppServer<T, RequestInfo>
AppSession<T, RequestInfo>
```

SuperSocket은 크게보면 위의 4가지로 이루어저 있다.

```ReceiveFilter```는 SuperSocket을 통하여 들어온 메시지를 만드는 부분으로 위의 4 가지 중에서 가장 처음으로 메시지가 도착하는 곳이다.  
해당 부분에서 하는 일은 메시지의 길이를 확인하고 더 받을 필요가 있는지 결정한다.  
또한 메시지를 다 받으면 ```RequestInfo``` 를 만드는 작업을 한다.


```RequestInfo```는 간단하게 해서 서버에서 온 메시지라고 볼 수 있다.  
```RequestInfo```는 위에서 설명한 ```ReceiveFilter``` 에서 만들어진다.


```AppServer```는 이름에서 볼 수 있듯이 서버의 메인 부분으로 새로운 세션이 서버에 연결하였을때, 메시지를 받았을때, 세션의 연결이 끊어질때 등의 상황에 호출할 메서드를 연결할 수 있다.

```AppSession```은 서버에 들어온 유저라고 생각하면 된다.  
하나의 ```AppSession``` 은 하나의 유저이다. 

그러면 SuperSocket의 구조는 여기서 마치고 나머지 부분은 다음 강에서 코드와 함께 설명하겠다.

[SuperSocket으로 서버 만들기 - 목차](/contents/2019/09/24/contents-supersocket/)