## 1. Persistent Base Scene: `XR+UI.unity`

Main responsibility: keep AR and navigation alive while period scenes are swapped in and out.

```mermaid
flowchart TD
    User["User taps timeline buttons"] --> UIDoc["UIDocument / UI Toolkit pages"]
    UIDoc --> EraNavigator["EraNavigator.cs"]

    EraNavigator --> Pages["eraPages UXML array"]
    EraNavigator --> SceneNames["arSceneNames array"]
    EraNavigator --> Loading["loading.uxml"]

    EraNavigator --> Load["LoadSceneAsync(scene, Additive)"]
    Load --> PeriodScene["Selected geological period scene"]
    EraNavigator --> HideUI["Hide + disable timeline UI"]

    PeriodScene --> ExitButton["Exit button in AR scene"]
    ExitButton --> ExitARScene["ExitARScene.cs"]
    ExitARScene --> EraNavigator
    EraNavigator --> Unload["UnloadSceneAsync(loadedScene)"]
    Unload --> RestoreUI["Restore same timeline page"]

    DimButton["Dim toggle button"] --> DimmerToggle["DimmerToggle.cs"]
    DimmerToggle --> Dimmer["Dimmer object tagged 'Dimmer'"]
    Dimmer --> Readability["Darkens camera feed for readable text"]

    ARSession["AR Session / XR Origin / AR Camera"] --> PeriodScene
    EventSystem["EventSystem + input modules"] --> UIDoc
    EventSystem --> PeriodScene
```

## 2. Main Menu Timeline Page

Main responsibility: introduce the app and move users into the geological timeline.

```mermaid
flowchart TD
    MainMenu["1. MainMenu.uxml"] --> NextButton["next-btn"]
    NextButton --> EraNavigator["EraNavigator.ShowPage(1)"]
    EraNavigator --> PrecambrianPage["Precambrian timeline page"]
```

## 3. Precambrian Scene: `1_precambrian`

Main responsibility: visual AR environment and informational content.

```mermaid
flowchart TD
    Base["XR+UI base scene"] --> Load["EraNavigator loads 1_precambrian"]
    Load --> Precambrian["Precambrian visual scene"]
    Precambrian --> Models["Atmospheric objects / sky / information"]
    Precambrian --> ExitButton["Exit button"]
    ExitButton --> ExitARScene["ExitARScene.cs"]
    ExitARScene --> BaseReturn["Return to timeline through EraNavigator"]
```

## 4. Cambrian Scene: `2_cambrian`

Main responsibility: bubble-popping interaction with score, facts, and feedback.

