# LiveData transformations

### 소개
 - LiveData를 컴포넌트 간에 전달할 때, 데이터를 형식화해서 전달하고 싶은 경우가 있습니다.
 - 예를 들어 현재 앱에서는 word보다는 word의 글자수를 보내고 싶은 경우가 있습니다.
 - 이를 해결하기 위해선 Transformations 클래스를 사용하면 됩니다.

 - 이번 코드랩에선 countDownTimer를 앱에 더하고, Transformations.map()을 사용하여 LiveData의 데이터를 형식화할 것입니다.

<hr>

### 사작하기 전에
 - 당신이 이전 실습을 따라오지 않았다면 \[[링크](https://github.com/google-developer-training/android-kotlin-fundamentals-apps/tree/master/GuessTheWordDataBinding)\]를 클릭하여 코드를 다운 받으세요.
 - 앱을 실행해서 가지고 놀아보세요.

<hr>

### 타이머 더하기
 - 이전까지는 게임 종료 버튼을 누르거나, 모든 문제를 풀면 게임이 종료 됐습니다.
 - 이번 실습에서는 타이머를 이용하여 시간이 끝나면 게임이 끝나도록 할 것입니다.
 - GameViewModel에 timer를 위한 companion object를 추가합니다.

``` kotlin
companion object {

   // Time when the game is over
   private const val DONE = 0L

   // Countdown time interval
   private const val ONE_SECOND = 1000L

   // Total time for the game
   private const val COUNTDOWN_TIME = 60000L

}

```

 - 타이머를 위한 시간을 저장하기 위한 라이브데이터를 추가합니다.

``` kotlin
// Countdown time
private val _currentTime = MutableLiveData<Long>()
val currentTime: LiveData<Long>
   get() = _currentTime
```

 - CountDownTimer를 위한 timer 변수를 선언합니다.

``` kotlin
private val timer: CountDownTimer
```

 - init 블록 내부에서 timer를 생성합니다. COUNTDOWN_TIME을 총시간, 주기를 ONE_SECOND로 부여합니다.
 - 콜벡 메소드인 onTick()과 onFinish()를 오버라이딩합니다. 그리고 timer를 시작합니다.

``` kotlin
// Creates a timer which triggers the end of the game when it finishes
timer = object : CountDownTimer(COUNTDOWN_TIME, ONE_SECOND) {

   // 매 주기마다 실행할 로직
   // millisUntilFinished는 타이머가 끝날 때까지 남은 시간.
   override fun onTick(millisUntilFinished: Long) {
       // 매 초마다 라이브데이터의 값을 남은시간으로 바꿔줍니다.
       // 나눠준 이유는 초 단위로 표현하기 위해서입니다.
       // 1000ms = 1s
       _currentTime.value = millisUntilFinished/ONE_SECOND
   }

   // 타이머가 마지막에 실행할 메소드
   override fun onFinish() {
       // 타이머가 끝나면서 시간은 끝나는 시간인 DONE이 됩니다.
       _currentTime.value = DONE
       // 타이머가 끝났으므로 게임이 끝나게 하기 위해 메서드를 부릅니다.
       onGameFinish()
   }
}

timer.start()
```

 - 단어 목록이 비었을 때, 다시 단어를 채워주기 위한 로직을 nextWord()에 작성합니다.

``` kotlin
private fun nextWord() {
   // Shuffle the word list, if the list is empty 
   if (wordList.isEmpty()) {
       resetList()
   } else {
   // Remove a word from the list
   _word.value = wordList.removeAt(0)
   }
}
```

 - onCleared() 메소드 내부에 타이머를 cancel하는 메소드를 추가합니다. 이는 메모리 누수를 방지합니다.
 - onCleared() 메소드는 ViewModel이 없어지기 전에 불립니다.

``` kotlin
override fun onCleared() {
   super.onCleared()
   // Cancel the timer
   timer.cancel()
}
```
 - 앱을 실행하고 60초를 기달리면 게임이 종료 되는 것을 볼 수 있습니다. 하지만, 텍스트에 시간이 표시 되지 않습니다. 이는 다음 과정에서 고칠 것입니다.

<hr>

### LiveData를 위한 transformation 추가하기
 - Transformations.map() 메소드는 LiveData를 조작하고, LiveData 객체를 리턴합니다.
 - 단, observer가 리턴하는 LiveData를 관찰하고 있을 때만 데이터를 조작합니다.
 - Transformations.map() 메소드는 매개변수로 LiveData 객체와 function을 둡니다. 이 때 매개변수로 오는 함수가 LiveData 객체를 조작합니다.

 - 주의할 것 : Transformation.map()의 매개변수로 오는 람다 함수는 메인 쓰레드에서 실행됩니다. 그러므로 장기작업은 포함하지 마세요.

 - 이번 실습에선, LiveData를 "MM:SS"형식으로 반환할 것입니다. 또한 반환된 시간을 화면에 표시할 것입니다.
 - GameViewModel로 들어가서 currentTimeString 변수를 선언합니다. 이 변수는 currentTime을 "MM:SS"에 맞게 변환하여 저장할 것입니다.
 - currentTImeString을 정의하기 위해, Transformations.map()을 작성합니다. 

``` kotlin
// formatted time
val currentTimeString = Transformations.map(currentTime) { time ->
    DateUtils.formatElapsedTime(time)
}
```

 - 뷰에 시간을 표시하기 위해, game_fragment로 들어가서 text를 수정해줍니다.

``` xml
<TextView
   android:id="@+id/timer_text"
   ...
   android:text="@{gameViewModel.currentTimeString}"
   ... />
```

 - 앱을 실행하면 시간이 제대로 표시 되는 것을 볼 수 있습니다.

![](/image/Android%20Kotlin%20기본/LiveDataTransformations/01.png)

### 완성본

\[[링크](https://github.com/google-developer-training/android-kotlin-fundamentals-apps/tree/master/GuessTheWordTransformation)\]