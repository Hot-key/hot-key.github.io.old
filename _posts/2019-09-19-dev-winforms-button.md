---
layout: post
title:  "[windows Form] Button 컨트롤이란"
subtitle:   "Button 컨트롤이란"
categories: develop
tags: winforms
---

# Button 컨트롤이란

## 기본정보
이름에서 볼 수 있듯이 그냥 버튼입니다.

![버튼](/assets/img/dev/winforms/button/button.PNG)  
모양은 위에 나온 사진처럼 생겼습니다.

## 기능

버튼하면 가장 중요한 기능이 무엇이라고 생각합니까?  
바로 누르는 것 입니다.

버튼을 눌른 시점을 얻기 위해서는 이벤트를 등록해야 합니다.  

등록방법으로는 크게 3가지가 있습니다.

### 1. 디자이너에서 버튼을 더블클릭한다.
   가장 간편한 방법으로 필자도 자주 시용하고 있는 기능 입니다.  
   Button 컨트롤러는 디자이너 창에서 두번 클릭하는 것 만으로 클릭 이벤트 등록이 가능합니다.

### 2. 속성창을 이용한다.  
   
![버튼](/assets/img/dev/winforms/button/button2.PNG)  
속성창에서 번개모양 버튼을 누르면

등록가능한 이벤트 목록이 나옵니다.

![버튼](/assets/img/dev/winforms/button/button3.PNG)  
여기서 Click를 찾아서 더블클릭하거나 원하는 메서드명을 입력하면  
VisualStudio 가 알아서 구현합니다.

### 3. 직접 이벤트를 연결한다.
   
   별로 추천하지는 않는 방법이지만 이벤트를 직접 등록 할 수 있습니다.

   ```{버튼이름}.Click += {메서드 이름};```  
   의 코드를 이용하여 등록이 가능 합니다.

![버튼](/assets/img/dev/winforms/button/button4.PNG)  
버튼이름은 속성의 (Name) 에서 확인이 가능합니다.

여기서 메서드는 다음의 코드와 같은 형식으로 작성하여야 합니다.

```csharp
private void Button1_Click(object sender, EventArgs e)
{

}
```

이러면 필자의 경우에는 다음과 같은 코드로 구현이 가능합니다.

```csharp
public Form1()
{
    InitializeComponent();

    button1.Click += Button1_Click;
}

private void Button1_Click(object sender, EventArgs e)
{
    // 버튼클릭시 진행할 내용
}
```

## 디자인

![버튼](/assets/img/dev/winforms/button/button5.PNG)
![버튼](/assets/img/dev/winforms/button/button6.PNG)  

속성에서 버튼의 FlatStyle 값을 변경하여 모양을 변경할 수 있다.

위의 사진의 버튼의 모양은 Flat, Popup, Standard, System 순으로 배치 한 것이다.

![버튼](/assets/img/dev/winforms/button/button7.gif)  

위의 사진처럼 버튼의 스타일 마다 마우스가 올라갔을때 모양과
클릭시 모양이 달라진다.