---
layout: post
title:  "[SuperSocket으로 서버 만들기] 0.SuperSocket 이란?"
subtitle:   "1.SuperSocket의 기본 사용법"
categories: develop
tags: csharp
---

# SuperSocket 이란?

SuperSocket은 kerryjiang가 c# 을 이용하여 만든 고성능 소켓서버 라이브러리로
깃허브에서 오픈소스로 개발 중이다.

[SuperSocket 깃허브주소](https://github.com/kerryjiang/SuperSocket)


이름에서 볼 수 있듯이 c#으로 만들었다고 믿기 힘든 Super한 성능을 보여준다.  
또한 다양한 커스터마이징이 가능하다는 점이 특징이다.

아직 Core 버전은 공식적으로 없지만 깃허브에 jacking75 이 구현한 .net core 버전이 있다.

[.net core 버전](https://github.com/jacking75/SuperSocketLite)

다음 글에서는 내가 SuperSocket을 사용한 방법에 대하여 적어 보겠다.

다음 글에 나오는 내용이 SuperSocket을 이런식으로 사용해라 하는 정답은 아니다.  
그저 나는 이런 식으로 사용하였다는 한가지 예시 중 하나라고 생각하고 보면 될 것 같다.

참고로 개발환경은 윈도우에서 VS2019를 이용하였다.