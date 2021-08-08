# Data binding with ViewModel and LiveData

 - 저번 실습에서는 ViewModel을 사용하여 기기 설정 변경(화면회전)이 일어나도 데이터가 유지되도록 만들었습니다.
 - 또한 LiveData를 이용하여, 데이터가 변경되면 Observer에서 관찰하여 특정 행동을 취하는 기능도 만들었습니다.
 - 이번 코드랩에서는 저번에 사용하였던 GuessTheWorld app을 이용하여 실습을 이어서 할 것입니다.
 - 첫번째로, view와 viewModel을 직접적으로 통신하게 만들 것입니다. 데이터바인딩과 ViewModel을 통합하면 이제 더 이상 click 리스너를 설정할 필요가 없습니다.
 - 두번째로, 이전 실습에서 사용한 코드에선 LiveData의 observer 메소드를 사용하여, 데이터의 변화를 알렸습니다. 이번 실습에선 observer 메소드를 사용하지 않고 LiveData를 데이터 바인딩 소스로 사용할 것입니다.

<hr>

### 이번 실습에서 배울 것들
 - DataBinding 요소들 사용법.
 - ViewModel과 dataBinding 통합법.
 - LiveData와 dataBinding 통합법.
 - listner binding으로 프레그먼트의 click listner 대체하기.
 - 문자열 형식을 데이터바인딩 표현식에 추가하는 법.

<hr>

### 이번 실습에서 할 것들
 - 앱의 뷰를 ViewModel과 직접적으로 연결하여 뷰와 ViewModel이 직접적으로 통신하게 만듭니다.
 - LiveData를 data-binding의 소스로 사용할 것입니다. 이렇게 되면, LiveData 객체가 데이터 변경 사항을 ui에 직접 알리게 됩니다. -> LiveData의 observe 메서드가 필요 없어집니다.

