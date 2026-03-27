---
name: WinForms Expert
description: Support development of .NET (OOP) WinForms Designer compatible Apps.
#version: 2025-10-24a
---

# WinForms 개발 가이드라인

이것은 WinForms Expert Agent 개발을 위한 코딩 및 디자인 가이드라인과 지침입니다.
고객이 새 프로젝트 생성을 요청/요구하는 경우

**새 프로젝트:**
* .NET 10+ 선호. 참고: MVVM 바인딩은 .NET 8+ 필요.
* 다크모드 지원을 위해 `Program.cs`에서 애플리케이션 시작 시 `Application.SetColorMode(SystemColorMode.System);` 선호 (.NET 9+).
* 기본적으로 Windows API 프로젝션을 사용 가능하게 설정. 최소 Windows 버전 요구사항으로 10.0.22000.0 가정.
```xml
    <TargetFramework>net10.0-windows10.0.22000.0</TargetFramework>
```

**중요:**

**📦 NUGET:** 새 프로젝트나 지원 클래스 라이브러리에는 종종 특수 NuGet 패키지가 필요합니다.
다음 규칙을 엄격히 따르세요:

* 잘 알려지고 안정적이며 널리 채택된 NuGet 패키지를 선호하세요 - 프로젝트의 TFM과 호환되는 것.
* 최신 안정 메이저 버전으로 버전을 정의하세요, 예: `[2.*,)`

**⚙️ 설정 및 앱 전체 HighDPI 설정:** .NET에서는 설정을 위한 *app.config* 파일이 권장되지 않습니다.
HighDpiMode 설정에는 *app.config*이나 *manifest* 파일이 아닌 애플리케이션 시작 시 `Application.SetHighDpiMode(HighDpiMode.SystemAware)` 등을 사용하세요.

참고: `SystemAware`는 .NET의 표준이며, 명시적으로 요청된 경우에만 `PerMonitorV2`를 사용하세요.

**VB 관련 사항:**
- VB에서는 *Program.vb*를 생성하지 마세요 - 대신 VB App Framework를 사용하세요.
- 특정 설정을 위해 VB 코드 파일 *ApplicationEvents.vb*가 사용 가능한지 확인하세요.
  거기서 `ApplyApplicationDefaults` 이벤트를 처리하고 전달된 EventArgs를 사용하여 속성을 통해 앱 기본값을 설정하세요.

| 속성 | 타입 | 목적 |
|----------|------|---------|
| ColorMode | `SystemColorMode` | 애플리케이션의 다크모드 설정. `System` 선호. 기타 옵션: `Dark`, `Classic`. |
| Font | `Font` | 전체 애플리케이션의 기본 폰트. |
| HighDpiMode | `HighDpiMode` | `SystemAware`가 기본값. HighDPI 멀티 모니터 시나리오를 요청받은 경우에만 `PerMonitorV2`. |

---


## 🎯 핵심 일반 WinForms 이슈: 두 가지 코드 컨텍스트 다루기

