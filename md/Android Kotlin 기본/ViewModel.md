# View Model

 - UI와 관련한 데이터를 저장하기 위해 ViewModel을 사용할 것입니다.
 - ViewModel은 화면 회전과 같이 Activity가 Destroy 되더라도, 데이터가 사라지지 않습니다.

# 앱 개요
 - 이번 코드랩에서는 GuessTHeWord app을 개발할 것입니다.
 - 첫번째 사람이 앱에 뜬 단어를 보고 몸으로 말해요를 한 후에 다른 사람이 맞추면 Got it 버튼을 눌러 점수를 올립니다.
 - 실패 시, 문제를 skip합니다.
 - 게임이 끝나면 END GAME을 누릅니다.

![](/image/Android%20Kotlin%20기본/ViewModel/01.png)

# 앱을 살펴보자
 - [주소](https://github.com/google-developer-training/android-kotlin-fundamentals-starter-apps/tree/master/GuessTheWord-Starter)로 들어가서 코드를 다운 받고 코드의 흐름을 살펴봅시다.

# 스타터 코드에서 문제점 찾기
 - End 버튼 안 먹음.
 - 화면 회전하면 액티비티 새로 그려지면서 점수 초기화.
 - 앱 껐다 키면 점수 초기화됨.

# 이 앱의 이슈
 - 이 앱은 화면회전이나, 앱이 꺼졌다가 다시 실행할 때, 앱의 상태를 저장하거나 회복하지 않습니다.
 - END GAME 버튼을 누르면, END GAME 화면으로 넘어가지 않음.
 - 이번 코드랩에서 이런 이슈들을 [앱 아키텍처](https://developer.android.com/jetpack/guide)의 컴포넌트들을 사용해서 해결할 것입니다.

# 앱 아키텍처
 - 앱 아키텍쳐는 앱의 클래스들과 클래스들 간의 관계들을 디자인 하는 방법입니다.
 - 이러한 아키텍처를 사용함으로서 앱이 어떤 시나리오에서도 구동이 잘 되고, 추가적으로 작업할 때도 편리하게 할 수 있습니다.
 - 이번 코드랩에서는 (안드로이드 앱 아키텍처)[https://developer.android.com/jetpack/guide]를 사용하여, 프로젝트를 개선할 것입니다.
 - 이 안드로이드 아키텍처를 MVVM(model-view-viewmodel) 아키텍처와 굉장히 유사한 패턴을 가지고 있습니다.
 - 주어진 GuessTheWord 앱은 관심사 분리 설계 원칙의 기본 원리를 따르고 있으며, 각각의 클래스는 별도의 관심사를 처리합니다.
 - 이번 코드랩에서는 UI controller와 View Model, ViewModelFactory를 사용할 것입니다.

# UI Controller
- UI 컨트롤러는 Activity나 Fragment UI 기반 클래스입니다.
- UI 컨트롤러예써는 UI 관련 코드와, 뷰 보여주기나 사용자 입력 받기와 같은 OS 관련 코드 로직을 작성해야합니다. 
- 표시할 텍스트를 결정하는 것과 같은 의사 결정 로직 코드를 넣지 않습니다. (그런 코드는 다른 곳에서 처리한 후, UI 컨트롤러는 그 명령대로 텍스트를 보여줄뿐)
 
<br>

- 이번 실습 코드에서는 GameFragment, ScoreFragment, TitleFragment가 UI 컨트롤러가 됩니다.
- 관심사 분리 원칙(separation of concerns)에 따라 GameFragment는 화면을 그리는 것과 클릭 이벤트에 처리만 합니다. 만약 사용자가 버튼을 클릭하면 이러한 정보들은 GameViewModel로 넘어갑니다. 

# ViewModel
 - 뷰모델은 뷰모델과 협동하는 프래그먼트나 액티비티에 보여줄 데이터를 가지고 있습니다.
 - 뷰모델은 UI 컨트롤러에 의해 보여질 데이터들로 간단한 계산이나 변환을 할 수 있습니다.
 - 이번 구조에서는 뷰모델이 의사결정을 하는 코드를 구현할 것입니다.

# ViewModelFactory
 - 뷰모델팩토리는 생성자를의 매개변수를 가질 수도 안 가질 수도 있습니다.
 - 뷰모델팩토리는 ViewModel 객체를 인스턴트화합니다.

![](/image/Android%20Kotlin%20기본/ViewModel/02.png)

 - 이번 과정에서 당신은 UI 컨트롤러와 ViewModel과 관련있는 Android Architecture Componets를 배울 것입니다.

<hr>

# Create the GameViewModel
 - 뷰모델은 UI와 관련 있는 데이터들을 저장하기 위해 디자인 된 클래스입니다.
 - 이번 수업에서 사용할 어플에선 ViewModel들은 각각 하나의 fragment와 협동합니다.
 - 이번 실습에선 GameViewModel이라는 GameFragment를 위해 사용될 뷰모델을 추가할 것입니다. 또한, 뷰모델이 수명 주기 인식(lifecycle-aware)을 한다는 게 무슨 뜻인지 알게 될 것입니다.

# Add the GameViewModel class
 - ViewModel 사용을 위해 build.gradle(app)의 디펜던시를 추가합니다.

``` xml
//ViewModel
implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0'
```

 - screen/game에 GameViewModel 클래스를 만듭니다.
 - GameViewModel에서 ViewModel을 상속받습니다.
 - 뷰모델이 수명 주기 인식을 어떻게 하는지 자세히 보여주기 위해 init 블록에 로그를 추가합니다.

``` kotlin
class GameViewModel : ViewModel() {
   init {
       Log.i("GameViewModel", "GameViewModel created!")
   }
}
```

# Override onCleared() and logging
 - ViewModel은 협동하는 프래그먼트가 detached 될 때, 혹은 액티비티가 끝났을 때 destroyed 됩니다. 
 - ViewModel이 destroyed 되기 전에 ViewModel의 리소스를 정리하기 위해, onCleared() 콜백이 불립니다.

<br>

 - GameViewModel 클래스에서 onCleared() 메소드를 오버라이딩합니다.
 - onCleared 메소드 내부에 GameViewModel 클래스의 생명주기를 추적하기 위해, 로그 코드를 추가합니다.
  

``` kotlin
override fun onCleared() {
   super.onCleared()
   Log.i("GameViewModel", "GameViewModel destroyed!")
}
```

# Associate GameViewModel with the game fragment
 - 뷰모델은 UI 컨트롤러와 협동해야 합니다. 
 - 이 둘이 협동하기 위해서는 UI 컨트롤러 내부에, 뷰모델 참조자를 만들어야 합니다.
 - 이번 단계에서, 당신은 GameViewModel 참조자를 협동하는 UI 컨트롤러에 만들어야 합니다. 바로 GameFragment입니다.
<br>

 - GameFragment에 탑레벨에 gameViewModel 참조자를 선언합니다.

``` kotlin
class GameFragment: Fragment() {
    // Reference of GameViewModel
    private lateinit var viewModel: GameViewModel
}
```

# Initialize the ViewModel
 - 화면 회전과 같은 설정 변경이 일어나면 프래그먼트와 같은 UI 컨트롤러들은 다시 생성(re-created)됩니다. 그러나! 뷰모델의 인스턴스는 죽지 않습니다.
 - ViewModel 클래스를 사용하여 VieModel 객체를 생성하면, 프래그먼트가 재생성될때마다 ViewModel 객체가 재생성됩니다.
 - ViewModel의 객체를 만드는데 ViewModel 클래스를 쓰는 대신, ViewModelProvider를 사용합시다.

![](/image/Android%20Kotlin%20기본/ViewModel/03.png)

<span style="color : red;">중요한 사실 : ViewModel 객체를 만들 땐 항상 ViewModelProvider를 사용해서 만드세요.</span>

# ViewModelProvider가 하는 일
 - ViewModelProvider는 만약에, ViewModel 객체가 존재하면 이미 있는 객체를 리턴하고, 만약에 없다면 객체를 새로 만들어서 리턴합니다.(싱글톤?)
 - ViewModelProvider는 주어진 스코프(Activity or Fragment)와 관련된 ViewModel 객체를 만듭니다.
 - 만들어진 ViewModel은 스코프가 살아있는 동안 계속 살아있습니다. 예를 들어, ViewModel의 스코프가 프래그먼트일 때, 뷰모델은 프래그먼트가 detached될 때까지 살아있습니다.

<hr>

# 실습

 - GameFragment 클래스에 onCreateView() 내부에서 binding 변수 정의가 끝난 다음 코드에 아래 코드를 작성합니다.

``` kotlin
override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                            savedInstanceState: Bundle?): View? {

    // Inflate view and obtain an instance of the binding class
    binding = DataBindingUtil.inflate(
            inflater,
            R.layout.game_fragment,
            container,
            false
    )

    Log.i("GameFragment", "Called ViewModelProvider.get")
    viewModel = ViewModelProvider(this).get(GameViewModel::class.java)


    return binding.root
}
```

 - 앱을 실행하고 Play 버튼을 누르면 logcat에 아래와 같은 문장이 나올 것입니다.

  ``` terminal
    I/GameFragment: Called ViewModelProvider.get
    I/GameViewModel: GameViewModel created!
  ```

![](/image/Android%20Kotlin%20기본/ViewModel/04.png)

  <hr>

 - 안드로이드 가상 기기의 자동 회전 기능을 켭니다.
 - 화면을 회전 시키고 logcat을 보면 화면이 돌아갈 때마다 gameFragment가 destroyed 되고 re-created 되는 것을 볼 수 있습니다. 이로 인해 ViewModelProvider.get()이 계속 불리는 것을 볼 수 있지만, GameViewModel 생성은 한 번만 되는 것을 볼 수 있습니다.

![](/image/Android%20Kotlin%20기본/ViewModel/05.png)

 - 이 상태에서 뒤로 가기를 눌러 게임 화면을 누르면 GameFragment와 GameViewModel이 destroyed 되는 것을 볼 수 있습니다. 또한 ViewModel이 종료 되기 전에 불리는 onCleared 메서드가 불리는 것도 볼 수 있습니다.

``` console
I/GameFragment: Called ViewModelProvider.get
I/GameViewModel: GameViewModel created!
I/GameFragment: Called ViewModelProvider.get
I/GameFragment: Called ViewModelProvider.get
I/GameFragment: Called ViewModelProvider.get
I/GameViewModel: GameViewModel destroyed!
```

# Populate the GameViewModel
 - ViewModel이 설정 변경에서도 살아남았으므로, ViewModel에는 설정 변경해도 유지될 데이터를 넣는데 좋을 것입니다.
 - 화면에 표시될 데이터를 넣고, 그 데이터에 작성할 코드를 작성합니다.
 - ViewModel에서는 절대 프래그먼트, 액티비티, 뷰를 참조하면 안됩니다. 왜냐하면 이러한 요소들은 설정 변경(화면 회전) 후에도 살아남지 못하기 때문입니다.

![](/image/Android%20Kotlin%20기본/ViewModel/06.png)

 - 이번 실습에선 앱의 UI 데이터와 데이터에 접근하는 메소드를 GameViewModel 클래스로 옮길 것입니다. 이렇게 함으로서 설정 변경이 일어나도 데이터의 유지가 이루어집니다.

<hr>

 # Move data fields and data processing to the ViewModel
  - GameFragment에 있는 word, score, wordList를 GameViewModel로 옮깁니다.
  - binding 변수는 옮기지 않습니다. 이 변수는 레이아웃과 클릭리스너를 위해 사용됩니다.
  - resetList(), nextWord() 메서드를 GameViewModel에 init 블록으로 옮깁니다.
  - 이 메소드들은 무조건 init 블록에 있어야합니다. 왜냐하면 wordList의 리셋은 ViewModel이 초기화 됐을 때 되어야하지, fragment가 만들어질 때마다 리셋되면 안되기 때문입니다.

 - 리팩토링 후에 코드

``` kotlin
//GameViewModel
class GameViewModel : ViewModel() {
   // The current word
   var word = ""
   // The current score
   var score = 0
   // The list of words - the front of the list is the next word to guess
   private lateinit var wordList: MutableList<String>

   /**
    * Resets the list of words and randomizes the order
    */
   private fun resetList() {
       wordList = mutableListOf(
               "queen",
               "hospital",
               "basketball",
               "cat",
               "change",
               "snail",
               "soup",
               "calendar",
               "sad",
               "desk",
               "guitar",
               "home",
               "railway",
               "zebra",
               "jelly",
               "car",
               "crow",
               "trade",
               "bag",
               "roll",
               "bubble"
       )
       wordList.shuffle()
   }

   init {
       resetList()
       nextWord()
       Log.i("GameViewModel", "GameViewModel created!")
   }
   /**
    * Moves to the next word in the list
    */
   private fun nextWord() {
       if (!wordList.isEmpty()) {
           //Select and remove a word from the list
           word = wordList.removeAt(0)
       }
       updateWordText()
       updateScoreText()
   }
 /** Methods for buttons presses **/
   fun onSkip() {
       score--
       nextWord()
   }

   fun onCorrect() {
       score++
       nextWord()
   }

   override fun onCleared() {
       super.onCleared()
       Log.i("GameViewModel", "GameViewModel destroyed!")
   }
}
```

``` kotlin
//GameFragment
/**
* Fragment where the game is played
*/
class GameFragment : Fragment() {


   private lateinit var binding: GameFragmentBinding


   private lateinit var viewModel: GameViewModel


   override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                             savedInstanceState: Bundle?): View? {

       // Inflate view and obtain an instance of the binding class
       binding = DataBindingUtil.inflate(
               inflater,
               R.layout.game_fragment,
               container,
               false
       )

       Log.i("GameFragment", "Called ViewModelProvider.get")
       viewModel = ViewModelProvider(this).get(GameViewModel::class.java)

       binding.correctButton.setOnClickListener { onCorrect() }
       binding.skipButton.setOnClickListener { onSkip() }
       updateScoreText()
       updateWordText()
       return binding.root

   }


   /** Methods for button click handlers **/

   private fun onSkip() {
       score--
       nextWord()
   }

   private fun onCorrect() {
       score++
       nextWord()
   }


   /** Methods for updating the UI **/

   private fun updateWordText() {
       binding.wordText.text = word
   }

   private fun updateScoreText() {
       binding.scoreText.text = score.toString()
   }
}
```

# Update references to click handlers and data fields in GameFragment
 - GameFragment에 onSkip(), onCorrect 메서드의 내용을 수정합니다.
 - score 관련 코드들을 지우고, viewModel의 onSkip()과 onCorrect()를 추가합니다.
 - onSkip()과 onCorrect() 밑에는 화면에 데이터를 업데이트해주는 updateScoreText(), updateWordText() 메서드를 추가해줍니다.

``` kotlin
private fun onSkip() {
   viewModel.onSkip()
   updateWordText()
   updateScoreText()
}
private fun onCorrect() {
   viewModel.onCorrect()
   updateScoreText()
   updateWordText()
}
```

 - GameFragment에 update 메소드들이 ViewModel의 데이터들을 입력 받아야 하므로 코드를 수정합니다.

``` kotlin
//(GameFragment의 수정 전 코드)
private fun updateWordText() {
    binding.wordText.text = word
}

private fun updateScoreText() {
    binding.scoreText.text = score.toString()
}
```

``` kotlin
//(GameFragment의 수정 후 코드)
private fun updateWordText() {
   binding.wordText.text = viewModel.word
}

private fun updateScoreText() {
   binding.scoreText.text = viewModel.score.toString()
}
```

### 명심하세요! ViewModel은 액티비티나 프래그먼트 뷰를 참조하면 안됩니다. 설정 변경이 되면 VIewModel은 살아남지만, 얘네들은 죽습니다.

 - GameViewModel에서 nextWord()에서 updateWordText()와 updateScoreText()를 지워줍니다. 이 메서드들은 이제 GameFragment에서 실행됩니다.

``` kotlin
//GameViewModel의 nextWord() 메소드.
private fun nextWord() {
    if (!wordList.isEmpty()) {
        //Select and remove a word from the list
        word = wordList.removeAt(0)
    }
}

```

# Implement click listener for the End Game button
 - 이번 실습에선 End Game 버튼을 위한 리스너를 설정할 것입니다.
 - GameFragment에 들어가서 onEndGame() 메소드를 추가합니다. 이 메소드는 End Game 버튼이 클릭 되면 불릴 메소드입니다.


``` kotlin
private fun onEndGame() {
        
}
```

 - GameFragment로 들어가서 endGame 버튼에 클릭 리스너를 설정해 클릭 되면 onEndGame() 메서드가 실행되게 합니다.

``` kotlin
binding.endGameButton.setOnClickListener { onEndGame() }
```

 - GameFragment에 점수화면으로 이동시킬 gameFinished() 메서드를 추가합니다. 매개변수는 Safe Args를 사용해서 넘깁니다.

``` kotlin
/**
* Called when the game is finished
*/
private fun gameFinished() {
   Toast.makeText(activity, "Game has just finished", Toast.LENGTH_SHORT).show()
   val action = GameFragmentDirections.actionGameToScore()
   action.score = viewModel.score
   NavHostFragment.findNavController(this).navigate(action)
}
```

- onEndGame() 메소드에서 gameFinished() 메소드를 부르세요.
``` kotlin
private fun onEndGame() {
   gameFinished()
}
```

 - 이제 앱을 실행하고 endGame을 누르면, 화면이 이동하고 점수도 이동합니다.


![](/image/Android%20Kotlin%20기본/ViewModel/07.png)
![](/image/Android%20Kotlin%20기본/ViewModel/08.png)

# Use a ViewModelFactory
 - 게임이 끝났을 때 ScoreFragment에서 점수를 보여주지 않습니다.
 - ViewModel이 ScoreFragment에서 보여줄 점수를 가지고 있길 원합니다.
 - [factory method pattern](https://en.wikipedia.org/wiki/Factory_method_pattern)을 사용하여, ViewModel 초기화 중에 점수 값을 전달합니다.

 - 팩토리 메소드 패턴은 [creational design patter](https://en.wikipedia.org/wiki/Creational_pattern)으로 팩토리 메소드를 사용하여 객체를 생성합니다. 
 - 팩토리 메소드는 클래스의 똑같은 객체를 리턴하는 메소드입니다.

<br>

 - 이번 실습에서는 ScoreFragment에 대한 매개 변수화된 생성자와 ViewModel을 인스턴스화하는 팩토리 메서드를 사용하여 ViewModel을 만듭니다.

<hr>

 - score 패키지로 들어가서 ScoreViewModel 클래스를 만듭니다.
 - 이 클래스는 ScoreFragment에 ViewModel이 될 것입니다.
 - ViewModel을 상속 받고, score를 위한 생성자를 추가합니다.
 - init 블록을 추가하고 내부에 로그를 작성하는 코드를 작성합니다.
 - 탑 레벨에 score 변수를 선언합니다.

``` kotlin

package com.example.android.guesstheword.screens.game

import android.util.Log
import androidx.lifecycle.ViewModel

class ScoreViewModel(finalScore: Int): ViewModel() {
    private val TAG: String = "로그"
    private val score = finalScore

    init {
        Log.d(TAG,"ScoreViewModel - () called")
    }
}

```

 - score 패키지에 ScoreViewModelFactory 클래스를 만듭니다. 이 클래스는 ScoreViewModel 객체를 만드는 역할을 가집니다.
 - ViewModelProvider.Factory 클래스를 상속 받습니다. 
 - 점수를 위한 생성자 매개변수를 추가합니다.

``` kotlin
class ScoreViewModelFactory(private val finalScore: Int): ViewModelProvider.Factory() {
}
```

 - create() 메소드를 오버라이딩합니다. create() 메소드 내부에는 새로 생성한 ScoreViewModel을 리턴하는 코드를 작성합니다.

``` kotlin
class ScoreViewModelFactory(private val finalScore: Int): ViewModelProvider.Factory() {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(ScoreViewModel::class.java)) {
            return ScoreViewModel(finalScore) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```
 - 위에 코드 중 isAssginableFrom 메서드는 특정 Class가 어떤 클래스/인터페이스를 상속/구현했는지 체크합니다. 위에서는 modelClass가 ScoreViewModel 클래스를 상속 받거나 구현했으면, ScoreViewModel을 만들어서 리턴합니다. (isAssignableFrom과 instanceof의 차이점)[https://jistol.github.io/java/2017/08/22/different-instanceof-isassignablefrom/]

<br>

 - ScoreFragment로 이동해서 ScoreViewModel과 ScoreViewModelFactory를 위한 변수를 선언합니다.

``` kotlin
private lateinit var viewModel: ScoreViewModel
private lateinit var viewModelFactory: ScoreViewModelFactory
```

 - ScoreFragment에서 onCreateView 내부에서 viewModelFactory를 초기화합니다. 이전에 Safe Args로 넘긴 값을 이용해서 값을 받아, ViewModelFactory를 만듭니다.

``` kotlin
viewModelFactory = ScoreViewModelFactory(ScoreFragmentArgs.fromBundle(requireArguments()).score)
```

 - viewModelFactory를 만들었으니 이제 viewModel 객체를 초기화합니다. ViewModelProvider.get() 메소드를 사용하여, scoreFragment의 context와 방금 초기화시킨 viewModelFactory를 이용합니다.
 - 이 작업은 viewModelFactory 내부에 있는 팩토리 메소드를 사용하여 ScoreViewModel 클래스를 만들 것입니다.

``` kotlin
viewModel = ViewModelProvider(this, viewModelFactory)
       .get(ScoreViewModel::class.java)
```

 - 이제 화면에 데이터를 뿌려주는 일만 남았습니다.
 - viewModel에 score를 scoreText에 뿌려줍니다.
 - 
``` kotlin
binding.scoreText.text = viewModel.score.toString()
```
 - 사실 방금 실습에선 viewModel을 팩토리를 사용하서 초기화할 필요가 없습니다.
 - 그저 viewModel.score로 접근해서 번들 값을 가져왔으면 됐습니다.
 - 그러나 viewModel이 만들어지면서 값이 바로 할당되어야 하는 경우가 있습니다. 이를 위해서 팩토리 메소드를 사용한 것입니다.

<hr>

<br>

``` kotlin
// 팩토리 메소드를 사용하지 않고 viewModel 만들기.
viewModel = ViewModelProvider(this).get(GameViewModel::class.java)
```

``` kotlin
// 팩토리 메소드를 사용하여 viewModel 만들기.
viewModel = ViewModelProvider(this, viewModelFactory)
       .get(ScoreViewModel::class.java)
```
