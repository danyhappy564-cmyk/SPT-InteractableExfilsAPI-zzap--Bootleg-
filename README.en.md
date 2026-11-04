<p align="center">
  <a href="README.md"><img alt="한국어" src="https://img.shields.io/badge/%ED%95%9C%EA%B5%AD%EC%96%B4-%ED%95%9C%EA%B5%AD%EC%96%B4%EB%A1%9C%20%EB%B3%B4%EA%B8%B0-6e7681?style=for-the-badge"></a>
  <a href="README.en.md"><img alt="English (this page)" src="https://img.shields.io/badge/English-This%20page-1f6feb?style=for-the-badge"></a>
</p>

### ⚠️ IMPORTANT NOTICE / DISCLAIMER

- **Original Author:** Jehree
- **Original Repository:** SPT-InteractableExfilsAPI
- **Original Link:** https://github.com/Jehree/SPT-InteractableExfilsAPI
- **License:** See upstream repository
- **This Port By:** R_F (danyhappy564-cmyk) — unofficial, AI-assisted port. Not affiliated with or endorsed by the original author.

1. **Reflection & Take-Downs:** I deeply reflect on the ECOT incident. As an AI-assisted "vibe coder," I will immediately delete files if the original authors ask.
2. **No Re-Distribution:** These ported builds are unverified, temporary fixes. Please do NOT re-upload or share them anywhere else.
3. **Do Not Pester Original Authors:** Never report bugs or pester original modders regarding issues from my unofficial ports.
4. **Full Credit & Respect:** I will always credit original creators on GitHub and prioritize their decisions above all else.
5. **Support Original Creators:** Instead of using my ports, please visit the original authors' Forge pages to leave kind words or tips.

---

## Changelog

The full changelog (1 entry) is in the [`changelog/`](changelog/) folder, 10 per file (higher number = newer, latest: [`CHANGELOG-001.md`](changelog/CHANGELOG-001.md)). Per-release summaries: [Releases](../../releases).

# Interactable Exfils API (fork)

> **Original author · Original repository**
> **Jehree** — https://github.com/Jehree/SPT-InteractableExfilsAPI
>
> This is a **fork** of the project above. Features are unchanged; it was only **ported
> to build and run on SPT 4.1**. At the fork point (`2093b56` = upstream `v2.1.0`, 4.0),
> its code was identical to upstream.

Currently targets **SPT 4.1**.

---

## Changes in the 4.1 port

Version 4.1 distributes a **deobfuscated** client. All five types this mod referenced
by name regained real names and namespaces.

| 4.0 | 4.1 | Used in |
|---|---|---|
| `ActionsReturnClass` | `EFT.UI.AvailableInteractionState` | CustomExfilTrigger, GetAvailableActionsPatch, InteractableExfilsService |
| `ActionsTypesClass` | `EFT.UI.InteractionAction` | CustomExfilAction, CustomExfilTrigger, GetAvailableActionsPatch |
| `GetActionsClass` | `EFT.InteractionContextHelper` | GetAvailableActionsPatch |
| `BindableStateClass<T>` | `Diz.Binding.BindableState<T>` | InteractableExfilsService |
| `NotificationManagerClass` | `EFT.Communications.NotificationManager` | CustomExfilTrigger, Examples |

In 4.0 all were in the **global namespace** and needed no `using`. Now they have their
own namespaces, so `EFT.UI` / `EFT.Communications` / `Diz.Binding` imports
were added per file.

All 364 source identifiers were compared with the SPT 4.1 wiki's 4.0→4.1 mapping table.
Only those five matched (the rest already had real names and remain unchanged).

### Cross-checked against actual logs, beyond the mapping table

SPT 4.1.5 client BepInEx logs include HarmonyX IL dumps of patch targets.
Actual 4.1 signatures were checked directly there:

```
EFT.InteractionContextHelper::GetAvailableActions(EFT.GamePlayerOwner owner, EFT.IInteractive interactive)
EFT.InteractionContextHelper::GetAvailableActions(EFT.GamePlayerOwner, EFT.Interactive.ExfiltrationPoint)
EFT.InteractionContextHelper::GetAvailableActions(EFT.GamePlayerOwner, EFT.Interactive.Switch)
EFT.UI.AvailableInteractionState::Actions / Error / SelectedAction / InitSelected() / SelectNextAction()
EFT.UI.InteractionAction::Name / Action / Disabled
EFT.Communications.NotificationManager::DisplayWarningNotification(System.String, ...)
Diz.Binding.BindableState
```

All members used by this mod remain present. The first argument to `GetAvailableActions`
is still named `owner` in 4.1 (`GetAvailableActionsPatch` locates its target using
that name).

---

## Build configuration was also rebuilt

- `net471` → `netstandard2.1` (SPT 4.1 client-plugin target)
- **`PathToSPT` was `..\..`**, resolving references only with the repository exactly two
  levels inside an SPT installation. Cloning elsewhere failed. Replaced with `SptRoot`
  (default `E:\SPT 4.1`, overridden by `-p:SptRoot=...` or environment variable).
  Existing `PathToSPT` arguments still work as an alias
