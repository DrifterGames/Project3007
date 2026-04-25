# Custom widgets

VNFramework expects every UI widget to be a `UCommonActivatableWidget`
subclass. There are three places to slot in:

| Subclass | When |
|---|---|
| `UVNBaseWidget` | Lightweight overlay — needs subsystem access + back action, doesn't care about DPI scaling or theme |
| `UVNLayoutContainer` | Custom panel — needs DPI scaling, safe zones, theme cache, but doesn't render a 9-slice frame |
| `UVNStyledFrame` | Themed panel with background + border + ornaments — the default for dialogue, choices, menus |

For buttons or text the styled element family handles theming; subclass
those directly.

## Pattern: `UVNBaseWidget` for a HUD overlay

`UVNBaseWidget` (in `Plugins/VNFramework/Source/VNUI/Public/Widgets/VNBaseWidget.h`)
holds a `UVNSubsystem` reference and exposes Blueprint-implementable
events for activation/back.

```cpp
// MyAffectionMeterWidget.h
#pragma once
#include "Widgets/VNBaseWidget.h"
#include "MyAffectionMeterWidget.generated.h"

class UProgressBar;

UCLASS(Blueprintable)
class MYGAME_API UMyAffectionMeterWidget : public UVNBaseWidget
{
    GENERATED_BODY()

public:
    virtual void InitializeWidget(UVNSubsystem* InSubsystem) override;

protected:
    UPROPERTY(meta=(BindWidget))
    TObjectPtr<UProgressBar> AffectionBar;

    UFUNCTION()
    void HandleDialogueAdvanced(const FVNDialogueLine& Line, int32 LineIndex);
};
```

```cpp
// MyAffectionMeterWidget.cpp
#include "MyAffectionMeterWidget.h"
#include "Components/ProgressBar.h"
#include "Subsystem/VNSubsystem.h"

void UMyAffectionMeterWidget::InitializeWidget(UVNSubsystem* InSubsystem)
{
    Super::InitializeWidget(InSubsystem);
    if (VNSubsystem)
    {
        VNSubsystem->OnDialogueAdvanced.AddDynamic(this,
            &UMyAffectionMeterWidget::HandleDialogueAdvanced);
    }
}

void UMyAffectionMeterWidget::HandleDialogueAdvanced(
    const FVNDialogueLine& /*Line*/, int32 /*LineIndex*/)
{
    if (!AffectionBar || !VNSubsystem) return;
    const float Affection = VNSubsystem->GetFloat(
        EVNVariableScope::Story, TEXT("affection"), 0.0f);
    AffectionBar->SetPercent(FMath::Clamp(Affection / 100.0f, 0.0f, 1.0f));
}
```

In a Blueprint Widget child set the `BindWidget` named `AffectionBar`.
In CommonUI mode, push the widget through `UVNUIManagerSubsystem`:

```cpp
UIManager->PushWidgetToGameLayer(MyAffectionMeterWidgetClass);
```

## Pattern: `UVNStyledFrame` for a themed panel

Every framed widget in VNFramework (dialogue, choices, backlog, settings,
quick menu, save/load, main menu) extends `UVNStyledFrame`. Subclassing
gets you DPI scaling, safe zones, layered frame rendering, and automatic
theme reapply on chapter/runtime theme changes — for free.

```cpp
// MyJournalWidget.h
#pragma once
#include "Elements/VNStyledFrame.h"
#include "MyJournalWidget.generated.h"

class UScrollBox;

UCLASS(Blueprintable)
class MYGAME_API UMyJournalWidget : public UVNStyledFrame
{
    GENERATED_BODY()

public:
    UMyJournalWidget(const FObjectInitializer& ObjectInitializer
        = FObjectInitializer::Get());

    virtual void InitializeContainer(UVNSubsystem* InSubsystem) override;
    virtual void ApplyTheme() override;

protected:
    UPROPERTY(meta=(BindWidget))
    TObjectPtr<UScrollBox> EntryList;

    void RefreshEntries();
};
```

