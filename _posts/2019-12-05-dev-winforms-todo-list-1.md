---
layout: post
title:  "[WindowsForm 기초] 1.Todo List 만들기"
subtitle:  "Todo List 레이아웃 구성하기"
categories: develop
tags: winforms
---

## 시작하기 전에

프론트엔드 기초를 할때 흔하게 만드는 Todo List를 구현해보겠다.

해당 Todo List를 만들면서 WindowsForm 에서 사용하는 기본적인 컨트롤부터 시작하여

커스텀 컨트롤 만들기, Nuget 을 통한 외부 라이브러리 활용, GDI+ 를 이용하여 그리기,  
Per Pixel Alpha Blend 를 이용하여 알파 값을 이용한 폼 만들기 등의 심화 과정까지 적을 생각이다.

강좌를 적을때마다 말하는 것이만 필자가 사용하는 방식이 정답은 아니다.  
다른 방식을 이용하여 구현할 수도 있고 필자가 사용하는 방식이 비효울 적일 수 있다.

또한 해당 글에 모든 내용을 적기보다 외부 링크를 이용하여 설명하는 경우가 많다(주로 MSDN) 이점 참고해서 봐주면 좋겠다.

강좌를 보다가 이런방법이 더 효율적이다 하는 부분이나 질문이 있으면 댓글을 남겨주길 바란다.

## 폼 레이아웃 구상하기

일단 처음으로 폼의 레이아웃을 잡아보자.

![레이아웃](/assets/img/dev/winforms/todo-list/1/레이아웃.png)

위의 레이아웃은 Panel과 [Dock 옵션](https://docs.microsoft.com/ko-kr/dotnet/framework/winforms/controls/how-to-dock-controls-on-windows-forms)을 이용한 것이다.

Dock 옵션은 간단하게 말하자면 해당 컨트롤의 위치를 가장자리에 도킹하거나 폼 또는 컨테이너 컨트롤에 채울 수 있도록 하는 옵션이다.

해당 옵션을 이용하여 3개의 `Panel`을 배치하였다.

**Form.Panel**  
  
| 이름             | Dock 옵션 | 위치          | 높이  |
|----------------|---------|-------------|-----|
| panelTitle     | Top     | 상단          | 90  |
| panelMain      | Fill    |             | 85  |
| panelInputData | Top     | panelMain내부 | 650 |
  

> **Dock 컨트롤러의 우선순위 지정**  
> 
>맨 앞으로 가져오기, 맨 뒤로 보내기 2가지 버튼을 통하여 조절할 수 있다.  
>![위치조정](/assets/img/dev/winforms/todo-list/1/위치조정.png)

위치를 잡았으면 색을 골라보자.  
필자는 [해당 사이트](https://paletton.com/#uid=13x0u0khZWH4b+8bMX+n-WetUWn) 를 이용하여 폼 디자인에 사용할 색을 결정하였다.

![중양정렬](/assets/img/dev/winforms/todo-list/1/색상결정.png)


색을 넣고 `panelTitle`에 `Label`을 이용하여 타이틀을 넣어준다.  
필자는 간단하게 Todo List라고 넣었다.  

**Label - labelTitle**  
   
| 속성        | 값                 |
|-----------|-------------------|
| (Name)    | labelTitle        |
| Text      | Todo List         |
| Font.Name | Segoe UI Semilight|
| Font.Size | 24               |
  

> **컨트롤러 중양 정렬하기**  
> 
>부모컨트롤러와 정렬할 컨트롤러를 클릭한뒤 상단의 버튼을 이용하여 정렬이 가능하다.
>![중양정렬](/assets/img/dev/winforms/todo-list/1/중양정렬.png)

다음으로는 `panelInputData` 부분에 입력을 위한 `TextBox`와 `Button`을 추가한다.

**TextBox - textBoxInput**  
  
| 속성          | 값                                   |
|-------------|-------------------------------------|
| (Name)      | textBoxInput                        |
| BackColor   | 222, 239, 253 |
| BorderStyle | None                                |
| Font.Name   | Segoe UI                            |
| Font.Size   | 12                                  |
  

**Button - buttonInput**  
  
| 속성                        | 값                 |
|---------------------------|-------------------|
| (Name)                    | buttonInput       |
| Text                      | 추가                |
| BackColor                 | 75, 169, 241      |
| FlatStyle                 | Flat              |
| FlatAppearance.BorderSize | 0                 |
| Font.Name                 | Segoe UI Semibold |
| Font.Size                 | 10                |
  

컨트롤을 배치 하고보니 `panelInputData` 가 하단에 있는 것이 더 좋아보여 위치를 변경하였다.
Dock와 Panel을 이용하여 만들면 이런식으로 쉽게 위치의 변경이 가능하다.   

**Form.Panel**  
  
| 이름             | Dock 옵션        | 위치          | 높이        |
|----------------|----------------|-------------|-----------|
| panelTitle     | Top            | 상단          | ~~80~~ 65 |
| panelMain      | Fill           |             | 620        |
| panelInputData | ~~Top~~ Bottom | panelMain내부 | 55       |
  

여기까지 작업하면 어느정도 모양이 나올 것이다.  
하지만 `textBoxInput`의 위치를 한번에 알 수 없고 Panel 간의 경계 부분이 부자연스럽다.  
이를 해결하기 위해 선을 몇 가닥 그려보자.

`OnPaint` 메소드를 이용할 수도 있지만 지금은 간단하게 Label을 이용하여 보겠다.

**Label**  
  
| 속성        | 값            |
|-----------|--------------|
| Text      |              |
| BackColor | 31, 150, 242 |
| AutoSize  | False        |
  

이제 위 Label을 3개 복사하여 `textBoxInput`하단에 하나, `panelTitle`하단에 하나, `panelInputData`상단에 하나 설치한다

![1차디자인](/assets/img/dev/winforms/todo-list/1/1차디자인.png)

여기까지 기본적인 디자인을 만들어 보았다.

위 부분까지 만든 [소스코드](https://github.com/Hot-key/Winform-TodoList/tree/16641a44dcba8538a4d9eb7b4cdbb7d677058b0d) 이다.

다음 강좌에서는 User control 을 이용하여 Todo List 에 표시될 아이템을 만들고 추가하는 것 까지 만들어 보자.