- If `SptRoot` is not an SPT installation, fails with **one explanatory error**
  instead of dozens of missing-type messages
- Post-build copying used an `Exec` call to a `cmd` one-liner, **reporting success
  while doing nothing outside Windows**. Replaced with MSBuild `Copy`

## Build

```
dotnet build InteractableExfilsAPI.csproj -c Release
dotnet build InteractableExfilsAPI.csproj -c Release -p:"SptRoot=D:\내 SPT 경로"
```

Building copies the dll + LICENSE to
`$(SptRoot)\BepInEx\plugins\InteractableExfilsAPI\`.

---

## Verified / Not verified

| | Status |
|---|---|
| Five type mappings | **Verified** — wiki 4.0→4.1 table + IL dumps from actual 4.1 client logs |
| Required members exist in 4.1 | **Verified** (IL dumps above) |
| Full compilation against 4.1-shaped assemblies | **Passed** — built against a 4.0 `Assembly-CSharp` DLL with the five renames actually applied via Cecil |
| Compilation against real 4.1 `Assembly-CSharp.dll` | **Not possible** — no 4.1 client assemblies in this environment |
| In-game verification | **Not done** |

### Troubleshooting

`GetAvailableActionsPatch` uses `AccessTools.FirstMethod(...)` to find the first overload
whose first argument is named `owner`. In 4.1, `InteractionContextHelper` has over 20
matching overloads; if reflection returns a different order from 4.0, it may **patch
the wrong overload and silently do nothing**. This is unchanged upstream behavior,
but is the first suspect if extract interactions do not appear at all.

---

## API Usage

### Initial setup in your project
The only thing you need to do to start development with Interactable Exfils API is to reference the dll in your `.csproj` file: 

```xml
<ItemGroup>
    <Reference Include="InteractableExfilsAPI">
        <HintPath>$(PathToSPT)\BepInEx\plugins\InteractableExfilsAPI.dll</HintPath>
    </Reference>
</ItemGroup>
```

Then you have to create at least one handler and register it with the instance of the `InteractableExfilsService`

### Create your custom handler

```cs
// This example will add an enabled static action to every single extract in the game
public static class Examples
{
    // this static function is an exfil actions handler you can register with the InteractableExfilsService
    public static OnActionsAppliedResult SimpleExample(ExfiltrationPoint exfil, CustomExfilTrigger customExfilTrigger, bool exfilIsAvailableToPlayer)
    {
        // This part of the code is ran everytime the prompt is created/refreshed
        // It occurs when:
        // 1. the player enter the exfil zone
        // 2. the player interact with the prompt (i.e. press the "F" key)
        // 3. the player changed a BepInEx config in InteractableExfilsAPI
        // 4. the customExfilTrigger.RefreshPrompt() method has been invoked
        // 5. the InteractableExfilsService.RefreshPrompt() method has been invoked

        bool isDisabled = false;

        // This represent the definition of 1 prompt item
        CustomExfilAction customExfilAction = new CustomExfilAction(
            "Example Interaction",
            isDisabled,
            () => {
                // This part of the code is ran when the player interact with this prompt item
                NotificationManagerClass.DisplayMessageNotification("Simple Interaction Example Selected!");
            }
        );

        // Here you have control over the ordering of the actions
        List<CustomExfilAction> actions = [customExfilAction];

        return new OnActionsAppliedResult(actions);
    }
}
```

Take a look to the [Examples class](./Examples.cs) for more.

### Register your actions

You can do this whenever you want but the recommended way for doing it as early as possible is in the `Start()` method of your plugin class

```cs
public class Plugin : BaseUnityPlugin {
    private void Awake() {
        // enable your patches here
    }
    private void Start() {
        // retrieve the interactable exfil singleton service
        InteractableExfilsService ieService = InteractableExfilsService.Instance();

        // register SimpleExample handler
        ieService.OnActionsAppliedEvent += Examples.SimpleExample;
    }
}
```

### Disable vanilla actions
If you don't want to let Interactable Exfils API show the car exfils and labs elevator exfils prompts, it's possible to disable them. Be aware that in this situation your mod should handle the extraction logic by itself otherwise the player couldn't extract.

```cs
public static void DisableVanillaAction()
{
    // e.g. disable vanilla action (for cars and labs elevator)
    InteractableExfilsService.Instance().DisableVanillaActions = true;
}
```

### Retrieve all active exfils

If you need a list of all the active exfils in raid, you can get it via the `InteractableExfilsSession` component

```cs
public static List<ExfiltrationPoint> ExampleGetExfils() {
    InteractableExfilsSession session = InteractableExfilsService.GetSession();
    return session.ActiveExfils;
}
```

## Mods that use Interactable Exfils API
If your mod use Interactable Exfils API, please make a PR here so we can point it as an example.

- [Path To Tarkov](https://hub.sp-tarkov.com/files/file/569-path-to-tarkov/): used in [PTT.Services.ExfilPromptService](https://github.com/guillaumearm/PathToTarkov/blob/cc5a24140ae3acd9e212b9e73729e42b77780a7d/PTT-Plugin/Services/ExfilPromptService.cs)

