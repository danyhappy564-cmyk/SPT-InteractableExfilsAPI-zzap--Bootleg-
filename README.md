### ⚠️ IMPORTANT NOTICE / DISCLAIMER

**Original Author:** Jehree
**Original Repository:** SPT-InteractableExfilsAPI
**Original Link:** https://github.com/Jehree/SPT-InteractableExfilsAPI
**License:** See upstream repository
**This Port By:** R_F (danyhappy564-cmyk) — unofficial, AI-assisted port. Not affiliated with or endorsed by the original author.

1. **Reflection & Take-Downs:** I deeply reflect on the ECOT incident. As an AI-assisted "vibe coder," I will immediately delete files if the original authors ask.
2. **No Re-Distribution:** These ported builds are unverified, temporary fixes. Please do NOT re-upload or share them anywhere else.
3. **Do Not Pester Original Authors:** Never report bugs or pester original modders regarding issues from my unofficial ports.
4. **Full Credit & Respect:** I will always credit original creators on GitHub and prioritize their decisions above all else.
5. **Support Original Creators:** Instead of using my ports, please visit the original authors' Forge pages to leave kind words or tips.

---

# Interactable Exfils API (fork)

> **원작자 · 원본 레포**
> **Jehree** — https://github.com/Jehree/SPT-InteractableExfilsAPI
>
> 이 레포는 위 원작의 **포크**입니다. 기능은 그대로고, **SPT 4.1에서 빌드·동작하도록
> 포팅**한 것이 전부입니다. 포크 시점(`2093b56` = 원작 `v2.1.0`, 4.0)에서 원본과 코드
> 차이는 없었습니다.

현재 기준 **SPT 4.1**.

---

## 4.1 포팅에서 바뀐 것

4.1은 클라이언트를 **역난독화**해서 배포합니다. 이 모드가 이름으로 잡고 있던 타입
5개가 전부 실명 + 네임스페이스를 되찾았습니다.

| 4.0 | 4.1 | 쓰인 곳 |
|---|---|---|
| `ActionsReturnClass` | `EFT.UI.AvailableInteractionState` | CustomExfilTrigger, GetAvailableActionsPatch, InteractableExfilsService |
| `ActionsTypesClass` | `EFT.UI.InteractionAction` | CustomExfilAction, CustomExfilTrigger, GetAvailableActionsPatch |
| `GetActionsClass` | `EFT.InteractionContextHelper` | GetAvailableActionsPatch |
| `BindableStateClass<T>` | `Diz.Binding.BindableState<T>` | InteractableExfilsService |
| `NotificationManagerClass` | `EFT.Communications.NotificationManager` | CustomExfilTrigger, Examples |

4.0에서는 전부 **전역 네임스페이스**에 있어서 `using` 이 필요 없었습니다. 이제 각자
네임스페이스가 생겨서 `EFT.UI` / `EFT.Communications` / `Diz.Binding` using 을 파일별로
추가했습니다.

소스에 등장하는 식별자 364개를 SPT 4.1 wiki 의 4.0→4.1 대응표에 전수 대조했고, 걸린 건
위 5개뿐입니다 (나머지는 원래부터 실명이라 표에 없음 = 그대로).

### 대응표만 믿지 않고 실기 로그로 교차 확인했습니다

SPT 4.1.5 클라이언트의 BepInEx 로그에는 HarmonyX 가 패치 대상 메서드의 IL 을 덤프해
둡니다. 거기서 실제 4.1 시그니처를 직접 확인했습니다:

```
EFT.InteractionContextHelper::GetAvailableActions(EFT.GamePlayerOwner owner, EFT.IInteractive interactive)
EFT.InteractionContextHelper::GetAvailableActions(EFT.GamePlayerOwner, EFT.Interactive.ExfiltrationPoint)
EFT.InteractionContextHelper::GetAvailableActions(EFT.GamePlayerOwner, EFT.Interactive.Switch)
EFT.UI.AvailableInteractionState::Actions / Error / SelectedAction / InitSelected() / SelectNextAction()
EFT.UI.InteractionAction::Name / Action / Disabled
EFT.Communications.NotificationManager::DisplayWarningNotification(System.String, ...)
Diz.Binding.BindableState
```

이 모드가 쓰는 멤버가 전부 살아 있고, `GetAvailableActions` 의 첫 인자 이름이 4.1에서도
`owner` 라는 것까지 확인됩니다 (`GetAvailableActionsPatch` 가 대상 메서드를 그 이름으로
찾습니다).

---

## 빌드 설정도 갈아엎었습니다

- `net471` → `netstandard2.1` (SPT 4.1 클라 플러그인 기준)
- **`PathToSPT` 가 `..\..` 였습니다** — 레포가 SPT 설치 폴더 정확히 두 단계 안에 있을
  때만 참조가 풀리는 구조라, 다른 데 클론하면 전부 실패했습니다. `SptRoot` 로 대체
  (기본값 `E:\SPT 4.1`, `-p:SptRoot=...` 또는 환경변수로 덮어쓰기). 기존에
  `PathToSPT` 를 넘기던 방식도 별칭으로 계속 동작합니다
- `SptRoot` 가 SPT 설치본이 아니면 "타입을 찾을 수 없음" 수십 줄 대신 **이유를 말하는
  에러 하나**로 실패합니다
- 빌드 후 복사가 `cmd` 원라이너를 `Exec` 로 부르는 방식이라 **Windows 밖에서는 아무것도
  안 하면서 성공으로 보고**했습니다. MSBuild `Copy` 로 교체

## 빌드

```
dotnet build InteractableExfilsAPI.csproj -c Release
dotnet build InteractableExfilsAPI.csproj -c Release -p:"SptRoot=D:\내 SPT 경로"
```

빌드하면 `$(SptRoot)\BepInEx\plugins\InteractableExfilsAPI\` 에 dll + LICENSE 를
복사합니다.

---

## 확인한 것 / 확인 못 한 것

| | 상태 |
|---|---|
| 타입 5개 대응 | **확인** — 위키 4.0→4.1 표 + 실제 4.1 클라 로그의 IL 덤프 |
| 쓰는 멤버가 4.1에 존재하는지 | **확인** (위 IL 덤프) |
| 4.1 형태 어셈블리로 전체 컴파일 | **통과** — 4.0 `Assembly-CSharp` 에 위 5개 리네임을 Cecil로 실제 적용한 DLL을 만들어 빌드 |
| 실제 4.1 `Assembly-CSharp.dll` 로 컴파일 | **못 함** — 이 작업 환경에 4.1 클라이언트 어셈블리가 없습니다 |
| 인게임 검증 | **안 함** |

### 안 되면 여기부터 보세요

`GetAvailableActionsPatch` 는 패치 대상을 `AccessTools.FirstMethod(...)` 로 "첫 인자
이름이 `owner` 인 첫 번째 오버로드" 라고 찾습니다. 4.1의 `InteractionContextHelper` 에는
그 조건을 만족하는 오버로드가 20개 넘게 있어서, 리플렉션이 돌려주는 순서가 4.0과 달라
지면 **엉뚱한 오버로드에 붙고 조용히 아무 일도 안 합니다.** 원작 그대로 둔 부분이라
포팅으로 바뀐 건 없지만, 출구 상호작용이 아예 안 뜨면 여기가 첫 번째 용의자입니다.

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
