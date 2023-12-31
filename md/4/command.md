## 명령 패턴
- 디자인 패턴을 사용하면 객체 내부에 동작을 캡슐화해서 넣어 둔 뒤 나중에 실행되도록 할 수 있음
- 동작 실행을 지연시키면 한꺼번에 많은 동작을 실행할 수도 있고, 심지어 동작의 실행 타이밍을 세밀하게 조절할 수 있음
- 3장의 트루퍼 관리 시스템 예시로 돌아가본다
- 앞의 예제에서 attack과 move함수를 다음과 같이 구현한다고 가정해보자
```
class Stormtrooper(...){
    fun attack(x : Long, y: Long) {
        println("($x, $y) 공격 중")
        // 여기에 실제 코드 구현
    }
    
    fun move(x: Long, y: Long) {
        println("($x, $y)로 이동 중")
        // 여기에 실제 코드 구현
    }
}
```
- 실제 구현을 위해 3장에서 배운 브리지 패턴을 사용할 수 있음
- 이제 트루퍼가 한 번에 기억할 수 있는 명령이 하나밖에 되지 않는다는 문제를 해결해야 한다
- 간단히 말해 다음과 같다.
- 예를 들어 어떤 트루퍼가 화면의 좌상단을 나타내는 (0,0)에 있다고 한다
- 만약 (20,0)으로 이동하라는 move(20,0) 명령과 (20,20)으로 이동하라는 move(20,20) 명령을 연달아 내리면 이 객체는 (0,0)에서 (20,20)으로 직선으로 이동할 것임
- 결국 반란군을 만나 전멸할지도 모름
```
[스톰트루퍼](0,0) -> 기대하는 이동 방향 -> (20,0)

    [반란군] [반란군]
[반란군] [반란군] [반란군]
    [반란군] [반란군]
        (5,20)                       (20,20)
```
- 책을 처음부터 계속 읽었거나 최소한 4장을 처음부터 읽고 있다면 어떻게 해야 할지 아이디어가 떠오를 것
- 코틀린에는 함수가 일급 객체라는 사실을 이미 설명헀기 때문
- 간단한 스케치를 해보면 명령을 하나밖에 기억하지 못하는 것이 문제면 이를 해결하기 위해서 무언가의 리스트를 들고 있어야 한다는 것을 알 수 있음
- 아직 어떤 타입의 객체가 돼야 할지는 모름
- 아래는 Any 타입을 사용한 것
```
class Trooper {
    private val orders = mutableListOf<Any>()

    fun addOrder(order: Any){
        this.orders.add(order)
    }
}
```
- 다음으로 리스트에 대해 반복을 수행하며 명령을 실행하기를 원함
```
class Trooper{
    ...
    // 외부 코드에서 가끔식 실행하는 함수
    fun executeOrders(){
        while (orders.isNotEmpty()){
            val order = orders.removeFirst()
            order.execute()
        }
    }
}
```
- 코틀린에는 집합 자료 구조가 비어 있지 않은지 검사하는 isNotEmpty() 함수가 있어서 !orders.isEmpty() 대신 사용할 수 있음
- 또한 집합 자료 구조를 마치 큐 queue 처럼 사용할 수 있도록하는 removeFirst() 함수도 있음
- 명령 디자인 패턴이 낯설더라도 이 코드가 동작하도록 하려먼 execute()라는 함수 하나를 갖는 인터페이스를 구현하면 된다는 것 정도는 짐작할 수 있을 것임
```
interface Command{
    fun execute()
}
```
- 그러면 멤버 변수는 Command 객체의 리스트가 됨
```
private val commands = mutableListOf<Command>()
```
- 어떤 종류의 명령이든 (이동이든 공격이든 관계없이) 이 인터페이스를 필요에 맞게 구현한다
- 이것이 일반적으로 자바에서 명령 패턴을 구현하는 방식
- 더 좋은방법은 없나
- command 인터페이스를 다시 들어다본다.
- execute() 메서드는 인수도 반환값이 없고, 어떤 동작을 할 뿐임
- 다음과 같이 코드를 작성하는 것과 동일
```
fun command(): Unit{
    // 어떤 동작을 하는 코드
}
```
- 이전에 봤던 코드와 동일
- 다음과 같이 더욱 간결하게 작성할 수 있음
```
() -> Unitq
```
- 이제 Command라고 하는 인터페이스 대신 typealias를 사용한 별칭을 만들 것
```
typealias Command = () -> Unit
```
- 이제 Command 인터페이스는 필요 없어 졌으므로 지워도됨
- 이번엔 다음 코드가 컴파일 되지 않는다
```
command.execute() // unresolved reference: execute
```
- execute() 는 직접 인터페이스를 정의할 때 붙여 줬던 이름이기 때문
- 코틀린에서 함수 객체를 호출할 때는 invoke()를 사용함
```
command.invoke()
```
- invoke()를 생락할 수 있다. 그럼 다음과 같이 됨
```
fun executeOrders() {
    while(orders.isNotEmpty()){
        val order = orders.removeFirst()
        order()
    }
}
```
- 썩 괜찮다. 하지만 현재 Command는 매개변수를 받을 수 없음
- 함수에 인수가 있으면 어떻게 되나?
- Command의 시그니처를 변경해서 2개의 매개변수를 받도록 하는 것도 한가지 방법
```
(x: Int, y: Int) -> Unit
```
- 그러나 만약 어떤 명령은 인수를 받지 않고, 어떤 명령은 하나나 둘, 또는 그 이상의 인수를 받는다면? 각 명령이 invoke()에 무엇을 전달해야 하는지 기억해야 할 것
- 더 좋은 방법이 있음
- 함수 생성기(function generator)를 사용하는 것이다.
- 함수 생성기란 다른 함수를 반환하는 함수를 뜻함
- 자바스크립트를 사용해 본 적이 있다면 시야를 제한하고 상태를 기억하기 위해 클로저를 사용하는 것이 일반적이라는 점을 알 것임
```
val moveGenerator = fun(trooper: Trooper, 
                        x: Int,
                        y: Int
                        ): Command{
        return fun(){
            trooper.move(x,y)
        }
    }
```
- 적절한 인수를 전달하면 moveGenerator함수는 또 다른 함수를 반환함
- 이 함수는 알맞은 때에 언제든 호출할 수 있으며, 다음의 세가지를 기억하고 있음
  - 어떤 메서드를 호출할지
  - 어떤 인수를 사용할지
  - 어떤 객체에 대해서 사용할지