```cpp
// MyJournalWidget.cpp
UMyJournalWidget::UMyJournalWidget(const FObjectInitializer& OI)
    : Super(OI)
{
    FrameType = EVNFrameType::Panel; // pulls PanelFrame from FramePack
    bShowCornerOrnaments = true;
}

void UMyJournalWidget::InitializeContainer(UVNSubsystem* InSubsystem)
{
    Super::InitializeContainer(InSubsystem); // wires subsystem + theme
    RefreshEntries();
    if (VNSubsystem)
    {
        VNSubsystem->OnSceneChanged.AddDynamic(this,
            &UMyJournalWidget::RefreshEntries);
    }
}

void UMyJournalWidget::ApplyTheme()
{
    Super::ApplyTheme(); // applies frame brushes from FramePack

    // Custom theme behavior: re-tint the EntryList background, restyle
    // child rows, etc.
}
```

!!! info "BindWidget visibility"
    `umg_widget` (the VNMCP authoring tool) defaults `is_variable=false`
    on most slots. For any `meta=(BindWidget*)` C++ property to bind,
    its child Blueprint widget must have `bIsVariable=true`. Set it in
    the widget designer or pass `is_variable=true` to `umg_widget(action="add", …)`.

## Pattern: extending `UVNStyledButton`

`UVNStyledButton` (in `Plugins/VNFramework/Source/VNUI/Public/Elements/VNStyledButton.h`)
already extends `UCommonButtonBase`, hooks hover/click sounds, and
applies theme brushes. You rarely need to subclass it in C++ — usually
you just create a Blueprint child:

1. Right-click in Content Browser → *User Interface → Widget Blueprint*.
2. Set parent to `VNStyledButton`.
3. In Class Defaults, set `Button Type = Choice` (or `Menu`/`Icon`).
4. Set `Click Sound` / `Hover Sound`.
5. (Optional) Set `Style` to a `UVNThemedButtonStyle` subclass so the
   button picks up FramePack brushes automatically.

If you need extra C++ behavior (e.g. a button that exposes a tooltip
fed from a `FVNVariable`), subclass it:

```cpp
UCLASS(Blueprintable)
class MYGAME_API UMyTooltipButton : public UVNStyledButton
{
    GENERATED_BODY()

public:
    virtual void NativeOnHovered() override
    {
        Super::NativeOnHovered(); // play hover sound + update brush

        if (UVNSubsystem* Sub = GetGameInstance()->GetSubsystem<UVNSubsystem>())
        {
            const FString Text = Sub->GetString(
                EVNVariableScope::Story, TooltipVarName, TEXT(""));
            ShowTooltip(FText::FromString(Text));
        }
    }

protected:
    UPROPERTY(EditAnywhere, Category="Tooltip")
    FString TooltipVarName;

    void ShowTooltip(const FText& InText);
};
```

## Pattern: subclassing `UCommonButtonBase` directly

`UVNStyledButton` is the recommended path — it derives from
`UCommonButtonBase` and adds theme + sound integration. Only roll your
own `UCommonButtonBase` subclass if your button doesn't fit the
hover/pressed/disabled brush model (e.g. a radial menu segment).

```cpp
UCLASS(Blueprintable)
class MYGAME_API UMyRadialButton : public UCommonButtonBase
{
    GENERATED_BODY()
    // Your radial-specific overrides go here.
};
```

## Where to wire it up

| Widget kind | Wiring |
|---|---|
| Replaces dialogue widget | Set `AVNGameModeBase::DialogueWidgetClass` to your subclass |
| Replaces choice widget | `AVNGameModeBase::ChoiceWidgetClass` |
| New menu (settings, backlog, etc.) | `AVNGameModeBase::SettingsWidgetClass` (etc.) — these are `TSoftClassPtr`, accept BP children |
| Runtime overlay | `UVNUIManagerSubsystem::PushWidgetToGameLayer` (CommonUI) or add as child of HUD widget (legacy) |

For the broader design context see
[concept: dialogue + choices](../../designers/concepts/data-assets.md)
and [Theme pack reference](../../designers/reference/theme-pack.md).
