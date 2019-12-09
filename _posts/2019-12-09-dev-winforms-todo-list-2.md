---
layout: post
title:  "[WindowsForm 기초] 2.Todo List 만들기"
subtitle:  "Todo List 에 넣을 항목 만들기"
categories: develop
tags: winforms
---

## Todo List 에 넣을 항목 만들기

Todo List 의 기본 레이아웃을 만들었으니 다음으로 Todo List에 넣을 항목을 만들어 보자.

일단 클래스 파일 하나을 만들고 `UserControl`을 상속받는 `TodoItem`을 만들어준다.

'추가' -> '사용자 정의 컨트롤'을 통하여 `UserControl`을 상속받는 클래스를 만들 수 있지만 디자인 파일이 추가로 생성되므로 필자는 사용하지 않았다.

그러면 Todo Item 에 들어갈 내용을 생각하여 보자.

일단 CheckBox와 내용을 표시할 Text가 필요 할 것이다.

CheckBox는 Winforms의 기본 컨트롤을 이용할 수 있지만 기본 컨트롤은 체크할 수 있는 영역의 크기 조절이 불가능하다.

그러므로 `TodoItem`의 `OnPaint` 메서드을 `override`하여 그려준다.

```csharp
public class TodoItem : UserControl
{
    private Font font = new Font("Segoe UI", 14F, System.Drawing.FontStyle.Regular,
        System.Drawing.GraphicsUnit.Point, ((byte) (0)));

    private bool checked_ = false;
    
    private string text;

    [
        Category("모양"),
        Description("버튼에 표시되는 문자열을 지정합니다."),
    ]
    [Browsable(true), EditorBrowsable(EditorBrowsableState.Always), Bindable(true)]
    [DesignerSerializationVisibility(DesignerSerializationVisibility.Visible)]
    public override string Text
    {
        get => text;
        set => text = value;
    }

    [DefaultValue(typeof(Font), "Segoe UI, 14pt, style=Regular")]
    public new Font Font
    {
        get => font;
        set => font = value;
    }

    [DefaultValue(false)]
    public bool Checked
    {
        get => checked_;
        set => checked_ = value;
    }

    protected override void OnPaint(PaintEventArgs e)
    {
        Graphics g = e.Graphics;

        float sizeInPoints = this.Font.SizeInPoints / 72 * e.Graphics.DpiX;

        var rc = new Rectangle(new Point(Height / 6, Height / 4), new Size(Height / 2, Height / 2));

        ControlPaint.DrawCheckBox(g, rc,
            this.checked_ ? ButtonState.Flat | ButtonState.Checked : ButtonState.Flat | ButtonState.Normal);

        g.DrawString(this.text, this.Font, Brushes.Black, rc.Width + rc.X * 2, (Height - sizeInPoints) / 2 - 5);

        base.OnPaint(e);
    }

    protected override void OnClick(EventArgs e)
    {
        checked_ = !checked_;
        this.Refresh();

        base.OnClick(e);
    }
}
```
위 코드를 설명하자면 프로퍼티를 통하여 디자이너에서 편집할 수 있는 값을 만든다.

또한 `DefaultValue`을 통하여 기본적으로 설정될 값을 정한다.