### 실습 시작하기 전에
 - 만약 당신이 이전 과정을 따라오지 않았다면 [링크](https://github.com/google-developer-training/android-kotlin-fundamentals-apps/tree/master/GuessTheWordLiveData)에서 코드를 받을 수 있습니다.

<hr>

### ViewModel에 데이터바인딩 추가하기
 - 현재 코드에선 data binding을 view의 안전하게 접근하기 위해 연결하여 사용하고 있습니다. 그러나 data binding의 진짜 장점은 데이터 바인딩이라는 이름 그대로 데이터를 연동하는데 있습니다.
 - 진짜 장점은 앱의 뷰 객체에 직접 데이터를 바인딩하는 것입니다.

#### 현재 앱의 아키텍처

 - 현재 앱에선 view들이 xml layout의 정의되어 있습니다. 그리고 view의 필요한 데이터들은 viewModel 객체에 있습니다. 
 - view(layout.xml)과 viewModel 사이에는 UI controller(MainActivity, GameFragment)가 사이에서 중계 역할을 합니다.

![](/image/Android%20Kotlin%20기본/DataBindingWithViewModel/01.png)

![](/image/Android%20Kotlin%20기본/DataBindingWithViewModel/02.png)

 - 예시
   - Got it 버튼은 game_fragment.xml 레이아웃 파일에 있음.
   - 사용자가 Got it 버튼을 누르면 GameFragment에서 리스너를 호출하고 GameViewModel에 메서드를 실행합니다.
   - GameViewModel에 onCorrect 메서드가 실행되며 GameViewModel에 있는 점수가 올라가고, Fragment에서 설정한 viewModel의 관찰자가 업데이트 된 점수를 관측 후 화면에 표시된 점수를 올려줍니다.


``` kotlin
override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                              savedInstanceState: Bundle?): View? {
    // 1.binding의 설정 된 Got it 버튼을 누르면 onCorrect가 호출됨.
    binding.correctButton.setOnClickListener { onCorrect() }

    // 3.viewModel의 score의 설정한 관찰자가 점수의 변화를 관찰하고
    // binding을 통해 화면에 점수를 1 올림.
    viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
        binding.scoreText.text= newScore.toString()
    })
}
// 2. viewModel에 onCorrect가 호출되고, 
//    viewModel의 점수가 올라감.
private fun onCorrect() {
    viewModel.onCorrect()
}
```

 - 예시에서 알 수 있듯이 Button을 클릭하면 바로 GameViewModel로 처리가 가는 게 아니라 fragment의 설정된 리스너에서 GameViewModel을 거치는 불필요한 과정이 발생하고 있습니다.
 - 즉, 화면에 버튼(뷰)와 점수(viewModel)이 직접적으로 통신을 하지 않는다는 것입니다.

#### 데이터 바인딩에 전달 된 ViewModel
 - 만약 layout파일(xml)과 viewModel이 직접 연결 된다면 Fragment를 중간 다리로 사용할 필요가 없다.

![](/image/Android%20Kotlin%20기본/DataBindingWithViewModel/01.png)
중간에 UI 컨트롤러를 낀 이전 구조
![](/image/Android%20Kotlin%20기본/DataBindingWithViewModel/03.png)
UI 컨트롤러가 빠진 layout과 viewModel이 직접 연결 된 구조

 - viewModel 객체들을 dataBinding함으로 인해, view와 viewModel의 통신을 UI 컨트롤러 없이 자동화할 수 있습니다.
 - 이번 실습에선 GameViewModel과 ScoreViewModel을 각각의 레이아웃과 협동시킬 것입니다.
 - 또한, binding listner를 설정하여, click 이벤트를 다룰 것입니다.

<hr>

### GameViewModel을 위한 data binding 추가하기
 - GameViewModel(viewModel)과 game_fragment.xml(layout)을 협동시킵니다.
 - game_framgnet에 GameViewModel 타입인 data-binding 변수를 추가합니다.
``` xml
<data>
    <variable
        name="gameViewModel"
        type="com.example.android.guesstheword.screens.game.GameViewModel" />
</data>
```


![](/image/Android%20Kotlin%20기본/DataBindingWithViewModel/04.png)

 - data binding 변수를 선언하였기 때문에 이제 gameViewModel에 binding을 통하여 접근할 수 있습니다.
 - **GameFragment(UI controller)** 파일로 들어가서 바인딩한 변수에 viewModel을 할당합니다.
``` kotlin
// Set the viewmodel for databinding - this allows the bound layout access 
// to all the data in the ViewModel
binding.gameViewModel = viewModel
```

<hr>


### 이벤트 핸들링을 위해 listner binding 사용하기
- **[Listner bindings](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings)** 은 binding 표현식으로 onClick(), onZoomIn(), onZoomOut()과 같은 이벤트가 발동했을 때 사용됩니다. 
- Listner binding은 람다식으로 쓰입니다.
- 데이터 바인딩은 리스너를 생성하고 뷰에 리스너를 설정합니다. listened-for 이벤트가 발생하면 리스너는 람다 식을 평가합니다. 리스너 바인딩은 Android Gradle 플러그인 버전 2.0 이상에서 작동합니다.<br>더 자세한 내용은 **[레이아웃과 바인딩 익스프레션](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings)**에 들어가서 확인하세요.

 - 이번 실습에서 view에 있는 리스너를 game_fragment(레이아웃)에 listner binding으로 대체할 것입니다.
 - game_fragment.xml에 들어가 skip_button에 onClick 속성을 더합니다.
 - 속성의 값으로는 @{() -> gameViewModel.onSkip()}을 씁니다.
 - 이같은 binding expression은 listner binding이라고 불립니다.

``` xml
<Button
   android:id="@+id/skip_button"
   ...
   android:onClick="@{() -> gameViewModel.onSkip()}"
   ... />
```

 - correct_button과 end_game_button에서도 listner binding을 이용하여, gameViewModel의 메서드를 실행시키게 설정합니다.

``` xml
<Button
   android:id="@+id/correct_button"
   ...
   android:onClick="@{() -> gameViewModel.onCorrect()}"
   ... />
<Button
   android:id="@+id/end_game_button"
   ...
   android:onClick="@{() -> gameViewModel.onGameFinish()}"
   ... />

```

 - 이제 레이아웃에서 리스너 바인딩을 설정하였으므로, GameFragment에 리스너를 제거해도 됩니다. (layout에 리스너 바인딩이 이벤트를 처리하니까 fragment에 리스너를 설정할 필요가 없음)

``` kotliun
// 삭제해야 할 코드들.

// 이벤트 설정 코드
binding.correctButton.setOnClickListener { onCorrect() }
binding.skipButton.setOnClickListener { onSkip() }
binding.endGameButton.setOnClickListener { onEndGame() }

/** Methods for buttons presses **/
// 이벤트가 viewModel에서 메서드를 실행하게 하는 메서드들.
// layout에서 @{() -> viewModel.onSkip()} 등으로 대체되었음.
private fun onSkip() {
   viewModel.onSkip()
}
private fun onCorrect() {
   viewModel.onCorrect()
}
private fun onEndGame() {
   gameFinished()
}
```

### ScoreViewModel을 위한 data binding 추가하기

 - score_fragment.xml 설정 코드


``` xml
<layout ...>
   <data>
       <variable
           name="scoreViewModel"
           type="com.example.android.guesstheword.screens.score.ScoreViewModel" />
   </data>
   <androidx.constraintlayout.widget.ConstraintLayout
```

 - 버튼에서 이벤트 처리하기


``` xml
<Button
   android:id="@+id/play_again_button"
   ...
   android:onClick="@{() -> scoreViewModel.onPlayAgain()}"
   ... />
```

 - ScoreFragment 가서 코드 작성

``` kotlin
viewModel = ...
binding.scoreViewModel = viewModel
```

 - ScoreFragment에서 아래 코드 삭제


``` kotlin
binding.playAgainButton.setOnClickListener {  viewModel.onPlayAgain()  }

```

 - 앱 실행해서 테스트해보세요. 정상적으로 작동할 것입니다.

<hr>

### dataBinding의 LiveData 추가하기
 - 데이터바인딩은 ViewModel과 사용되는 LiveData에도 잘 쓰입니다.
 - 방금 실습으로 ViewModel에 데이터 바인딩을 했으므로 LiveData를 통합할 준비가 됐습니다.

#### 단어를 위한 LiveData를 game_fragment에 추가하기
- game_fragment.xml 파일에 들어가서 word_text 아이디를 가진 텍스트 뷰에 text 속성의 값을 @{gameViewModel.word}로 줍니다.

``` xml
<TextView
   android:id="@+id/word_text"
   ...
   android:text="@{gameViewModel.word}"
   ... />
```

 - 위에 코드에서 보면 알 수 있지만, gameViewModel.word를 쓰는 것을 볼 수 있습니다. 이로 인해 LiveData 객체를 바로 사용할 수 있는 것을 알 수 있습니다.
 - LiveData 객체는 현재 값을 표시합니다. 만약 LiveData가 null이면 빈 문자열("")을 반환합니다.
 - GameFragment에 onCreateView 메서드로 들어가서 프래그먼트를 binding 변수의 LifeCycleOwner로 설정합니다. 
 - 이것은 LiveData 객체의 Scope를 정의하여, game_fragment.xml 레이아웃의 값을 자동으로 업데이트하게 합니다.
 - 
``` kotlin
binding.gameViewModel = ...
// 프래그먼트 뷰를 바인딩의 라이프사이클 오너로 등록합니다.
// 이를 설정함으로 인해, 바인딩이 LiveData의 업데이트를 감지할 수 있게 됩니다.
// This is used so that the binding can observe LiveData updates
binding.lifecycleOwner = viewLifecycleOwner
```

 - binding의 lifeCycleOwner를 등록해서 업데이트 감시를 할 수 있게 됐으므로 LiveData의 observer 메소드는 이제 필요 없어졌습니다.
 - GameFragment에서 아래 코드를 삭제합니다.


``` kotlin
// 삭제해야할 코드들.
viewModel.word.observe(viewLifecycleOwner, Observer { newWord ->
   binding.wordText.text = newWord
})
```

 - 앱을 실행해보고 나타나는 단어들이 계속 업데이트 되는 걸 볼 수 있습니다.

#### 점수를 위한 LiveData를 score_fragment에 추가하기
 - 위와 똑같이 합니다. 단, 점수는 Integer이므로 형변환이 필요합니다.

``` xml
<TextView
   android:id="@+id/score_text"
   ...
   android:text="@{String.valueOf(scoreViewModel.score)}"
   ... />
```

 - ScoreFragment로 들어가서 아래 코드를 작성합니다.

``` kotlin
binding.scoreViewModel = ...
// Specify the fragment view as the lifecycle owner of the binding.
// This is used so that the binding can observe LiveData updates
binding.lifecycleOwner = viewLifecycleOwner
```

 - observer를 삭제합니다.

``` kotlin
// Add observer for score
viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
   binding.scoreText.text = newScore.toString()
})
```

<hr>


#### 데이터 바인딩의 문자열 형식 추가하기
 - 레이아웃에서 문자열 형식과 dataBinding을 함께 사용할 수 있습니다.
 - 이번 실습에선 나오는 문자에 더블 쿼터를 추가하고, 점수에는 "Current Score: "를 접두어로 추가할 것입니다.

![](/image/Android%20Kotlin%20기본/DataBindingWithViewModel/05.png)


 - 1.string.xml에 들어가서 아래와 같이 string format을 추가해줍니다.
  
``` xml
<string name="quote_format">\"%s\"</string>
<string name="score_format">Current Score: %d</string>
```

 - game_fragment.xml에 들어가서 id가 word_text인 텍스트뷰에 text를 아래와 같이 수정합니다.
``` xml
<TextView
   android:id="@+id/word_text"
   ...
   android:text="@{@string/quote_format(gameViewModel.word)}"
   ... />
```


 - id가 score_text 텍스트뷰에 text의 값도 아래와 같이 수정합니다.

``` xml
<TextView
   android:id="@+id/score_text"
   ...
   android:text="@{@string/score_format(gameViewModel.score)}"
   ... />
```

 - GameFragment에 들어가서 onCreateView에 score 옵저버를 삭제합니다.

``` kotlin
viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
   binding.scoreText.text = newScore.toString()
})
```

 - 앱을 실행해봅니다.

![](/image/Android%20Kotlin%20기본/DataBindingWithViewModel/06.png)

<hr>

### 완성본 코드
 - ***[완성본 코드 깃허브](https://github.com/google-developer-training/android-kotlin-fundamentals-apps/tree/master/GuessTheWordDataBinding)***

<hr>

### 요약
```

```