- 이제 Trooper 클래스에는 다음과 같은 메서드가 필요할 것임
```
fun appendMove(x: Int, y: Int) = apply{
    command.add(moveGenerator(this, x, y))
}
```
- 이 함수를 사용하면 다음과 같이 유창한 문법을 사용해서 아름다운 코드를 작성할 수 있음

```
val trooper = Trooper trooper.appendMove(20,0)
                             .appendMove(20,20)
                             .appendMove(5, 20)
                             .execute()
```
- 유창한 문법이란 동일한 객체에 대해 메서드를 사슬처럼 계속 호출함으로써 객체 이름을 여러번 반복하지 않도록 하는것을 의미함
- 위의 코드를 실행하면 다음과 같은 출력이 나올 것
```
>(20,0)로 이동중
>(20,20)로 이동중
>(5,20)로 이동중
```
- 이제 트루퍼 객체가 내부적으로 명령을 어떻게 실행하는지에 관계없이 얼마든지 많은 명령을 내릴 수 있음
- 다른 함수를 입력으로 받거나 반환하는 함수를 고차함수라고 함
- 고챃마수는 책 전반에서 여러번 등장함

### 명령 실행 취소
- 지금까지의 내용과 직접적인 관련은 없지만, 명령 디자인 패턴의 장점 중 하나는 명령 실행을 되돌릴 수 있다는 것
- 이런 실행 취소를 구현하고 싶다면 어떻게 해야 하나?
- 실행 취소는 일반적으로 상당히 까다로운 기능임
- 다음 세 가지 중 하나에 해당하기 때문
  - 이전 상태를 반환(클라이언트가 여럿인 경우에는 많은 메모리가 필요하기 때문에 불가능)
  - 차이점을 계산(구현하기 까다로움)
  - 역연산을 정의(항상 가능하지는 않음)
- 예제에서 '(0,0)에서 (0,20)으로 이동' 명령의 반대는 어딘가에서 (0,0)으로 이동'일 것이다
- 다음과 같이 명령의 쌍을 저장해서 이를 구현할 수 있음
```
private val commands = mutableListOf<Pair<Command, Command>>()
```
- 이제 appendMove함수를 수정해서 항상 반대 명령을 함께 저장하도록 한다
```
fun appendMove(x: Int, y: Int) = apply{
    val oppsiteMove = /* 첫 번째 명령이라면 현재 위치로 이동하는 명령을 생성함.
    그렇지 않으면 이전명령을 가져옴 */
    commands.add(moveGenerator(this, x, y) to oppsiteMove)
}
```
- 지금은 트루퍼의 위치를 저장하고 있지 않기 때문에(이것이 주어진 제한 조건이었음)
- 반대로 이동하는 명령을 생성하는 것은 꽤 복잡한 일임
- 경계 조건까지도 고려해서 구현해야 함
- 하지만 어쨋든 어떤 식으로 구현할지 힌트는 얻을 수 있었을 것임
- 명령 디자인 패턴도 언어 자체에 녹아든 기능 중 하나이다.
- 이 경우 함수가 일급 객체라는 사실 덕분에 디자인 패턴에 직접 구현하지 않아도 됨
- 실무에서는 여러 동작을 대기열에 넣어 놓거나 나중에 실행되도록 스케줄링 할 때 명령 디자인 패턴을 유용하게 사용할 수 있음