위의 변수 선언 구조에 대한 자세한 정보는 [msdn](https://docs.microsoft.com/ko-kr/dotnet/framework/winforms/controls/defining-a-property-in-windows-forms-controls)에서 볼 수 있다.

`OnPaint`메서드에서는 폰트사이즈와 `CheckBox`가 그려질 공간의 크기를 구한다. 

이후 `ControlPaint.DrawCheckBox`을 이용하여 `CheckBox`를 그리고 `Graphics.DrawString`로 Text를 그린다.

`OnClick` 에서는 checked_의 값을 반전시킨 후 `this.Refresh();`을 호출한다.

이는 변경된 CheckBox 를 다시 그리기 위함이다.  

한번 만든 컨트롤의 크기와 배경색을 조절해서 `panelMain`에 넣어보자.

> **만든 컨트롤이 나오지 않을때**  
> F6 버튼을 누르거나 '빌드' -> '솔루션 빌드'를 한번 해보자.  
> 윈폼의 사용자 컨트롤의 변경사항은 빌드시 적용된다.
  

필자의 경우 아래와 같은 크기와 배경색을 이용하였다.
  
**TodoItem**
  
| 속성        | 값       |
|-----------|---------|
| BackColor | White   |
| Size      | 453, 75 |
  

![TodoItem](/assets/img/dev/winforms/todo-list/2/TodoItem.png)  
  
위 이미지를 보면 `TodoItem` 과 `panelMain`의 색상이 비슷해서 잘 보이지 않는다.

이를 해결하기 위하여 약간 그림자 비슷한 효과를 넣어보겠다.

`panelMain`내부에 그림자가 적용된 `TodoItem`이 추가될 panel을 하나 만들어 준다.

필자는 `panelTodoItem` 이라는 이름으로 만들었다.

**Panel - panelTodoItem**  
  
| 속성        | 값          |
|-----------|------------|
| Location  | 0, 0       |
| Dock      | Fill       |
| BackColor | WhiteSmoke |

그림자 효과는 [해당 글](https://stackoverflow.com/questions/2463519/drop-shadow-in-winforms-controls)을 참조하여 만들었다.

`panelTodoItem`을 더블클릭 하거나 속성창 상단의 번개모양(이벤트) 아이콘을 눌러
`Paint` 이벤트를 추가하여 준다. 

```csharp
private void panelTodoItem_Paint(object sender, PaintEventArgs e)
{
    Color[] shadow = new Color[4];
    shadow[0] = Color.FromArgb(181, 181, 181);
    shadow[1] = Color.FromArgb(195, 195, 195);
    shadow[2] = Color.FromArgb(211, 211, 211);

    using (Pen pen = new Pen(shadow[0]))
    {
        foreach (TodoItem item in panelTodoItem.Controls.OfType<TodoItem>())
        {
            Point pt = item.Location;
            pt.Y += item.Height;
            for (var sp = 0; sp < shadow.Length; sp++)
            {
                pen.Color = shadow[sp];
                e.Graphics.DrawLine(pen, pt.X + sp, pt.Y + sp, pt.X + item.Width + sp, pt.Y + sp);

                e.Graphics.DrawLine(pen, pt.X + item.Width + sp, pt.Y - item.Height + sp, pt.X + item.Width + sp, pt.Y + sp);
            }
        }
    }
}
```

해당 메서드를 추가하면 아래의 사진처럼 그림자 비슷한 효과가 추가될 것이다.

![TodoItem](/assets/img/dev/winforms/todo-list/2/TodoItem2.png)

그럼 다음으로 추가 버튼 클릭시 항목이 추가되는 것을 구현하여 보자.

추가버튼을 더블클릭 하거나 속성창 상단의 번개모양(이벤트) 아이콘을 눌러
`Click` 이벤트를 추가하여 준다. 

```csharp
private void buttonInput_Click(object sender, EventArgs e)
{
    TodoItem todoItem = new TodoItem();

    todoItem.Location = new Point(20, panelTodoItem.Controls.Count * 91 + 16);
    todoItem.Size = new Size(this.Width - 63, 75);
    todoItem.Text = textBoxInput.Text;
    todoItem.BackColor = Color.White;
    todoItem.Anchor = AnchorStyles.Left | AnchorStyles.Right | AnchorStyles.Top;
    panelTodoItem.Controls.Add(todoItem);

    panelTodoItem.Refresh();
    textBoxInput.Clear();   
}
```

코드 설명을 하자면 새로운 `TodoItem`을 만들고 Location, Size, Text등의 값을 할당한다.

그다음 할당된 `TodoItem`을 `panelTodoItem`의 `Control`목록에 추가하는 것 이다.

추가후 `panelTodoItem`을 다시 그리기 위하여 `Refresh()`을 호출한다.

여기까지 끝나면 버튼을 눌러 항목을 추가 할 수 있을 것 이다.

![TodoItem](/assets/img/dev/winforms/todo-list/2/TodoItem3.png)

항목을 추가하다 보면 화면을 넘어간다.  

하지만 스크롤이 불가능 하기 때문에 화면을 넘어간 항목을 볼 수 없다.  

그럼 이번에는 스크롤을 추가해보자.

스크롤 바를 넣기 위한 panel을 하나 만든다.

**Panel - panelTodoItemScroll**  

| 속성               | 값                        |
|------------------|--------------------------|
| Location         | 0, -17                   |
| AutoScrollMargin | 0, 32                    |
| Size             | 510, 654                 |
| BackColor        | WhiteSmoke               |
| AutoScroll       | True                     |
| Anchor           | Top, Bottom, Left, Right |
  

만든 panel 속에 기존에 만든 `panelTodoItem`을 넣어준다.

**Panel - panelTodoItem**  
  
| 속성        | 값                |
|-----------|------------------|
| Location  | ~~0, 0~~ 0, 17   |
| Dock      | ~~Fill~~  None   |
| Size      | 492, 606         |
| BackColor | WhiteSmoke       |
| Anchor    | Top, Left, Right |
  
여기서 `panelTodoItemScroll`을 폼의 크기보다 크게 만드는 이유는 스크롤 바를 일부분 숨겨 조금 더 심플한 디자인을 만들기 위하여 이다.  

스크롤바를 사용하기 위하여 코드를 약간 수정해 보자.

`Form.SizeChanged` 이벤트를 등록하여 준다.

Form을 누르고 속성창 상단의 번개모양(이벤트) 아이콘을 눌러
`SizeChanged` 이벤트를 추가하면 된다.

> **컨트롤러에 가려진 항목 누르기**  
> 
> 해당 컨트롤러의 위치에서 오른쪽클릭을 하거나 속성창의 드롭다운 박스를 이용하면 가려진 항목의 선택이 가능하다
> ![가려진 항목](/assets/img/dev/winforms/todo-list/2/가려진항목.png)
  

```csharp
private void Form1_SizeChanged(object sender, EventArgs e)
{
    panelTodoItem.Size = new Size(this.Width - 24, Math.Max(panelTodoItemScroll.Height - 48, (panelTodoItem.Controls.Count + 1) * 91 + 16 + 8));
    panelTodoItem.Refresh();
}

private void buttonInput_Click(object sender, EventArgs e)
{
    panelTodoItem.Size = new Size(this.Width - 24,  Math.Max(panelTodoItemScroll.Height - 48, (panelTodoItem.Controls.Count + 1) * 91 + 16 + 8));

    TodoItem todoItem = new TodoItem();

    todoItem.Location = new Point(20, panelTodoItem.Controls.Count * 91 + 16);
    todoItem.Size = new Size(this.Width - 63, 75);
    todoItem.Text = textBoxInput.Text;
    todoItem.BackColor = Color.White;
    todoItem.Margin = Padding.Empty;
    todoItem.Anchor = AnchorStyles.Left | AnchorStyles.Right | AnchorStyles.Top;
    panelTodoItem.Controls.Add(todoItem);

    panelTodoItem.Refresh();
    textBoxInput.Clear();   
}
```

코드의 변경 사항을 보면 사이즈를 조절하는 부분이 추가되었다.

사이즈 변경 부분에 보면 높이 값을 `Math.Max`를 사용하여 정하는 것을 볼 수 있다.

이는 높이 변화에 따라 스크롤 바가 없어질 수 있으므로 최소값을 지정한 것이다.

스크롤 바가 없어지면 일어나는 일이 궁금한 사람은 `Math.Max`의 첫번째 인수를 0 으로 변경하면 알 수 있다.

다음으로 `TodoItem`에 한번 마우스를 올렸을때 위로 올라오는 듯 한 느낌을 줘보자.

```csharp
public class TodoItem : UserControl
{
    private int zIndex = 0;

    private bool isMouseEnter = false;

    private Timer timer1 = new Timer();

    //...
}
```

해당 효과를 만들기 위하여 변수 3개를 선언하였다.

`zIndex`는 `TodoItem`의 올라온 정도를 파악하기 위한 변수이다.

`isMouseEnter`는 마우스가 올라왔는지 파악하기 위한 변수이다.

`timer1`은 위의 효과를 돌리기 위한 타이머 이다.

`OnLoad, OnMouseLeave, OnMouseEnter` 메서드를 오버로드 하여 만들어 보자.

```csharp
public class TodoItem : UserControl
{
    //...

    protected override void OnMouseEnter(EventArgs e)
    {
        isMouseEnter = true;
        base.OnMouseEnter(e);
    }

    protected override void OnMouseLeave(EventArgs e)
    {
        isMouseEnter = false;
        base.OnMouseLeave(e);
    }

    protected override void OnLoad(EventArgs e)
    {
        timer1.Interval = 25;
        timer1.Enabled = !DesignMode;
        timer1.Tick += timer1_Tick;

        base.OnLoad(e);
    }
}
```
`OnMouseEnter`와 `OnMouseLeave`에서는 간단하게 `isMouseEnter`의 값을 설정하여 준다.

`OnLoad`에서는 디자인 모드인지 확인 후 타이머를 켜준다.

```csharp
public class TodoItem : UserControl
{
    //...

    private void timer1_Tick(object sender, EventArgs e)
    {
        if (isMouseEnter)
        {
            if (zIndex < 5)
            {
                this.Location = new Point(this.Location.X - 1, this.Location.Y - 1);
                zIndex++;
            }
        }
        else
        {
            if (zIndex > 0)
            {
                this.Location = new Point(this.Location.X + 1, this.Location.Y + 1);
                zIndex--;
            }
        }
    }
}
```

타이머가 돌아가는 `timer1_Tick` 메서드 이다.   
해당 애니메이션에 대한 실질적 처리가 이루어지는 부분이기도 하다.

구현은 간단하다.  
타이머를 돌리며 마우스가 올라가 있는지 확인하고, zIndex을 더하거나 빼가면서 위치를 변경한다.  

여기 까지 만들면 최종적으로 이런 기능을 하는 Todo List 를 만들 수 있다.

![2강완성](/assets/img/dev/winforms/todo-list/2/2강완성.gif)

위 부분까지 만든 [소스코드](https://github.com/Hot-key/Winform-TodoList/tree/459d2518fbf3cc6a69be4d04c7db07fbb4d455c8)이다.

다음 강좌에서는 항목을 지우고 순서를 변경하는 작업까지 해보겠다.