---
layout: post
title:  "[WindowsForm] Calendar 컨트롤 - 달력을 이용해 보자"
subtitle:   "Button 컨트롤이란"
categories: develop
tags: winforms
---

최근에 윈폼 작업을 하다가 달력 컨트롤이 필요한 상황이 있었다.  
하지만 달력을 처음부터 만들기는 귀찮아서 찾은 컨트롤이 있다.  

[Calendar.NET](https://www.codeproject.com/Articles/378900/Calendar-NET)

하지만 내가 원하던 그런 달력이 아니였다.  
그래서 달력을 수정 하기로 했다.  

수정된 점  

1. 날짜의 클릭이 가능하도록 하며 이벤트를 이용하여 클릭한 날짜를 알 수 있도록 한다.
2. 다른버튼 (Today) 등의 버튼도 이벤트를 이용하여 클릭 시점을 알 수 있도록 한다.
3. 선택한 날짜를 강조표시 한다.
4. 한글이 잘 보이도록 폰트를 조정한다.

이벤트는 사진에서 볼 수 있는 것 처럼  

![버튼](/assets/img/dev/winforms/calendar/event.png)  

날짜 변경시 호출되는 것 - CalenderDateChange  
달력의 날짜 클릭시 호출되는 것 - DateCalenderClick  
달력의 다음달 버튼(▶) 클릭시 호출되는것 - RightButtonClick  
달력의 이전달 버튼(◀) 클릭시 호출되는것 - LeftButtonClick  
달력의 오늘 클릭시 호출되는 것 - TodayButtonClick  

으로 총 5개가 있다.
 
완성된 모습이다.

![버튼](/assets/img/dev/winforms/calendar/calendar2.gif)  

해당 파일의 소스코드는 [github](https://github.com/Hot-key/CalendarNET)에서 볼 수 있다.

다른 수정이 필요하다고 생각하는 부분이 있으면 댓글로 남겨주면 가능한 범위에서 수정을 해 보겠다.