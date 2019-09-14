---
layout: post
title:  "[AutoHotkey]원클릭 아이피 고정/유동 변경기"
subtitle:   "유동,고정 아이피 변경기"
categories: project
tags: server
---

# 유동,고정 아이피 변경기

학교에서 사용하기 위해서 만든 프로그램 중 하나로

유동아이피와 고정아이피 변경을 할 수 있는 프로그램이다.

![폼 이미지](/assets/img/projects/other/autohotkey-ip/form.png)

사용법은 간단하다 우축의 리스트에서 변경할 인터페이스를 선택한 후

고정버튼이나 동적 버튼을 누르면 된다.

​
블러오기와 저장버튼은 netsh -c interface dump 명령을 이용해서 만들었다.
​
작동에 문제가 있을시 Set 폴더 안에 있는 파일을 삭제해주면 아이피를 다시 설정할 수 있다.

[다운로드](http://blogattach.naver.net/5fca43f3e1b9bb6748abc4f9c62254258cdf20cd84/20190208_284_blogfile/rhanwnwmf_1549555714102_b3ve0p_exe/%EC%95%84%EC%9D%B4%ED%94%BC+%EB%B3%80%EA%B2%BD%EA%B8%B0.exe)

[소스코드](http://blogattach.naver.net/5bce47f7e5b9bf634cafc0fdc226502188db24c988/20190208_245_blogfile/rhanwnwmf_1549555734024_r9WYs0_zip/%EC%86%8C%EC%8A%A4%EC%BD%94%EB%93%9C.zip)