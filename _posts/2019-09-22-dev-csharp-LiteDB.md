---
layout: post
title:  "C#으로 로컬 NoSql 사용하기"
subtitle:   "LiteDb 사용하기"
categories: develop
tags: csharp
---

각종 프로그램을 만들다 보면 데이터를 저장 할 필요가 있다.

적은 양의 데이터면 text 파일에 저장해서 이용이 가능하지만  
데이터의 양이 많아지면 어림도 없는 방법이다.

보통 이런상황에서 사용하는 로컬db가 하나 있다.  
바로 SQLite 다.  

하지만 SQLite 는 기본적으로 sql 문으로 작동하는 rdb 이므로 c# 코드와의 결합이 힘든 부분이 있다.

SQLite 용 EF 등을 이용하면 해결이 가능하지만 추가로 설치할 필요가 있으니 불편 할 수 있다.

이런 상황에 이용 가능한 .NET 용 내장 NoSQL 데이터베이스가 있다.  
바로 [litedb](https://www.litedb.org/)다.

이름에서 볼 수 있듯이 매우 가벼운 db로 단일 dll 파일(350kb)만 있으면 사용 가능한 db 이다.

해당 DB 는 Nuget 에서 받아서 바로 사용 가능하다.

![버튼](/assets/img/dev/csharp/LiteDB/nuget.PNG)  


[소스코드(github))]()