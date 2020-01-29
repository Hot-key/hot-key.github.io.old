---
layout: post
title:  "[WindowsForm] 윈폼에서도 Aero Glass 효과를 사용해보자"
subtitle:  "Aero glass 효과 만들기"
categories: develop
tags: winforms
---

인터넷에 검색을 해보면 wpf 로 Aero Glass를 만드는 법은 많이 나와있다.

나역시 Wpf로 Aero Glass를 만드는 법 에 관련하여 [쓴 글](https://hot-key.github.io/develop/2019/09/29/dev-wpf-aeroglass/)이 있다.

하지만 WindowsForm을 이용하여 Aero Glass효과를 만드는 방법에 대한 글은 별로 없어서 한번 구현하면서 찾아보고 사용한 내용을 정리해볼까 한다. 

일단 Wpf에서 Aero Glass 효과를 만드는 코드의 일부분 이다.

```csharp
class TransparencyConverter
{
    private readonly Window _window;

    public TransparencyConverter(Window window)
    {
        _window = window;
    }

    internal void EnableBlur()
    {
        var windowHelper = new WindowInteropHelper(_window);

        var accent = new AccentPolicy();
        var accentStructSize = Marshal.SizeOf(accent);
        accent.AccentState = AccentState.ACCENT_ENABLE_BLURBEHIND;

        var accentPtr = Marshal.AllocHGlobal(accentStructSize);
        Marshal.StructureToPtr(accent, accentPtr, false);

        var data = new WindowCompositionAttributeData();
        data.Attribute = WindowCompositionAttribute.WCA_ACCENT_POLICY;
        data.SizeOfData = accentStructSize;
        data.Data = accentPtr;

        SetWindowCompositionAttribute(windowHelper.Handle, ref data);

        Marshal.FreeHGlobal(accentPtr);
    }
    // ... 중략
}
```

위의 코드를 winforms 에서 간단하게 사용할 수 있도록 수정하여 보았다.

```csharp
internal class TransparencyConverter
{
    internal static void EnableBlur(IntPtr handle)
    {
        var accent = new Win32.AccentPolicy();
        var accentStructSize = Marshal.SizeOf(accent);
        accent.AccentState = state;

        var accentPtr = Marshal.AllocHGlobal(accentStructSize);
        Marshal.StructureToPtr(accent, accentPtr, false);

        var data = new Win32.WindowCompositionAttributeData();
        data.Attribute = Win32.WindowCompositionAttribute.WCA_ACCENT_POLICY;
        data.SizeOfData = accentStructSize;
        data.Data = accentPtr;

        Win32.SetWindowCompositionAttribute(handle, ref data);

        Marshal.FreeHGlobal(accentPtr);
    }
    // ... 중략
}
```

여기까지 완성한 결과물이다.

![form1](/assets/img/dev/winforms/aeroglass/form1.png)

잘 작동하는 것 처럼 보이지만 무언가 이상한걸 알 수 있다.


![form2](/assets/img/dev/winforms/aeroglass/form2.png)

이 이미지를 보면 버튼의 글자도 투명이 적용되었다.  
이러면 폼을 사용하기 힘들다.

이를 해결하기 위하여 Per Pixel Alpha Blend 을 이용하여 그리는 코드를 작성하였다. 

```csharp
public class NanoAeroForm : Form
{
    public new Color BackColor;

    private Timer drawTimer = new Timer();
    public NanoAeroForm()
    {
        this.FormBorderStyle = FormBorderStyle.None;
        SetStyle(ControlStyles.SupportsTransparentBackColor, true);
    }

    protected override void OnLoad(EventArgs e)
    {
        if (!DesignMode)
        {
            drawTimer.Interval = 1000 / 60;
            drawTimer.Tick += DrawForm;
            drawTimer.Start();
            TransparencyConverter.EnableBlur(this.Handle, Win32.AccentState.ACCENT_ENABLE_BLURBEHIND);
        }
        base.OnLoad(e);
    }

    private void DrawForm(object sender, EventArgs e)
    {
        using (Bitmap backImage = new Bitmap(this.Width, this.Height))
        {
            using (Graphics graphics = Graphics.FromImage(backImage))
            {
                Rectangle rectangle = new Rectangle(0, 0, this.Width - 1, this.Height - 1);
                using (Brush backColorBrush = new SolidBrush(this.BackColor))
                {
                    graphics.FillRectangle(backColorBrush, rectangle);
                }

                OnPaint(new PaintEventArgs(graphics,rectangle));

                foreach (Control ctrl in this.Controls)
                {
                    using (Bitmap bmp = new Bitmap(ctrl.Width, ctrl.Height))
                    {
                        Rectangle rect = new Rectangle(0, 0, ctrl.Width, ctrl.Height);
                        ctrl.DrawToBitmap(bmp, rect);
                        graphics.DrawImage(bmp, ctrl.Location);
                    }
                }
                PerPixelAlphaBlend.SetBitmap(backImage, Left, Top, Handle);
            }
        }
    }

    protected override void Dispose(bool disposing)
    {
        drawTimer.Stop();
        drawTimer.Dispose();
        base.Dispose(disposing);
    }

    protected override CreateParams CreateParams
    {
        get
        {
            CreateParams cp = base.CreateParams;
            if (!DesignMode)
            {
                cp.ExStyle |= 0x00080000;
            }
            return cp;
        }
    }
}
```

코드 설명을 하자면  
OnLoad 에서 타이머로 DrawForm 메서드를 호출하여 폼을 그린다.

`DrawForm` 에서는 폼과 같은 크기의 Graphics을 만들어 각종 요소를 그리는 것을 볼 수 있다.  
또한 메모리 누수를 방지하기 위하여 Using 을 사용하여 관리한다.


크게 어려운 코드는 아니지만 관련된 글이 별로 없어서 힘들기는 하였다.


다른 궁금한 부분이 있으면 댓글을 통하여 질문 부탁한다.

해당 폼을 사용하기 위한 전체코드이다.
<details>
<summary>코드 접기/펼치기 버튼</summary>
<div markdown="1">

// NanoAeroForm.cs
public class NanoAeroForm : Form
{
    public new Color BackColor;

    private Timer drawTimer = new Timer();
    public NanoAeroForm()
    {
        this.FormBorderStyle = FormBorderStyle.None;
        SetStyle(ControlStyles.SupportsTransparentBackColor, true);
    }

    protected override void OnLoad(EventArgs e)
    {
        if (!DesignMode)
        {
            drawTimer.Interval = 1000 / 60;
            drawTimer.Tick += DrawForm;
            drawTimer.Start();
            TransparencyConverter.EnableBlur(this.Handle, Win32.AccentState.ACCENT_ENABLE_BLURBEHIND);
        }
        base.OnLoad(e);
    }

    private void DrawForm(object sender, EventArgs e)
    {
        using (Bitmap backImage = new Bitmap(this.Width, this.Height))
        {
            using (Graphics graphics = Graphics.FromImage(backImage))
            {
                Rectangle rectangle = new Rectangle(0, 0, this.Width - 1, this.Height - 1);
                using (Brush backColorBrush = new SolidBrush(this.BackColor))
                {
                    graphics.FillRectangle(backColorBrush, rectangle);
                }

                OnPaint(new PaintEventArgs(graphics,rectangle));

                foreach (Control ctrl in this.Controls)
                {
                    using (Bitmap bmp = new Bitmap(ctrl.Width, ctrl.Height))
                    {
                        Rectangle rect = new Rectangle(0, 0, ctrl.Width, ctrl.Height);
                        ctrl.DrawToBitmap(bmp, rect);
                        graphics.DrawImage(bmp, ctrl.Location);
                    }
                }
                PerPixelAlphaBlend.SetBitmap(backImage, Left, Top, Handle);
            }
        }
    }

    protected override void Dispose(bool disposing)
    {
        drawTimer.Stop();
        drawTimer.Dispose();
        base.Dispose(disposing);
    }

    protected override CreateParams CreateParams
    {
        get
        {
            CreateParams cp = base.CreateParams;
            if (!DesignMode)
            {
                cp.ExStyle |= 0x00080000;
            }
            return cp;
        }
    }
}

// Win32.cs
internal class Win32
{
    public enum Bool
    {
        False = 0,
        True
    };


    [StructLayout(LayoutKind.Sequential)]
    public struct Point
    {
        public Int32 x;
        public Int32 y;

        public Point(Int32 x, Int32 y) { this.x = x; this.y = y; }
    }


    [StructLayout(LayoutKind.Sequential)]
    public struct Size
    {
        public Int32 cx;
        public Int32 cy;

        public Size(Int32 cx, Int32 cy) { this.cx = cx; this.cy = cy; }
    }


    [StructLayout(LayoutKind.Sequential, Pack = 1)]
    struct ARGB
    {
        public byte Blue;
        public byte Green;
        public byte Red;
        public byte Alpha;
    }


    [StructLayout(LayoutKind.Sequential, Pack = 1)]
    public struct BLENDFUNCTION
    {
        public byte BlendOp;
        public byte BlendFlags;
        public byte SourceConstantAlpha;
        public byte AlphaFormat;
    }


    public const Int32 ULW_COLORKEY = 0x00000001;
    public const Int32 ULW_ALPHA = 0x00000002;
    public const Int32 ULW_OPAQUE = 0x00000004;

    public const byte AC_SRC_OVER = 0x00;
    public const byte AC_SRC_ALPHA = 0x01;


    [DllImport("user32.dll", ExactSpelling = true, SetLastError = true)]
    public static extern Bool UpdateLayeredWindow(IntPtr hwnd, IntPtr hdcDst, ref Point pptDst, ref Size psize, IntPtr hdcSrc, ref Point pprSrc, Int32 crKey, ref BLENDFUNCTION pblend, Int32 dwFlags);

    [DllImport("user32.dll", ExactSpelling = true, SetLastError = true)]
    public static extern IntPtr GetDC(IntPtr hWnd);

    [DllImport("user32.dll", ExactSpelling = true)]
    public static extern int ReleaseDC(IntPtr hWnd, IntPtr hDC);

    [DllImport("gdi32.dll", ExactSpelling = true, SetLastError = true)]
    public static extern IntPtr CreateCompatibleDC(IntPtr hDC);

    [DllImport("gdi32.dll", ExactSpelling = true, SetLastError = true)]
    public static extern Bool DeleteDC(IntPtr hdc);

    [DllImport("gdi32.dll", ExactSpelling = true)]
    public static extern IntPtr SelectObject(IntPtr hDC, IntPtr hObject);

    [DllImport("gdi32.dll", ExactSpelling = true, SetLastError = true)]
    public static extern Bool DeleteObject(IntPtr hObject);


    public const int WM_NCLBUTTONDOWN = 0xA1;
    public const int HT_CAPTION = 0x2;

    [System.Runtime.InteropServices.DllImport("user32.dll")]
    public static extern int SendMessage(IntPtr hWnd, int Msg, int wParam, int lParam);

    [System.Runtime.InteropServices.DllImport("user32.dll")]
    public static extern bool ReleaseCapture();

    [DllImport("user32.dll")]
    internal static extern int SetWindowCompositionAttribute(IntPtr hwnd, ref WindowCompositionAttributeData data);

    [StructLayout(LayoutKind.Sequential)]
    internal struct WindowCompositionAttributeData
    {
        public WindowCompositionAttribute Attribute;
        public IntPtr Data;
        public int SizeOfData;
    }

    internal enum WindowCompositionAttribute
    {
        // ...
        WCA_ACCENT_POLICY = 19
        // ...
    }

    internal enum AccentState
    {
        ACCENT_DISABLED = 0,
        ACCENT_ENABLE_GRADIENT = 1,
        ACCENT_ENABLE_TRANSPARENTGRADIENT = 2,
        ACCENT_ENABLE_BLURBEHIND = 3,
        ACCENT_INVALID_STATE = 4
    }

    [StructLayout(LayoutKind.Sequential)]
    internal struct AccentPolicy
    {
        public AccentState AccentState;
        public int AccentFlags;
        public int GradientColor;
        public int AnimationId;
    }
}

// TransparencyConverter.cs
internal class TransparencyConverter
{
    internal static void EnableBlur(IntPtr handle, Win32.AccentState state = Win32.AccentState.ACCENT_ENABLE_TRANSPARENTGRADIENT)
    {
        var accent = new Win32.AccentPolicy();
        var accentStructSize = Marshal.SizeOf(accent);
        accent.AccentState = state;

        var accentPtr = Marshal.AllocHGlobal(accentStructSize);
        Marshal.StructureToPtr(accent, accentPtr, false);

        var data = new Win32.WindowCompositionAttributeData();
        data.Attribute = Win32.WindowCompositionAttribute.WCA_ACCENT_POLICY;
        data.SizeOfData = accentStructSize;
        data.Data = accentPtr;

        Win32.SetWindowCompositionAttribute(handle, ref data);

        Marshal.FreeHGlobal(accentPtr);
    }
}

// PerPixelAlphaBlend.cs
internal static class PerPixelAlphaBlend
{
    public static void SetBitmap(Bitmap bitmap, int left, int top, IntPtr handle)
    {
        SetBitmap(bitmap, 255, left, top, handle);
    }

    public static void SetBitmap(Bitmap bitmap, byte opacity, int left, int top, IntPtr handle)
    {
        if (bitmap.PixelFormat != PixelFormat.Format32bppArgb)
            throw new ApplicationException("The bitmap must be 32ppp with alpha-channel.");


        IntPtr screenDc = Win32.GetDC(IntPtr.Zero);
        IntPtr memDc = Win32.CreateCompatibleDC(screenDc);
        IntPtr hBitmap = IntPtr.Zero;
        IntPtr oldBitmap = IntPtr.Zero;

        try
        {
            hBitmap = bitmap.GetHbitmap(Color.FromArgb(0));
            oldBitmap = Win32.SelectObject(memDc, hBitmap);

            Win32.Size size = new Win32.Size(bitmap.Width, bitmap.Height);
            Win32.Point pointSource = new Win32.Point(0, 0);
            Win32.Point topPos = new Win32.Point(left, top);
            Win32.BLENDFUNCTION blend = new Win32.BLENDFUNCTION();
            blend.BlendOp = Win32.AC_SRC_OVER;
            blend.BlendFlags = 0;
            blend.SourceConstantAlpha = opacity;
            blend.AlphaFormat = Win32.AC_SRC_ALPHA;

            Win32.UpdateLayeredWindow(handle, screenDc, ref topPos, ref size, memDc, ref pointSource, 0, ref blend, Win32.ULW_ALPHA);
        }
        finally
        {
            Win32.ReleaseDC(IntPtr.Zero, screenDc);
            if (hBitmap != IntPtr.Zero)
            {
                Win32.SelectObject(memDc, oldBitmap);
                Win32.DeleteObject(hBitmap);
            }

            Win32.DeleteDC(memDc);
        }
    }
}

</div>
</details>

아직 개발중이기는 하지만 해당 [라이브러리를](https://github.com/Hot-key/NanoFormFramework) 사용하면 참조 추가후 `NanoAeroForm` 을 상속받아 사용가능하다.



참고한 글  
https://www.kechuang.org/t/79675  
https://stackoverflow.com/questions/10202130/how-draw-a-control-over-a-ws-ex-layered-form  
https://stackoverflow.com/questions/5015036/alpha-blended-control-on-layered-window-in-c-sharp  
https://blog.walterlv.com/post/create-blur-background-window-en.html  