```mermaid
flowchart TD
    Load["EraNavigator loads 2_cambrian"] --> Intro["Instruction text and image visible"]

    Intro --> TapButton["tap_button.cs on active UI Button"]
    TapButton --> HideIntro["Hides instruction text and RawImage"]
    TapButton --> HideButton["Hides its own button GameObject"]
    TapButton --> ScoreUI["Shows score text"]
    ScoreUI --> GameStart["Bubble activity is now the main interaction"]

    GameStart --> GoodBubble["GoodBubble in show_info.cs"]
    GameStart --> BadBubble["BadBubble in bad_bubble.cs"]
    GameStart --> Bobbing["Bobbing.cs on bubbles"]

    Bobbing --> BubbleMotion["Floating up/down motion"]

    UserTap["User taps screen"] --> CameraRay["AR camera ScreenPointToRay"]
    CameraRay --> PhysicsRaycast["Physics.Raycast against bubble colliders"]

    PhysicsRaycast --> GoodBubble
    GoodBubble --> FactText["Show fact text"]
    GoodBubble --> Add10["ScoreManager.AddScore(+10)"]
    GoodBubble --> GoodLeft["ScoreManager.GoodBubblePopped()"]
    GoodBubble --> PopAudio["Play pop sound"]
    GoodBubble --> DestroyGood["Destroy good bubble"]

    PhysicsRaycast --> BadBubble
    BadBubble --> Minus5["ScoreManager.AddScore(-5)"]
    BadBubble --> BadAudio["Play pop sound"]
    BadBubble --> DestroyBad["Destroy bad bubble"]

    ScoreManager["ScoreManager.cs singleton"] --> ScoreUI
    GoodLeft --> DonePanel["Show done panel when all good bubbles are popped"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 5. Ordovician Scene: `3_ordovician`

Main responsibility: slider-controlled climate change, text visibility, and extinction effect.

```mermaid
flowchart TD
    Load["EraNavigator loads 3_ordovician"] --> Slider["Unity UI Slider"]

    Slider --> ChangingSaturation["ChangingSaturation.cs"]
    ChangingSaturation --> URPVolume["URP Volume / ColorAdjustments"]
    URPVolume --> ColdLook["Desaturation + blue colour filter"]

    Slider --> OpacitySlider["OpacitySlider.cs"]
    OpacitySlider --> WarmText["Show text at one end of slider"]

    Slider --> InverseOpacitySlider["InverseOpacitySlider.cs"]
    InverseOpacitySlider --> ColdText["Show text at opposite end of slider"]

    Slider --> ExtinctSlider["ExtinctSlider.cs"]
    ExtinctSlider --> LifeGroup["lifeGroup children"]
    LifeGroup --> FishDie["FishDie.cs on each fish"]
    FishDie --> Rigidbody["Rigidbody settings"]
    Rigidbody --> FishEffect["Alive = kinematic, dead = gravity falls"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 6. Silurian Scene: `4_silurian`

Main responsibility: visual/informational exhibit-style AR scene.

```mermaid
flowchart TD
    Load["EraNavigator loads 4_silurian"] --> Silurian["Silurian scene content"]
    Silurian --> Models["Models / images / information panels"]
    Silurian --> ExitButton["Exit button"]
    ExitButton --> ExitARScene["ExitARScene.cs"]
    ExitARScene --> Return["Unload scene and return to timeline"]
```

## 7. Devonian Scene: `5_devonian`

Main responsibility: environmental scene with terrain, water, lighting, and information.

```mermaid
flowchart TD
    Load["EraNavigator loads 5_devonian"] --> Devonian["Devonian environment"]
    Devonian --> Terrain["Terrain / water / models"]
    Devonian --> Volume["Post-processing volume"]
    Devonian --> InfoPanels["Information panels"]
    Devonian --> ExitButton["Exit button"]
    ExitButton --> ExitARScene["ExitARScene.cs"]
```

## 8. Lower Carboniferous Scene: `6_lower_carboniferous`

Main responsibility: start a quiz after hiding the normal scene content.

```mermaid
flowchart TD
    Load["EraNavigator loads 6_lower_carboniferous"] --> NormalScene["Initial information / objects / facts"]

    StartButton["Start Quiz button"] --> StartQuiz["start_quiz.cs"]
    StartQuiz --> ShowQuizPanel["Show quizPanel"]
    StartQuiz --> HideStart["Hide startButton"]
    StartQuiz --> HideScene["Hide restOfScene / info / allFacts"]
    StartQuiz --> Q1["Show question1"]
    StartQuiz --> Q2Hidden["Hide question2"]
    StartQuiz --> Q3Hidden["Hide question3"]

    Q1 --> QuizManager1["QuizManager.cs"]
    QuizManager1 --> AnswerCheck["CheckAnswer(selected)"]
    AnswerCheck --> Correct["Show correct image for 0.5s"]
    Correct --> DestroyQ1["Destroy current question buttons/text"]
    DestroyQ1 --> Q2["Activate nextQuestion"]

    Q2 --> QuizManager2["QuizManager.cs"]
    QuizManager2 --> Q3["Activate nextQuestion"]

    AnswerCheck --> Wrong["Show wrong image for 0.5s"]
    Wrong --> LockWrong["Wrong button turns red and is disabled"]


    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 9. Upper Carboniferous Scene: `7_upper_carboniferous`

Main responsibility: quiz-based scene using the same shared quiz pattern.

```mermaid
flowchart TD
    Load["EraNavigator loads 7_upper_carboniferous"] --> Content["Initial Carboniferous scene content"]

    StartButton["Start Quiz button"] --> StartQuiz["start_quiz.cs"]
    StartQuiz --> QuizPanel["Show quiz panel"]
    StartQuiz --> HideContent["Hide scene content and facts"]
    StartQuiz --> FirstQuestion["Show first question only"]

    FirstQuestion --> QuizManagerA["QuizManager.cs"]
    QuizManagerA --> CorrectFlow["Correct: feedback, destroy current question, show next"]
    QuizManagerA --> WrongFlow["Wrong: feedback, red disabled button"]
    CorrectFlow --> NextQuestion["nextQuestion GameObject"]
    NextQuestion --> QuizManagerB["QuizManager.cs"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 10. Triassic Scene: `8_triassic`

Main responsibility: midpoint slider controlling environment and water level.

```mermaid
flowchart TD
    Load["EraNavigator loads 8_triassic"] --> Slider["SliderTriassic"]

    Slider --> Saturation["ChangingSaturationTriassic.cs"]
    Saturation --> MidPeak1["midPeak = strongest in middle"]
    MidPeak1 --> URPVolume["URP Volume / ColorAdjustments"]
    URPVolume --> ColdMiddle["Middle = desaturated blue, ends = normal"]

    Slider --> WaterRise["WaterRise.cs"]
    WaterRise --> MidPeak2["Same midpoint peak mapping"]
    MidPeak2 --> WaterTransform["Water transform Y position"]
    WaterTransform --> SeaLevel["Water highest in middle"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 11. Jurassic Scene: `9_jurassic`

Main responsibility: quiz gate followed by AR dinosaur placement.

```mermaid
flowchart TD
    Load["EraNavigator loads 9_jurassic"] --> JurassicQuiz["JurassicQuizManager.cs"]

    JurassicQuiz --> InitialHide["Hide quiz UI, feedback images, completion text"]
    JurassicQuiz --> DisablePlacement["Disable TapToPlace at start"]
    JurassicQuiz --> NormalVolume["Set scene colour normal"]
    JurassicQuiz --> Delay["Wait 6 seconds"]

    Delay --> ShowQuiz["Show question and buttons"]
    ShowQuiz --> Darken["Darken/desaturate scene via URP ColorAdjustments"]

    UserAnswer["User presses answer"] --> CheckAnswer["CheckAnswer(selected)"]
    CheckAnswer --> Wrong["Wrong answer"]
    Wrong --> RedLock["Highlight wrong button red and disable"]
    Wrong --> WrongImage["Show wrong image for 0.5s"]

    CheckAnswer --> Correct["Correct answer"]
    Correct --> CorrectImage["Show correct image for 0.5s"]
    Correct --> ResetVolume["Reset saturation and exposure"]
    Correct --> HideRest["Hide restOfScene"]
    Correct --> Completion["Show completion text"]
    Correct --> EnablePlacement["Enable TapToPlace.cs"]
    Correct --> DestroyQuiz["Destroy quiz buttons and text"]

    EnablePlacement --> TapToPlace["TapToPlace.cs"]
    TapToPlace --> ARRaycast["ARRaycastManager.Raycast"]
    ARRaycast --> PlaneCheck["TrackableType.PlaneWithinPolygon + flatness check"]
    PlaneCheck --> Spawn["Instantiate dinosaur prefab"]
    Spawn --> AfterText["Show afterText and resetButton"]

    ResetButton["Reset button"] --> ResetPlacement["TapToPlace.ResetPlacement()"]
    ResetPlacement --> DestroySpawned["Destroy spawned dinosaur"]
    ResetPlacement --> BeforeText["Show beforeText again"]

    Spawn --> DinoTapAttack["DinoTapAttack.cs on dinosaur prefab"]
    UserTap["User taps dinosaur"] --> PhysicsRaycast["Physics.Raycast"]
    PhysicsRaycast --> DinoTapAttack
    DinoTapAttack --> Animator["Animator.SetTrigger('Attack')"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 12. Cretaceous Scene: `10_cretaceous`

Main responsibility: quiz scene plus swimming movement for prehistoric sea content.

```mermaid
flowchart TD
    Load["EraNavigator loads 10_cretaceous"] --> Content["Cretaceous content"]

    Content --> SwimRandom["SwimRandom.cs"]
    SwimRandom --> PickTarget["Pick random target near start position"]
    SwimRandom --> Move["Move toward target each frame"]
    SwimRandom --> Bob["Add sine-wave bobbing"]
    SwimRandom --> Rotate["Rotate toward swim direction"]
    Move --> NewTarget["When close, pick new target"]

    StartButton["Start Quiz button"] --> StartQuiz["start_quiz.cs"]
    StartQuiz --> QuizPanel["Show quiz panel"]
    StartQuiz --> HideContent["Hide normal scene content"]
    QuizPanel --> QuizManager["QuizManager.cs"]
    QuizManager --> CorrectFlow["Correct: feedback and next question"]
    QuizManager --> WrongFlow["Wrong: feedback and red disabled button"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 13. Tertiary Scene: `11_tertiary`

Main responsibility: quiz-based scene.

```mermaid
flowchart TD
    Load["EraNavigator loads 11_tertiary"] --> TertiaryContent["Tertiary scene content"]

    StartButton["Start Quiz button"] --> StartQuiz["start_quiz.cs"]
    StartQuiz --> HideContent["Hide initial content"]
    StartQuiz --> ShowQ1["Show first quiz question"]

    ShowQ1 --> QuizManager["QuizManager.cs / QuizManagerTertiary.cs pattern"]
    QuizManager --> ButtonA["Button A"]
    QuizManager --> ButtonB["Button B"]
    QuizManager --> ButtonC["Button C"]
    ButtonA --> Check["CheckAnswer(0)"]
    ButtonB --> Check["CheckAnswer(1)"]
    ButtonC --> Check["CheckAnswer(2)"]
    Check --> Correct["Correct: image, destroy current UI, show nextQuestion"]
    Check --> Wrong["Wrong: image, red disabled button"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 14. Quaternary Scene: `12_quarternary`

Main responsibility: Ice Age reveal through fade-in image and text.

```mermaid
flowchart TD
    Load["EraNavigator loads 12_quarternary"] --> Button["Reveal button"]
    Button --> ViewPresent["view_present_script.cs"]

    ViewPresent --> StartState["Start: image alpha 0, text hidden"]
    UserClick["User clicks button"] --> OnButtonClick["OnButtonClick()"]
    OnButtonClick --> HideButton["Hide button image and label"]
    OnButtonClick --> FadeIn["StartCoroutine(FadeIn)"]
    FadeIn --> RawImage["Increase RawImage alpha over fadeDuration"]
    RawImage --> FullVisible["Image fully visible"]
    FullVisible --> DisplayText["Show displayText"]
    DisplayText --> DisableButton["Disable reveal button object"]

    ExitButton["Exit button"] --> ExitARScene["ExitARScene.cs"]
```

## 15. Shared Quiz Pattern

This pattern appears in Lower Carboniferous, Upper Carboniferous, Cretaceous, and Tertiary.

```mermaid
flowchart TD
    StartQuiz["start_quiz.cs"] --> HideNormal["Hide normal scene objects"]
    StartQuiz --> ShowPanel["Show quizPanel"]
    StartQuiz --> ShowFirst["Show question1 only"]

    Question["Question GameObject"] --> QuizManager["QuizManager.cs"]
    QuizManager --> Buttons["Answer buttons"]
    Buttons --> CheckAnswer["CheckAnswer(selected)"]

    CheckAnswer --> IsCorrect{"selected == correctAnswer?"}
    IsCorrect --> Correct["Correct"]
    Correct --> CorrectImage["Show correct image"]
    Correct --> DestroyQuestion["Destroy current buttons and question text"]
    DestroyQuestion --> NextQuestion["Activate nextQuestion"]

    IsCorrect --> Wrong["Wrong"]
    Wrong --> WrongImage["Show wrong image"]
    Wrong --> Highlight["Set button red"]
    Highlight --> Disable["Disable wrong button"]
```

## 16. Shared Exit Pattern

Every AR scene uses the same exit idea.

```mermaid
flowchart TD
    ExitButton["Exit button in period scene"] --> ExitARScene["ExitARScene.Exit()"]
    ExitARScene --> FindNavigator["FindFirstObjectByType<EraNavigator>()"]
    FindNavigator --> ReturnToMenu["EraNavigator.ReturnToMenu()"]
    ReturnToMenu --> Unload["Unload current additive scene"]
    Unload --> EnableUIDoc["Enable UIDocument"]
    EnableUIDoc --> ShowPage["Show previous timeline page"]
```