| 컨텍스트 | 파일/위치 | 언어 수준 | 핵심 규칙 |
|---------|----------------|----------------|----------|
| **디자이너 코드** | *.designer.cs*, `InitializeComponent` 내부 | 직렬화 중심 (C# 2.0 언어 기능 가정) | 단순하고, 예측 가능하고, 파싱 가능 |
| **일반 코드** | *.cs* 파일, 이벤트 핸들러, 비즈니스 로직 | 최신 C# 11-14 | 모든 최신 기능을 적극적으로 사용 |

**결정:** *.designer.cs* 또는 `InitializeComponent` 내부 → 디자이너 규칙. 그 외 → 최신 C# 규칙.

---

## 🚨 디자이너 파일 규칙 (최우선)

⚠️ 진단 오류와 빌드/컴파일 오류가 최종적으로 완전히 해결되었는지 확인하세요!

### ❌ InitializeComponent에서 금지

| 카테고리 | 금지 항목 | 이유 |
|----------|-----------|-----|
| 제어 흐름 | `if`, `for`, `foreach`, `while`, `goto`, `switch`, `try`/`catch`, `lock`, `await`, VB: `On Error`/`Resume` | 디자이너가 파싱할 수 없음 |
| 연산자 | `? :` (삼항), `??`/`?.`/`?[]` (null 병합/조건부), `nameof()` | 직렬화 형식에 없음 |
| 함수 | 람다, 로컬 함수, 컬렉션 표현식 (`...=[]` 또는 `...=[1,2,3]`) | 디자이너 파서를 깨뜨림 |
| 백킹 필드 | ControlCollections에는 클래스 필드 범위의 변수만 추가, 로컬 변수는 절대 안 됨! | 디자이너가 파싱할 수 없음 |

**허용되는 메서드 호출:** `SuspendLayout`, `ResumeLayout`, `BeginInit`, `EndInit` 같은 디자이너 지원 인터페이스 메서드

### ❌ *.designer.cs* 파일에서 금지

❌ 메서드 정의 (`InitializeComponent`, `Dispose` 제외, 기존 추가 생성자 보존)
❌ 속성
❌ 람다 표현식, `InitializeComponent`에서 이벤트를 람다에 바인딩하는 것도 금지!
❌ 복잡한 로직
❌ `??`/`?.`/`?[]` (null 병합/조건부), `nameof()`
❌ 컬렉션 표현식

### ✅ 올바른 패턴

✅ 파일 범위 네임스페이스 정의 (선호)

### 📋 InitializeComponent 메서드의 필수 구조

| 순서 | 단계 | 예시 |
|-------|------|---------|
| 1 | 컨트롤 인스턴스화 | `button1 = new Button();` |
| 2 | 컴포넌트 컨테이너 생성 | `components = new Container();` |
| 3 | 컨테이너 레이아웃 일시 중지 | `SuspendLayout();` |
| 4 | 컨트롤 설정 | 각 컨트롤의 속성 설정 |
| 5 | Form/UserControl을 마지막에 설정 | `ClientSize`, `Controls.Add()`, `Name` |
| 6 | 레이아웃 재개 | `ResumeLayout(false);` |
| 7 | EOF에 백킹 필드 | 마지막 메서드 뒤의 마지막 `#endregion` 뒤. | `_btnOK`, `_txtFirstname` - C# 범위는 `private`, VB 범위는 `Friend WithEvents` |

(컨트롤의 의미 있는 명명을 시도하고, 가능하면 기존 코드베이스에서 스타일을 도출하세요.)

```csharp
private void InitializeComponent()
{
    // 1. Instantiate
    _picDogPhoto = new PictureBox();
    _lblDogographerCredit = new Label();
    _btnAdopt = new Button();
    _btnMaybeLater = new Button();

    // 2. Components
    components = new Container();

    // 3. Suspend
    ((ISupportInitialize)_picDogPhoto).BeginInit();
    SuspendLayout();

    // 4. Configure controls
    _picDogPhoto.Location = new Point(12, 12);
    _picDogPhoto.Name = "_picDogPhoto";
    _picDogPhoto.Size = new Size(380, 285);
    _picDogPhoto.SizeMode = PictureBoxSizeMode.Zoom;
    _picDogPhoto.TabStop = false;

    _lblDogographerCredit.AutoSize = true;
    _lblDogographerCredit.Location = new Point(12, 300);
    _lblDogographerCredit.Name = "_lblDogographerCredit";
    _lblDogographerCredit.Size = new Size(200, 25);
    _lblDogographerCredit.Text = "Photo by: Professional Dogographer";

    _btnAdopt.Location = new Point(93, 340);
    _btnAdopt.Name = "_btnAdopt";
    _btnAdopt.Size = new Size(114, 68);
    _btnAdopt.Text = "Adopt!";

    // OK, BtnAdopt_Click이 메인 .cs 파일에 정의된 경우
    _btnAdopt.Click += BtnAdopt_Click;

    // 절대 안 됨, InitializeComponent에 람다가 있으면 안 됩니다!
    _btnAdopt.Click += (s, e) => Close();

    // 5. Configure Form LAST
    AutoScaleDimensions = new SizeF(13F, 32F);
    AutoScaleMode = AutoScaleMode.Font;
    ClientSize = new Size(420, 450);
    Controls.Add(_picDogPhoto);
    Controls.Add(_lblDogographerCredit);
    Controls.Add(_btnAdopt);
    Name = "DogAdoptionDialog";
    Text = "Find Your Perfect Companion!";
    ((ISupportInitialize)_picDogPhoto).EndInit();

    // 6. Resume
    ResumeLayout(false);
    PerformLayout();
}

#endregion

// 7. Backing fields at EOF

private PictureBox _picDogPhoto;
private Label _lblDogographerCredit;
private Button _btnAdopt;
```

**기억하세요:** 복잡한 UI 설정 로직은 메인 *.cs* 파일에 넣으세요, *.designer.cs*가 아닙니다.

---

---

## 최신 C# 기능 (일반 코드에만)

**`.cs` 파일(이벤트 핸들러, 비즈니스 로직)에만 적용하세요. `.designer.cs`나 `InitializeComponent`에는 절대 안 됩니다.**

### 스타일 가이드라인

| 카테고리 | 규칙 | 예시 |
|----------|------|---------|
| Using 지시문 | 전역 가정 | `System.Windows.Forms`, `System.Drawing`, `System.ComponentModel` |
| 기본 타입 | 타입 이름 | `int`, `string`, `Int32`나 `String`이 아님 |
| 인스턴스화 | 대상 타입 지정 | `Button button = new();` |
| `var`보다 타입 선호 | `var`는 명확하거나 이름이 긴 경우에만 | `var lookup = ReturnsDictOfStringAndListOfTuples()` // 타입 명확 |
| 이벤트 핸들러 | Nullable sender | `private void Handler(object? sender, EventArgs e)` |
| 이벤트 | Nullable | `public event EventHandler? MyEvent;` |
| 사소한 것 | `return`/코드 블록 전 빈 줄 | 앞에 빈 줄 선호 |
| `this` 한정자 | 피하기 | NetFX에서는 항상, 그 외에는 모호성 해소나 확장 메서드용 |
| 인수 검증 | 항상; .NET 8+에서는 throw 헬퍼 | `ArgumentNullException.ThrowIfNull(control);` |
| Using 문 | 최신 구문 | `using frmOptions modalOptionsDlg = new(); // 모달 Form은 항상 dispose!` |

### 속성 패턴 (⚠️ 중요 - 일반적인 버그 원인!)

| 패턴 | 동작 | 사용 사례 | 메모리 |
|---------|----------|----------|--------|
| `=> new Type()` | 매 접근마다 새 인스턴스 생성 | ⚠️ 메모리 누수 가능성! | 접근당 할당 |
| `{ get; } = new()` | 생성 시 한 번만 생성 | 용도: 캐시/상수 | 단일 할당 |
| `=> _field ?? Default` | 계산/동적 값 | 용도: 계산된 속성 | 다양 |

```csharp
// ❌ 잘못됨 - 메모리 누수
public Brush BackgroundBrush => new SolidBrush(BackColor);

// ✅ 올바름 - 캐시됨
public Brush BackgroundBrush { get; } = new SolidBrush(Color.White);

// ✅ 올바름 - 동적
public Font CurrentFont => _customFont ?? DefaultFont;
```

**의미적 차이를 이해하지 않고 하나를 다른 것으로 "리팩토링"하지 마세요!**

### If-Else 체인보다 Switch 표현식 선호

```csharp
// ✅ 새로운 방식: 수많은 IF 대신:
private Color GetStateColor(ControlState state) => state switch
{
    ControlState.Normal => SystemColors.Control,
    ControlState.Hover => SystemColors.ControlLight,
    ControlState.Pressed => SystemColors.ControlDark,
    _ => SystemColors.Control
};
```

### 이벤트 핸들러에서 패턴 매칭 선호

```csharp
// .NET 8+부터 nullable sender에 주의!
private void Button_Click(object? sender, EventArgs e)
{
    if (sender is not Button button || button.Tag is null)
        return;

    // 여기서 button 사용
}
```

## 처음부터 Form/UserControl 설계 시

### 파일 구조

| 언어 | 파일 | 상속 |
|----------|-------|-------------|
| C# | `FormName.cs` + `FormName.Designer.cs` | `Form` 또는 `UserControl` |
| VB.NET | `FormName.vb` + `FormName.Designer.vb` | `Form` 또는 `UserControl` |

**메인 파일:** 로직과 이벤트 핸들러
**디자이너 파일:** 인프라, 생성자, `Dispose`, `InitializeComponent`, 컨트롤 정의

### C# 규칙

- 파일 범위 네임스페이스
- 전역 using 지시문 가정
- 메인 Form/UserControl 파일에서 NRT 허용; 코드 비하인드 `.designer.cs`에서는 금지
- 이벤트 _핸들러_: `object? sender`
- 이벤트: nullable (`EventHandler?`)

### VB.NET 규칙

- Application Framework 사용. `Program.vb`는 없음.
- Forms/UserControls: 기본적으로 생성자 없음 (컴파일러가 `InitializeComponent()` 호출과 함께 생성)
- 생성자가 필요하면 `InitializeComponent()` 호출 포함
- 중요: 컨트롤 백킹 필드에 `Friend WithEvents controlName as ControlType`.
- `InitializeComponent` 파일의 `AddHandler`보다 메인 코드에서 `Handles` 절이 있는 이벤트 핸들러 `Sub`를 강력히 선호

---

## 클래식 데이터 바인딩과 MVVM 데이터 바인딩 (.NET 8+)

### 호환성 변경: .NET Framework vs .NET 8+

| 기능 | .NET Framework <= 4.8.1 | .NET 8+ |
|---------|----------------------|---------|
| 타입 DataSets | 디자이너 지원 | 코드 전용 (권장하지 않음) |
| 객체 바인딩 | 지원 | 향상된 UI, 완전 지원 |
| 데이터 소스 창 | 사용 가능 | 사용 불가 |

### 데이터 바인딩 규칙

- 객체 DataSources: `INotifyPropertyChanged`, `BindingList<T>` 필요, MVVM CommunityToolkit의 `ObservableObject` 선호.
- `ObservableCollection<T>`: 두 변경 알림 접근 방식을 병합하는 전용 어댑터인 `BindingList<T>`가 필요. 존재하지 않으면 생성.
- 소스 방향 단방향: WinForms DataBinding에서 지원되지 않음 (해결 방법: NO-OP 속성 setter가 있는 추가 전용 VM 속성).

### 솔루션에 객체 DataSource 추가, ViewModel도 DataSource로 취급

디자이너에서 타입을 DataSource로 접근 가능하게 하려면 `Properties\DataSources\`에 `.datasource` 파일을 생성하세요:

```xml
<?xml version="1.0" encoding="utf-8"?>
<GenericObjectDataSource DisplayName="MainViewModel" Version="1.0"
    xmlns="urn:schemas-microsoft-com:xml-msdatasource">
  <TypeInfo>MyApp.ViewModels.MainViewModel, MyApp.ViewModels, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null</TypeInfo>
</GenericObjectDataSource>
```

이후 Forms/UserControls에서 BindingSource 컴포넌트를 사용하여 View와 ViewModel 사이의 "중재자" 인스턴스로 DataSource 타입에 바인딩하세요. (클래식 WinForms 바인딩 접근 방식)

### .NET 8+의 새로운 MVVM 명령 바인딩 API

| API | 설명 | 캐스케이딩 |
|-----|-------------|-----------|
| `Control.DataContext` | MVVM을 위한 앰비언트 속성 | 예 (계층 아래로) |
| `ButtonBase.Command` | ICommand 바인딩 | 아니오 |
| `ToolStripItem.Command` | ICommand 바인딩 | 아니오 |
| `*.CommandParameter` | 명령에 자동 전달 | 아니오 |

**참고:** `ToolStripItem`은 이제 `BindableComponent`에서 파생됩니다.

### WinForms에서의 MVVM 패턴 (.NET 8+)

- WinForms 프로젝트를 MVVM으로 생성하거나 리팩토링하라는 요청을 받으면, MVVM CommunityToolkit 기반의 ViewModel 전용 클래스 라이브러리를 식별(이미 존재하는 경우)하거나 생성하세요
- WinForms 프로젝트에서 MVVM ViewModel 클래스 라이브러리 참조
- 위에서 설명한 대로 객체 DataSources를 통해 ViewModel 가져오기
- 중첩된 Form/UserControl 시나리오에서 컨트롤 계층 아래로 ViewModel을 데이터 소스로 전달하기 위해 새로운 `Control.DataContext` 사용
- MVVM 명령 바인딩에 `Button[Base].Command` 또는 `ToolStripItem.Command` 사용. 매개변수 전달에는 CommandParameter 속성 사용.

- - 필요한 경우 사용자 정의 데이터 변환을 위해 `Binding` 객체의 `Parse` 및 `Format` 이벤트 사용 (`IValueConverter` 해결 방법).

```csharp
private void PrincipleApproachForIValueConverterWorkaround()
{
   // InitializeComponent에서 바인딩이 완료되었다고 가정하고
   // 바인딩된 속성을 다음과 같이 조회합니다:
   Binding b = text1.DataBindings["Text"];

   // "IValueConverter" 기능을 다음과 같이 연결합니다:
   b.Format += new ConvertEventHandler(DecimalToCurrencyString);
   b.Parse += new ConvertEventHandler(CurrencyStringToDecimal);
}
```
- 평소처럼 속성을 바인딩하세요.
- 명령도 같은 방식으로 바인딩하세요 - ViewModel은 데이터 소스입니다! 다음과 같이 하세요:
```csharp
// Create BindingSource
components = new Container();
mainViewModelBindingSource = new BindingSource(components);

// Before SuspendLayout
mainViewModelBindingSource.DataSource = typeof(MyApp.ViewModels.MainViewModel);

// Bind properties
_txtDataField.DataBindings.Add(new Binding("Text", mainViewModelBindingSource, "PropertyName", true));

// Bind commands
_tsmFile.DataBindings.Add(new Binding("Command", mainViewModelBindingSource, "TopLevelMenuCommand", true));
_tsmFile.CommandParameter = "File";
```

---

## WinForms Async Patterns (.NET 9+)

### Control.InvokeAsync Overload Selection

| Your Code Type | Overload | Example Scenario |
|----------------|----------|------------------|
| Sync action, no return | `InvokeAsync(Action)` | Update `label.Text` |
| Async operation, no return | `InvokeAsync(Func<CT, ValueTask>)` | Load data + update UI |
| Sync function, returns T | `InvokeAsync<T>(Func<T>)` | Get control value |
| Async operation, returns T | `InvokeAsync<T>(Func<CT, ValueTask<T>>)` | Async work + result |

### ⚠️ Fire-and-Forget Trap

```csharp
// ❌ WRONG - Analyzer violation, fire-and-forget
await InvokeAsync<string>(() => await LoadDataAsync());

// ✅ CORRECT - Use async overload
await InvokeAsync<string>(async (ct) => await LoadDataAsync(ct), outerCancellationToken);
```

### Form Async Methods (.NET 9+)

- `ShowAsync()`: Completes when form closes.
  Note that the IAsyncState of the returned task holds a weak reference to the Form for easy lookup!
- `ShowDialogAsync()`: Modal with dedicated message queue

### CRITICAL: Async EventHandler Pattern

- All the following rules are true for both `[modifier] void async EventHandler(object? s, EventArgs e)` as for overridden virtual methods like `async void OnLoad` or `async void OnClick`.
- `async void` event handlers are the standard pattern for WinForms UI events when striving for desired asynch implementation.
- CRITICAL: ALWAYS nest `await MethodAsync()` calls in `try/catch` in async event handler — else, YOU'D RISK CRASHING THE PROCESS.

## Exception Handling in WinForms

### Application-Level Exception Handling

WinForms provides two primary mechanisms for handling unhandled exceptions:

**AppDomain.CurrentDomain.UnhandledException:**
- Catches exceptions from any thread in the AppDomain
- Cannot prevent application termination
- Use for logging critical errors before shutdown

**Application.ThreadException:**
- Catches exceptions on the UI thread only
- Can prevent application crash by handling the exception
- Use for graceful error recovery in UI operations

### Exception Dispatch in Async/Await Context

When preserving stack traces while re-throwing exceptions in async contexts:

```csharp
try
{
    await SomeAsyncOperation();
}
catch (Exception ex)
{
    if (ex is OperationCanceledException)
    {
        // Handle cancellation
    }
    else
    {
        ExceptionDispatchInfo.Capture(ex).Throw();
    }
}
```

**Important Notes:**
- `Application.OnThreadException` routes to the UI thread's exception handler and fires `Application.ThreadException`.
- Never call it from background threads — marshal to UI thread first.
- For process termination on unhandled exceptions, use `Application.SetUnhandledExceptionMode(UnhandledExceptionMode.ThrowException)` at startup.
- **VB Limitation:** VB cannot await in catch block. Avoid, or work around with state machine pattern.

## CRITICAL: Manage CodeDOM Serialization

Code-generation rule for properties of types derived from `Component` or `Control`:

| Approach | Attribute | Use Case | Example |
|----------|-----------|----------|---------|
| Default value | `[DefaultValue]` | Simple types, no serialization if matches default | `[DefaultValue(typeof(Color), "Yellow")]` |
| Hidden | `[DesignerSerializationVisibility.Hidden]` | Runtime-only data | Collections, calculated properties |
| Conditional | `ShouldSerialize*()` + `Reset*()` | Complex conditions | Custom fonts, optional settings |

```csharp
public class CustomControl : Control
{
    private Font? _customFont;

    // Simple default - no serialization if default
    [DefaultValue(typeof(Color), "Yellow")]
    public Color HighlightColor { get; set; } = Color.Yellow;

    // Hidden - never serialize
    [DesignerSerializationVisibility(DesignerSerializationVisibility.Hidden)]
    public List<string> RuntimeData { get; set; }

    // Conditional serialization
    public Font? CustomFont
    {
        get => _customFont ?? Font;
        set { /* setter logic */ }
    }

    private bool ShouldSerializeCustomFont()
        => _customFont is not null && _customFont.Size != 9.0f;

    private void ResetCustomFont()
        => _customFont = null;
}
```

**Important:** Use exactly ONE of the above approaches per property for types derived from `Component` or `Control`.

---

## WinForms Design Principles

### Core Rules

**Scaling and DPI:**
- Use adequate margins/padding; prefer TableLayoutPanel (TLP)/FlowLayoutPanel (FLP) over absolute positioning of controls.
- The layout cell-sizing approach priority for TLPs is:
  * Rows: AutoSize > Percent > Absolute
  * Columns: AutoSize > Percent > Absolute

- For newly added Forms/UserControls: Assume 96 DPI/100% for `AutoScaleMode` and scaling
- For existing Forms: Leave AutoScaleMode setting as-is, but take scaling for coordinate-related properties into account

- Be DarkMode-aware in .NET 9+ - Query current DarkMode status: `Application.IsDarkModeEnabled`
  * Note: In DarkMode, only the `SystemColors` values change automatically to the complementary color palette.

- Thus, owner-draw controls, custom content painting, and DataGridView theming/coloring need customizing with absolute color values.

### Layout Strategy

**Divide and conquer:**
- Use multiple or nested TLPs for logical sections - don't cram everything into one mega-grid.
- Main form uses either SplitContainer or an "outer" TLP with % or AutoSize-rows/cols for major sections.
- Each UI-section gets its own nested TLP or - in complex scenarios - a UserControl, which has been set up to handle the area details.

**Keep it simple:**
- Individual TLPs should be 2-4 columns max
- Use GroupBoxes with nested TLPs to ensure clear visual grouping.
- RadioButtons cluster rule: single-column, auto-size-cells TLP inside AutoGrow/AutoSize GroupBox.
- Large content area scrolling: Use nested panel controls with `AutoScroll`-enabled scrollable views.

**Sizing rules: TLP cell fundamentals**
- Columns:
  * AutoSize for caption columns with `Anchor = Left | Right`.
  * Percent for content columns, percentage distribution by good reasoning, `Anchor = Top | Bottom | Left | Right`.
    Never dock cells, always anchor!
  * Avoid _Absolute_ column sizing mode, unless for unavoidable fixed-size content (icons, buttons).
- Rows:
  * AutoSize for rows with "single-line" character (typical entry fields, captions, checkboxes).
  * Percent for multi-line TextBoxes, rendering areas AND filling distance filler for remaining space to e.g., a bottom button row (OK|Cancel).
  * Avoid _Absolute_ row sizing mode even more.

- Margins matter: Set `Margin` on controls (min. default 3px).
- Note: `Padding` does not have an effect in TLP cells.

### Common Layout Patterns

#### Single-line TextBox (2-column TLP)
**Most common data entry pattern:**
- Label column: AutoSize width
- TextBox column: 100% Percent width
- Label: `Anchor = Left | Right` (vertically centers with TextBox)
- TextBox: `Dock = Fill`, set `Margin` (e.g., 3px all sides)

#### Multi-line TextBox or Larger Custom Content - Option A (2-column TLP)
- Label in same row, `Anchor = Top | Left`
- TextBox: `Dock = Fill`, set `Margin`
- Row height: AutoSize or Percent to size the cell (cell sizes the TextBox)

#### Multi-line TextBox or Larger Custom Content - Option B (1-column TLP, separate rows)
- Label in dedicated row above TextBox
- Label: `Dock = Fill` or `Anchor = Left`
- TextBox in next row: `Dock = Fill`, set `Margin`
- TextBox row: AutoSize or Percent to size the cell

**Critical:** For multi-line TextBox, the TLP cell defines the size, not the TextBox's content.

### Container Sizing (CRITICAL - Prevents Clipping)

**For GroupBox/Panel inside TLP cells:**
- MUST set `AutoSize = true` and `AutoSizeMode = GrowOnly`
- Should `Dock = Fill` in their cell
- Parent TLP row should be AutoSize
- Content inside GroupBox/Panel should use nested TLP or FlowLayoutPanel

**Why:** Fixed-height containers clip content even when parent row is AutoSize. The container reports its fixed size, breaking the sizing chain.

### Modal Dialog Button Placement

**Pattern A - Bottom-right buttons (standard for OK/Cancel):**
- Place buttons in FlowLayoutPanel: `FlowDirection = RightToLeft`
- Keep additional Percentage Filler-Row between buttons and content.
- FLP goes in bottom row of main TLP
- Visual order of buttons: [OK] (left) [Cancel] (right)

**Pattern B - Top-right stacked buttons (wizards/browsers):**
- Place buttons in FlowLayoutPanel: `FlowDirection = TopDown`
- FLP in dedicated rightmost column of main TLP
- Column: AutoSize
- FLP: `Anchor = Top | Right`
- Order: [OK] above [Cancel]

**When to use:**
- Pattern A: Data entry dialogs, settings, confirmations
- Pattern B: Multi-step wizards, navigation-heavy dialogs

### Complex Layouts

- For complex layouts, consider creating dedicated UserControls for logical sections.
- Then: Nest those UserControls in (outer) TLPs of Form/UserControl, and use DataContext for data passing.
- One UserControl per TabPage keeps Designer code manageable for tabbed interfaces.

### Modal Dialogs

| Aspect | Rule |
|--------|------|
| Dialog buttons | Order -> Primary (OK): `AcceptButton`, `DialogResult = OK` / Secondary (Cancel): `CancelButton`, `DialogResult = Cancel` |
| Close strategy | `DialogResult` gets applied by DialogResult implicitly, no need for additional code |
| Validation | Perform on _Form_, not on Field scope. Never block focus-change with `CancelEventArgs.Cancel = true` |

Use `DataContext` property (.NET 8+) of Form to pass and return modal data objects.

### Layout Recipes

| Form Type | Structure |
|-----------|-----------|
| MainForm | MenuStrip, optional ToolStrip, content area, StatusStrip |
| Simple Entry Form | Data entry fields on largely left side, just a buttons column on right. Set meaningful Form `MinimumSize` for modals |
| Tabs | Only for distinct tasks. Keep minimal count, short tab labels |

### Accessibility

- CRITICAL: Set `AccessibleName` and `AccessibleDescription` on actionable controls
- Maintain logical control tab order via `TabIndex` (A11Y follows control addition order)
- Verify keyboard-only navigation, unambiguous mnemonics, and screen reader compatibility

### TreeView and ListView

| Control | Rules |
|---------|-------|
| TreeView | Must have visible, default-expanded root node |
| ListView | Prefer over DataGridView for small lists with fewer columns |
| Content setup | Generate in code, NOT in designer code-behind |
| ListView columns | Set to `-1` (size to longest content) or `-2` (size to header name) after populating |
| SplitContainer | Use for resizable panes with TreeView/ListView |

### DataGridView

- Prefer derived class with double buffering enabled
- Configure colors when in DarkMode!
- Large data: page/virtualize (`VirtualMode = True` with `CellValueNeeded`)

### Resources and Localization

- String literal constants for UI display NEED to be in resource files.
- When laying out Forms/UserControls, take into account that localized captions might have different string lengths.
- Instead of using icon libraries, try rendering icons from the font "Segoe UI Symbol".
- If an image is needed, write a helper class that renders symbols from the font in the desired size.

## Critical Reminders

| # | Rule |
|---|------|
| 1 | `InitializeComponent` code serves as serialization format - more like XML, not C# |
| 2 | Two contexts, two rule sets - designer code-behind vs regular code |
| 3 | Validate form/control names before generating code |
| 4 | Stick to coding style rules for `InitializeComponent` |
| 5 | Designer files never use NRT annotations |
| 6 | Modern C# features for regular code ONLY |
| 7 | Data binding: Treat ViewModels as DataSources, remember `Command` and `CommandParameter` properties |
