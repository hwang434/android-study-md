# LiveData and LiveData observers

 - 이번 실습에서는 LiveData와 ViewModel의 통합을 배울 것입니다.
 - LiveData는 안드로이드 아키텍처 구성요소 중 하나로, data 객체를 만들고, data 객체의 변화가 생기면 뷰에 이 변화를 알립니다.
 - LiveData를 사용하기 위해서는 observers를 설정해야 합니다.
 - observers는 app의 데이터의 변화를 감지합니다.
 - LiveData는 수명주기 인식을 합니다. 그래서 라이프 사이클이 active 상태인 앱의 구성 요소 관찰자만 업데이트합니다.

# 이번에 배울 것.
 - 뭐가 LiveData 객체들을 유용하게 만드나.
 - 뷰모델에 저장된 데이터에 LiveData를 더하는 방법.
 - MutableLiveData를 언제 어떻게 써야하는지.
 - LiveData의 변화를 감지하는 observer 메소드를 어떻게 만드는지.
 - backing property를 사용해서 LiveData를 캡슐화하는 법.
 - UI 컨트롤러와 협업하는 ViewModel과 통신하는 방법.

# 앱의 개요

[](/image/Android%20Kotlin%20기본/LiveData/01.png)

# 실습 시작하기

 - 만약 당신이 이전 과정을 따라오지 않았다면 [링크](https://github.com/google-developer-training/android-kotlin-fundamentals-apps/tree/master/GuessTheWordViewModel)에서 코드를 받을 수 있습니다.
 - 앱을 실행해보세요.
 - skip 버튼 got it 버튼 end game 버튼 눌러보세요.

# GameViewModel에 라이브데이터 추가하기.

 - 라이브데이터는 관찰 가능한 data holder(데이터를 가지고 있는 객체)로 수명주기 인식이 가능합니다.
 - 예를 들어, GuessTheWorld 앱에 current score를 LiveData로 감쌀 수 있습니다.
 - 이번 코드랩에서 LiveData의 몇가지 특징을 배워볼 것입니다.

## LiveData의 특징
 - LiveData는 관찰 가능합니다. 즉, LiveData가 가지고 있는(hold) 객체가 바뀐다면, 관찰자(observer)에게 이를 알립니다.
 - LiveData는 데이터를 가지고 있습니다.
 - LiveData는 wrapper(감쌀수있다?)로, 어떤 데이터와도 같이 사용할 수 있습니다.
 - ~~LiveData는 수명주기를 인식합니다. 관찰자(observer)를 LiveData에 붙일 때 관찰자는 LifecycleOwner(주로 Activity나 Fragment)와 협동하고 있습니다. LiveData는 활동 중인 라이프 사이클(STARTED or RESUMED) 안에 있는 관찰자에게만 정보를 업데이트합니다.
 - 라이브데이터와 관찰에 대해서는 [링크](https://developer.android.com/topic/libraries/architecture/livedata.html#work_livedata)에서 읽을 수 있습니다.
<br>  

 - 이번 실습에선 GameViewModel의 현재 점수 및 현재 단어 데이터를 LiveData로 변환하여 모든 데이터 유형을 LiveData 개체로 래핑하는 방법을 배웁니다.
 - 나중 실습에서는 LiveData의 관찰자를 추가하고, LiveData를 관찰하는 법을 배웁니다.

Step 1: Change the score and word to use LiveData
 - screens/game 패키지에 들어갑니다. GameViewModel 파일에 들어갑니다.
 - score과 word를 MutableLiveData로 변경합니다.
 - MutableLiveData는 값이 바뀔 수 있는 LiveData입니다. 제네릭 클래스로 가지고 있을 데이터 타입을 정의해줘야 합니다.


``` kotlin
// The current word
val word = MutableLiveData<String>()
// The current score
val score = MutableLiveData<Int>()
```

 - init 블록에서 LiveData의 값을 변경하기 위해 score와 word를 초기화 합니다.

``` kotlin
init {
    word.value = ""
    score.value = 0
}
```
 
# LiveData 참조 객체를 업데이트 하기

 - GameViewModel 클래스에서 onSkip() 메소드를 변경합니다. 
 - score -> score.value

``` kotlin
fun onSkip() {
   score.value = (score.value)?.minus(1)
   nextWord()
}
```

 - onCorrect와 nextWord 메소드도 이와 같이 수정합니다.


``` kotlin
fun onCorrect() {
   score.value = (score.value)?.plus(1)
   nextWord()
}

private fun nextWord() {
   if (!wordList.isEmpty()) {
       //Select and remove a word from the list
       word.value = wordList.removeAt(0)
   }
}
```

 - GameFragment에 메소드들도 수정합니다.

``` kotlin
/** Methods for updating the UI **/
private fun updateWordText() {
   binding.wordText.text = viewModel.word.value
}

private fun updateScoreText() {
   binding.scoreText.text = viewModel.score.value.toString()
}

private fun gameFinished() {
   Toast.makeText(activity, "Game has just finished", Toast.LENGTH_SHORT).show()
   val action = GameFragmentDirections.actionGameToScore()

   //viewModel의 score.value가 null이면 0을 대입.
   action.score = viewModel.score.value?:0
   NavHostFragment.findNavController(this).navigate(action)
}
```

# LiveData 객체에 Observer(관찰자) 붙이기
 - 이번 실습에서는 Observer 객체를 LiveData 객체에 붙일 것입니다.
 - 프래그먼트(viewLifecycleOwner)를 LifecycleOwner로 사용합니다.

# 왜 viewLifecycleOwner를 사용하는가?
 - 프래그먼트는 프래그먼트 자체가 파괴되지 않아도 프래그먼트에서 벗어나면 프래그먼트가 파괴됩니다.
 - 이는 필수적으로 두가지의 라이프 사이클을 만듭니다. 하나는 fragment의 라이프사이클 나머지 하나는 fragment의 view의 라이프 사이클.
 - 프래그먼트 뷰의 라이프사이클 대신 프래그먼트의 라이프사이클을 참조하면 프래그먼트 뷰를 업데이트할 때 버그가 생길 수 있습니다.
 - 그러므로, 프래그먼트 뷰의 영향을 끼치는 Observer를 설정할 땐.<br>1.반드시 observer의 설정을 onCreateView()에서 해야합니다. <br>2.viewLifecycleOwner를 observer에게 넘겨야 합니다.

<br>
<br>
<hr>
 - GameFragment의 onCreateView() Observer를 LiveDate의 붙일 것입니다.
 - observe 메소드를 사용합니다.
 - 점수를 위한 viewModel.score의 observer를 추가합시다.


``` kotlin
/** Setting up LiveData observation relationship **/
//viewModel의 score는 LiveData.
//LiveData의 observe 메소드를 이용해서 Observer 추가.
//Observer는 score가 변할 때마다 binding.score.text에 점수를 출력해줌.
viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
   binding.scoreText.text = newScore.toString()
})
```

 - 이제 문제를 위한 observer를 추가합시다.

``` kotlin
/** Setting up LiveData observation relationship **/
viewModel.word.observe(viewLifecycleOwner, Observer { newWord ->
   binding.wordText.text = newWord
})
```

 - 이제 점수 LiveData와 단어 LiveData의 관찰자를 추가했고, observer에서 데이터가 변하면 화면에 텍스트를 변경하게 설정했습니다.
 - 이제 화면에 변경을 위한 메서드가 필요 없습니다.(observer에서 데이터가 변경 되면 알아서 화면을 변경해주기 때문에) updateWordText()와 updateScoreText()를 모두 지웁니다.

6. Task: Encapsulate the LiveData

 - 현재 코드에서는 score와 word를 viewModel.score.value로 접근하여 수정할 수 있습니다.
 - 오직 ViewModel만 데이터를 수정할 수 있어야 합니다. 그러나 UI 컨트롤러까 data를 읽어야 하므로, data 필드는 완전하게 private이 될 수는 없습니다.
 - 어플리케이션을 캡슐화하기 위해서는 MutableLiveData와 LiveData 객체를 이용해야 합니다.

# MutableLiveData vs LiveData
 - MutableLiveData 객체에 있는 데이터들은 바뀔 수 있습니다. 현재 코드에 ViewModel 내부에 있는 데이터는 수정이 가능해야 하므로 MutableLiveData를 사용했습니다.
 - LiveData 객체에 있는 데이터들을 읽기만 가능합니다. ViewModel 외부에서 데이터는 읽을 수만 있어야 하며, 수정이 불가능해야합니다. 그러므로 데이터는 MutableLiveData가 아닌 LiveData로 노출되어야 합니다.

 - 이를 구현하기 위해 여기에선 Kotlin의 [backing property](https://kotlinlang.org/docs/properties.html#backing-properties)를 사용해야 합니다.


# Add a backing property to score and word
 - GameViewModel에서 score를 private으로 만듭니다.
 - backing properties의 네이밍 컨벤션을 따르기 위해, score를 _score로 변경합니다. _socre은 이제부터 game score에서 내부적으로 사용 될 score의 수정 가능한 버전입니다.
 - LiveData 타입의 score 변수를 만듭니다.
 - initialization error가 뜹니다. 이 오류는 GameFragment 내부에서 점수가 LiveData 참조이기 때문에 발생합니다. score는 더 이상 해당 setter에 접근할 수 없습니다. 이에 대해 자세히 알아보려면 [링크](https://kotlinlang.org/docs/reference/properties.html#getters-and-setters)를 클릭하세요.
 - 이 에러를 해결하기 위해 score에 get메소드를 오버라이딩하고 backing property인 _score를 리턴하세요.<br>이제 score에 접근하면, _score가 자동으로 불려옵니다.

 - GameViewModel에서 score를 참조하는 애들을 전부 _score로 변경해주세요.

``` kotlin
init {
   ...
   _score.value = 0
   ...
}

...
fun onSkip() {
   _score.value = (score.value)?.minus(1)
  ...
}

fun onCorrect() {
   _score.value = (score.value)?.plus(1)
   ...
}
```

 - GameViewModep에 word도 똑같이 _word로 이름을 변경하고, backing property를 추가하세요.

``` kotlin
// The current word
private val _word = MutableLiveData<String>()
val word: LiveData<String>
   get() = _word
...
init {
   _word.value = ""
   ...
}
...
private fun nextWord() {
   if (!wordList.isEmpty()) {
       //Select and remove a word from the list
       _word.value = wordList.removeAt(0)
   }
}
```

 - 잘했습니다. 이제 LiveData 객체인 word와 score가 캡슐화 되었습니다.

<hr>

 - 현재 앱은 End Button을 누르면 점수 화면으로 넘어갑니다. 하지만 당신은 모든 단어를 풀면 score 점수로 넘어가는 기능을 원합니다.
 - 이번엔 모든 단어를 유저가 다 돌면, End Button을 누르지 않고도 화면이 이동하는 기능을 만들고자 합니다.
 - 이를 구현하기 위해서 모든 단어가 끝났을 때 fragment와 viewModel이 통신하는 이벤트가 필요합니다.<br>이를 위해 이번 실습에선 LiveData observer pattern을 사용할 것입니다.

![event가 상태 변화를 야기합니다.](/image/Android%20Kotlin%20기본/LiveData/02.png)

- 이번 실습에서 observable한 것은 LiveData 객체이고, observer들은 UI 컨트롤러에 있는 메소드들입니다.
- LiveData 클래스들은 ViewModel과 fragment가 통신하는데 굉장히 중요합니다.

# LiveData를 사용하여 game-finished 이벤트를 감지하기
 - GameViewModel에 Boolean형 MutableLiveData인 _eventGameFinish를 만듭니다.
 - _eventGameFinish에 backing property인 eventGameFinish를 만듭니다.

``` kotlin
// Event which triggers the end of the game
private val _eventGameFinish = MutableLiveData<Boolean>()
val eventGameFinish: LiveData<Boolean>
   get() = _eventGameFinish
```

 - GameViewModel에 onGameFinish() 메소드를 만듭니다. 게임이 끝나면 eventGameFinish를 true로 만들어줍니다.

``` kotlin
/** Method for the game completed event **/
fun onGameFinish() {
   _eventGameFinish.value = true
}
```

 - GameVIewModel에 nextWord() 메서드에 wordList가 비었으면 게임을 끝내는 메서드를 넣어줌.

```
private fun nextWord() {
   if (wordList.isEmpty()) {
       onGameFinish()
   } else {
       //Select and remove a _word from the list
       _word.value = wordList.removeAt(0)
   }
}
```

 - GameFragment로 들어가서 onCreateView 내부에 observer를 eventGameFinish에 붙이기.

``` kotlin
// Observer for the Game finished event
viewModel.eventGameFinish.observe(viewLifecycleOwner, Observer<Boolean> { hasFinished ->
   if (hasFinished) gameFinished()
})
```

 - 이제 앱을 실행하고 모든 단어를 스킵해보세요. 마지막 단어가 끝나면 score 화면으로 자동으로 넘어갈 것입니다.

![](/image/Android%20Kotlin%20기본/LiveData/03.png)
![](/image/Android%20Kotlin%20기본/LiveData/04.png)

 - 그러나 이렇게 추가한 코드로 인해 버그가 생길 것입니다.
 - GameFragment로 들어가서 gameFinished에 Toast를 제외한 메시지들을 주석 처리합니다.

``` kotlin
private fun gameFinished() {
       Toast.makeText(activity, "Game has just finished", Toast.LENGTH_SHORT).show()
//        val action = GameFragmentDirections.actionGameToScore()
//        action.score = viewModel.score.value?:0
//        NavHostFragment.findNavController(this).navigate(action)
   }
```

 - 이제 앱을 다시 실행한 후에 다시 모든 단어들을 스킵합니다.
 - 그럼 당연하게도 Toast는 주석 처리 안했기 때문에 Toast 메시지가 뜰 것이고 화면은 이동하지 않을 것입니다. 이는 action을 주석처리했기 때문에 당연한 것입니다. 
 
![](/image/Android%20Kotlin%20기본/LiveData/05.png)

 - 화면을 회전해봅니다.

![](/image/Android%20Kotlin%20기본/LiveData/06.png)

 - 화면을 회전할 때마다 Toast 메시지가 뜨는 것을 확인할 수 있습니다. 이는 의도치 않은 버그입니다. 게임이 종료됐다는 메시지는 fragment가 re-create될 때마다 뜨는 게 아니라, 딱 한 번만 떠야 합니다. 이 버그는 다음 실습에서 고칠 것입니다.

# Reset the game-finished event

 - 일반적으로 LiveData는 데이터가 변경 되었을 때만 observer에게 업데이트를 전달합니다.
 - 예외적으로 observer가 inactive상태에서 active 상태로 갈 때도 업데이트를 전달한다는 것입니다.
 - 화면 회전으로 인해, GameFragment가 re-create 되면서, onCreateView 내부에서 observer와 viewModel의 연결이 다시 이루어집니다.
 - 게임이 이미 종료된 상태이므로 viewModel의 eventGameFinish는 true일 것이며, 다시 연결함으로 인해, gameFinished 메소드가 다시 불릴 것입니다.

<hr>

 - 이번 실습에서는 이러한 문제를 고치도록 하기 위해 eventGameFinish를 게임이 끝나면 reset하기로 했습니다.
 - GameViewModel에 onGameFinishComplete() 메소드를 추가합니다. 이 메소드는 게임이 끝나면 eventGameFinish를 false로 바꿔 게임이 끝났음을 초기화합니다.

``` kotlin
/** Method for the game completed event **/

fun onGameFinishComplete() {
   _eventGameFinish.value = false
}
```

 - GameFragment에서 gameFinished에 끝에 onGameFinishComplete 메서드를 추가합니다. 이제 게임이 끝나고 마지막에 onGameFinish를 초기화 하는 onGameFinishComplete 메소드가 추가 될 것입니다.

``` kotlin
    private fun gameFinished() {
        Toast.makeText(activity, "Game has just Finished.", Toast.LENGTH_SHORT).show()
//        val action = GameFragmentDirections.actionGameToScore()
//        action.score = viewModel.score.value?:0
//        NavHostFragment.findNavController(this).navigate(action)
        viewModel.onGameFinishComplete()
    }
```

 - 앱을 다시 실행해보면 이제 화면 전환이 돼도 Toast 메시지가 뜨지 않는 것을 볼 수 있습니다.
 - 왜냐? re-create 되면서 observer에게 업데이트가 가지만, onGameFinish가 false로 바꼈기 때문.
 - 이제 주석처리한 GameFragment의 코드들을 다시 해제합니다.

``` kotlin
private fun gameFinished() {
   Toast.makeText(activity, "Game has just finished", Toast.LENGTH_SHORT).show()
   val action = GameFragmentDirections.actionGameToScore()
   action.score = viewModel.score.value?:0
   findNavController(this).navigate(action)
   viewModel.onGameFinishComplete()
}
```

 - 이제 지금까지 했던 예제를 가지고 ScoreViewModel에 데이터들도 ViewModel과 LiveData를 사용하여 수정해보겠습니다.

 - scoreViewModel의 score 변수를 _score로 바꾸고, 타입은 MutableLiveData로 바꿈.
 - backing property를 위해 score: LiveData<Int>를 추가.
 - 
``` kotlin
private val _score = MutableLiveData<Int>()
val score: LiveData<Int>
   get() = _score
```

 - init 블록에 밑에 코드 추가.

``` kotlin
init {
   _score.value = finalScore
}
```

 - ScoreFragmen에 ViewModel에서 score에 observer 추가.

``` kotlin
// Add observer for score
viewModel.score.observe(viewLifecycleOwner, Observer { newScore ->
   binding.scoreText.text = newScore.toString()
})
```

# Play Again Button 추가하기
 - res/layout/score_fragment.xml 들어가서 play_again_button visibility 속성을 visible로 변경.
 - ScoreViewModel에 Boolean을 가지고 있을 LiveData _eventPlayAgain 변수 추가.

``` kotlin
private val _eventPlayAgain = MutableLiveData<Boolean>()
val eventPlayAgain: LiveData<Boolean>
   get() = _eventPlayAgain
```

 - ScoreViewModel에 onPlayAgain 메서드와 onPlayAgaminComplete 메서드를 추가.
 - onPlayAgain은 _eventPlayAgain을 true로 만들어주는 메서드로, 이 메서드가 실행 되어서 _eventPlayAgain이 true가 되면, eventPlayAgain을 관찰하던 observer 변화를 감지해, Game화면으로 이동시켜줍니다.
 - onPlayAgainComplete() 메서드는 onPlayAgain이 끝난 후, _eventPlayAgain 값을 원래 값인 false로 두기 위한 메서드입니다.

``` kotlin
fun onPlayAgain() {
   _eventPlayAgain.value = true
}
fun onPlayAgainComplete() {
   _eventPlayAgain.value = false
}
```

 - ScoreFragment에서 observer를 viewModel에 붙입니다.

``` kotlin
// Navigates back to game when button is pressed
// 옵저버 설정.
viewModel.eventPlayAgain.observe(viewLifecycleOwner, Observer { playAgain ->
   //만약 eventPlayAgain이 true면 게임 화면으로 이동.
   if (playAgain) {
      findNavController().navigate(ScoreFragmentDirections.actionRestart())
       viewModel.onPlayAgainComplete()
   }
})
```

 - ScoreFragment에서 리스타트 버튼을 누르면 onPlayAgain 메서드가 불리게 설정합니다.

``` kotlin
binding.playAgainButton.setOnClickListener {  viewModel.onPlayAgain()  }
```

# 요약
```
11. Summary
LiveData
LiveData is an observable data holder class that is lifecycle-aware, one of the Android Architecture Components.
You can use LiveData to enable your UI to update automatically when the data updates.
LiveData is observable, which means that an observer like an activity or an fragment can be notified when the data held by the LiveData object changes.
LiveData holds data; it is a wrapper that can be used with any data.
LiveData is lifecycle-aware, meaning that it only updates observers that are in an active lifecycle state such as STARTED or RESUMED.
To add LiveData
Change the type of the data variables in ViewModel to LiveData or MutableLiveData.
MutableLiveData is a LiveData object whose value can be changed. MutableLiveData is a generic class, so you need to specify the type of data that it holds.

To change the value of the data held by the LiveData, use the setValue() method on the LiveData variable.
To encapsulate LiveData
The LiveData inside the ViewModel should be editable. Outside the ViewModel, the LiveData should be readable. This can be implemented using a Kotlin backing property.
A Kotlin backing property allows you to return something from a getter other than the exact object.
To encapsulate the LiveData, use private MutableLiveData inside the ViewModel and return a LiveData backing property outside the ViewModel.
Observable LiveData
LiveData follows an observer pattern. The "observable" is the LiveData object, and the observers are the methods in the UI controllers, like fragments. Whenever the data wrapped inside LiveData changes, the observer methods in the UI controllers are notified.
To make the LiveData observable, attach an observer object to the LiveData reference in the observers (such as activities and fragments) using the observe() method.
This LiveData observer pattern can be used to communicate from the ViewModel to the UI controllers.
```


<hr>

잘모르겠는 거
캡슐화라는데 백킹 필드는 도대체 정확히 뭐냐. 직접 접근을 막는 거 같은데 헷갈림
// Event which triggers the end of the game
private val _eventGameFinish = MutableLiveData<Boolean>()
val eventGameFinish: LiveData<Boolean>
   get() = _eventGameFinish

   대충 이해하기로는 클래스 내부에서는 _eventGameFinish로 접근해서 라이브데이터를 수정가능하고,
   LiveData클래스에서는 _eventGameFinish에는 접근이 불가능하지만, eventGameFinish로 접근 가능함.
   근데 eventGameFinish로 가져온 건 Mutable이 아닌 그냥 LiveData이므로 수정이 불가능함 -> _eventGameFinish가 수정이 불가능함.