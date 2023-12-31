# 구조 패턴 이해하기
- 3장은 구조패턴을 활용하는 방법을 설명함
- 구조 패턴이란 쉽게말해 객체 간 관계를 다루는 디자인 패턴
- 먼저 클래스 사이의 복잡한 계층 관계를 만들지 않고도 객체의 기능을 확장하는 법을 알아낼 것
- 그리고 미래의 변경 사항을 반영하여 과거의 설계를 수정하는 방법을 살펴보고 마지막으로는 메모리 사용량을 줄이는 방법도 다룸

## 해당 장에서 다루는 디자인 패턴 종류
- 데코레이터 패턴
- 어댑터 패턴
- 브리지 패턴
- 합성 패턴
- 퍼사드 패턴
- 경량패턴
- 프록시 패턴
  
#### 예제코드 사이트
- https://github.com/PacktPublishing/Kotlin-Design-Patterns-and-Best-Practices/tree/main/Chapter03

## 데코레이터 패턴
- 2장에서는 프로토타입 패턴을 살펴봄
- 프로토타입 패턴을 사용하면 같은 클래스의 인스턴스이면서 속성이 조금 다른 (때로는 많이 다른) 객체를 쉽게 생성할 수 있다
- 그렇다면 이런 질문도 해볼 수 있음
- 속성이 아닌 동작이 조금씩 다른 클래스를 여럿 만들어야 한다면 어떻게 할 것인가?
- 코틀린에서는 함수가 일급 객체(4장에 나옴)이기 때문에 프로토 타입 디자인 패턴을 사용해서도 이 목적을 달성할 수 있음
- 자바스크립트도 이런 방법으로 동작이 조금씩 다른 여러 클래스를 만들어냄
- 하지만 3장에서는 조금 다른 접근법을 설명하려고 하는데 어쨰든 디자인 패턴은 여러 접근법을 배우기 때문임
- 데코레이터 디자인 패턴을 구현하면 코드 사용자가 어떤 기능을 추가할지 자유롭게 선택할 수 있음

### 클래스에 기능 추가하기
- 스타트랙 시리즈의 모든 선장과 함선을 등록하는 간단한 클래스가 다음과 같이 구현돼 있다고 생각
```
open class StarTrekRepository{
    private val starshipCaptains = mutableMapOf("USS 엔터프라이즈" to "장뤽 피카드")

    open fun getCaptain(starshipName: String): String{
        return starshipCaptains[starshipName]? : "알 수 없음"
    }

    open fun addCaptain(starshipName: String, captainName: String) {
        starshipCaptains[starshipName] = captainName
    }
}
```
- 어느날 관리자가 긴급 요구 사항을 갖고 옴
- 이제부터 선장을 검색하면 반드시 콘솔에 로그를 남겨달라고 한 것
- 간단해 보이는 작업이지만 제약이 있는데 starTrekRepository 클래스를 직접 수정하면 안된다는 것
- 다른 사람도 이 클래스를 사용하는데 그들은 로그 기록은 뤈치않기 때문임
- 이 문제를 파고들기 전에 먼저 이 클래스 코드에서 눈에 띄는 부분, 즉 getCaptain 함수에 있는 이상한 연산자를 짚고 넘어가자

### 엘비스 연산자
- 코틀린은 강한 타입 언어 일 뿐만 아니라 null 안전성도 보장한다는 것을 배웠음
- 만약 위의 예제에서 특정한 키에 대응하는 값이 맴에 저장돼 있지 않다면 어덯게 될까?
- 맵을 사용한다면 코틀린 맵에 내장된 getOrDefault 메서드를 사용하는 것도 한 가지 방법임
- 이 코트에서 맵을 사용하기 때문에 이 방법을 쓸 수 있지만 null 값을 다룰 때 일반적으로 사용할 수 있는 방법은 아님
- 다른 방법은 엘비스 연산자(?:)를 이용하는 것
- 왜 이런 이름이 붙었는지 궁금하다면 엘비스 프레슬리의 머리 스타일을 떠올려보라
- 엘비스 연산자의 목적은 null 값을 받았을 때 사용할 기본 값을 지정하는 것임
- getCaptain 함수를 다시 보면 무슨 뜻인지 이해할 수 있을 것
- 엘비스 연산잘르 사용하지 않고 같은 기능을 구현하면 다음과 같은 코드가 됨
```
return if(starshipCaptains[starshipName] == null)
    "알 수 없음" else starshipCaptains[starshipName]
```
- 엘비스 연산자 덕분에 코드 길이가 많이 줄어든 것을 알 수 있음

### 상속의 문제점
- 다시 관리자의 긴급 요구사항으로 StarTrekRepository 클래스와 이에 속한 메서드가 모두 open으로 선언돼 있기 때문에 클래스를 상속받고 필요한 함수를 다음과 같이 덮어 쓸 수 있음
```
class LoggingGetCaptainStarTrekRepository : StarTrekRepository(){
    override fun getCaptain(starshipName: String): String {
        println("$starshipName 함선의 선장을 조회 중입니다")
        return super.getCaptain(starshipName)
    }
}
```
- 어렵진 않고, 클래스 이름이 좀 긴게 아쉬움
- super 클래스를 사용하여 구현의 일부를 부모 클래스에 위임함
- 그런데 다음날 관리자가 와서 이번에는 선장을 추가할 때 이름이 일곱글자를 넘지 않는지 검사하는 기능을 추가해달라고 함
- 그럼 클린온족 선장중에는 추가할 수없는 경우도 생기겠지만 무튼 기능은 구현하기로함
- 이 기능은 이전에 구현한 로그 기능과는 독립적이어야 한다
- 로그 기록만 하고싶을 때도 있고 문자열 길이 검사만 하고 싶을 때도 있기 때문이다
- 이런 요구사항을 만족시키는 새 클래스는 다음과 같이 구현됨
```
class ValidatingAddCaptainStarTrekRepository : StarTrekRepository(){
    override fun addCaptain(starshipName: String, captainName: String) {
        if (captainName.length > 7){
            throw RuntimeException("${captainName}의 이름이 7자가 넘음")
        }
        super.addCaptain(starshipName, captainName)
    }
}
```
- 또하나의 요구사항 변경을 잘 넘겼다
- 하지만 다음날 또 다른 요구사항이 접수됐는데 어떤 경우에는 로그 기록과 문자열 길이 검사를 동시에 수행할 수 있도록 해야 한다는 것임
- 이제 클래스 이름을 LoggingGetGaptainValidatingAddAcaptainStarTrekRepository로 지어야 할 것 같음
- 이런 종류의 문제는 의외로 매우 흔하게 볼 수 있는데, 곧 디자인 패턴이 필요하다는 뚜렷한 징후임
- 데코레이터 패턴의 목적은 객체에 새로운 동작을 동적으로 추가하는 것
- 위의 예제에서는 '로그기록'과 '문자열 길이 검사'가 선택적으로 적용돼야 할 객체의 동작에 해당함
- 먼저 StarTrekReposptory를 인터페이스로 변경해보자
```
interface StarTrekRepository{
    fun getCaptain(starshipName: String): String
    fun addCaptain(starshipName: String, captainName: String)
}
```
- 이제 기존과 동일한 로직을 사용하여 인터페이스를 구현
```
class DefaultStarTrekRepository : StarTrekRepository{
    private val starshipCaptains = mutableMapOf("USS 엔터프라이즈" to "장뤽 피카드")
    override fun getCaptain(starshipName: String) : String {
        return starshipCaptains[starshipName] ?: "알 수 없음"
    }
    override fun addCaptain(starshipName: String, captainName: String) {
        starshipCaptains[starshipName] = captainName
    }
}
```
- 다음으로는 이 구체 클래스는 상속받는 대신 인터페이스를 구현하고 by라는 새로운 키워드를 사용할 것

```
class LoggingGetCaptain(private al repository: StarTrekRepository) : StarTrekRepository by repository{
    override fun getCaptain(starshipName: String): String{
        println("$starshipName 함선의 선장을 조회중")
        return repository.getCaptain(starshipName)
    }
}
```
- by 키워드는 인터페이스 구현을 다른 객체에 위임함
- 그래서 인터페이스에 선언된 함수를 하나도 구현할 필요가 없었던 것
- 이 인스턴스가 감싸고 있는 다른 객체가 기본적으로 모든 구현을 대신함
- 여기서는 클래스의 시그니처가 어떤 의미인지 주의깊게 살펴야 함
- 데커레이터 패턴을 구현할 때 필요한 요소는 다음과 같음
  - 데코레이션(새로운 동작)을 추가할 대상 객체를 입력으로 받음
  - 대상 객체에 대한 참조를 계속 유지함
  - 데코레이터 클래스의 메서드가 호출되면 들고 있는 대상 객체의 동작을 변경할 지 또는 처리에 위임할지 결정
  - 대상 객체에서 인터페이스를 추출하거나 또는 해당 클래스가 이미 구현되고 있는 인터페이스를 사용함
- 데코레이터의 메서드에서는 더이상 super키워드를 사용하지 않는다는 점에 유의할것
- super를 사용하려고 하면 오류가 발생할 것 클래스를 상속받은게 아니기 때문임
- 대신 들고 있는 대상 객체의 인터페이스 참조를 사용
- 확실한 이해를 위해 두 번째 데코레이터를 작성해보자
```
class ValidatingAdd(private val repository: StarTrekRepository):
StarTrekRepository by repository{
    private val maxNameLength = 7
    override fun addCaptain(starshipName: String, captainName: String){
        require(captainName.length < maxNameLength) {
            "${captainName}의 이름이 7자가 넘습니다!"
        }
        repository.addCaptain(starshipName, captainNameq)
    }
}
```
- 이 예제 코드가 ValidatingAddCaptainStarTrekRepository와 다른 점은 if 표현식 대신 require함수를 사용했다는 것 뿐임
- 이렇게 하면 식이 false일 때 IllegalArgumentException을 던지는 것은 똑같지만 가독성이 향상됨
- 구현한 클래스를 사용하는법을 살펴본다
```
val starTrekRepository = DefaultStarTrekRepository()
val withValidation = ValidatingAdd(starTrekRepository)
val withLoggingAndValidating = LoggingGetCaptain(withValidating)
withLoggingAndValidating.getCaptain("USS 엔터프라이즈")
withLoggingAndValidating.addCaptain("USS보이저", "캐서린 제인웨이")
```
- 마지막 줄은 다음과 같이 예외를 던질 것
```
캐서린 제인웨이의 이름이 7자를 넘는다
```
- 예제에서 볼 수 있듯 데코레이터 패턴을 사용하면 원하는 대로 동작을 조합할 수 있음
- 잠시 주제를 벗어나 코틀린의 연산자 오버로딩을 살펴본다
- 다음 디자인 패턴을 이해하는 데에 도움이 될 것

### 연산자 오버로딩
- 위의 예제에서 추출한 인터페이스를 다시 살펴본다
- 여기선 배열이나 맵에 접근하고 값을 할당할 때 주로 사용하는 기본 연산자를 설명할 것
- 코틀린에는 연산자 오버로딩이라는 편의 문법이 있음
- 코틀린에서 맵을 사용하는 것이 얼마나 직관적인지는 DefaultStarTrekRepository 코드를 보면 알 수 있음
```
starshipCaptains[starshipName]
starshipCaptains[starshipName] = captainName
```
- 그렇다면 직접 구현한 객체도 다음과 같이 맵처럼 사용할 수 있다면 편리할 것
```
withLoggingAndValidating["USS 엔터프라이즈"]
withLoggingAndValidating["USS 보이저"] = "캐서린 제인웨이"
```
- 코틀린에선 이는 그리 어려운 일이 아님
- 먼저 인터페이스를 다음과 같이 변경한다
```
interface StarTrekRepository{
    operator fun get(starshipName: String): String
    operator fun set(starshipName: String, captainName : String)
}
```
- 함수 선언부 앞에 operator 키워드를 추가헀다.
### Operator의 역할
- 대부분의 프로그래밍 언어는 어떤 식으로든 연산자 오버로딩을 지원함.
- 예를들어 자바에서는 다음과 같은 코드를 작성한다.
```
System.out.println(1+1)
System.out.println("1" + "1") // 11출력
```
- 인수가 문자열인지 정수인지에 따라 더하기 연산자(+)가 다르게 동작하는 것을 확인할 수 있음
- 즉 정수의 덧셈과 문자열 연결에 똑같은 연산자를 쓸 수 있다는 뜻
- 다른 타입에 대해 더하기 연산자를 정의하는 것도 상상해 봄직 하다
- 예를 들어 이 연산자로 리스트 2개를 연결할 수 있음
```
List.of("a") + List.of("b")
```
- 안타깝게도 자바에서 이 코드는 컴파일되지 않음
- 개발자가 딱히 할 수 있는 것은 없음
- 연산자 오버로딩은 자바 언어 자체의 기능이라 사용자가 확장할 수 없기 때문
- 반대쪽 극단에는 스칼라가 있음
- 스칼라에서는 아무 문자열이나 연산자가 될 수 있음
- 다음과 같은 코드를 마주칠 수 있음
```
Seq("a") === Seq("b") // 코드의 의미를 추측해야 한다.
```
- 코틀린은 자바와 스칼라의 중간정도에 있음
- 코틀린에서는 '잘 알려진' 연산 몇가지는 오버로딩할 수 있음
- 그러나 제하적으로만 가능함
- 비록 제한적이라고 하더라도 오버로딩이 가능한 연산자의 목록은 상당히 길기 때문에 책에 모두 옮기지는 않을 것
- 지원이 되지 않는 함수에 operator키워드를 사용하거나 잘못된 인수를 사용하면 컴파일 오류가 발생함
- 위의 코드 예제에서 사용한 대괄호 연산자는 첨자 접근 연산자라고도 하며 코드에서 볼 수 있듯이 get(x)와 set(x,y) 메서드에 대응

### 데코레이터 패턴 사용 시 주의사항
- 데코레이터 패턴은 강력하다. 즉석에서 객체를 만들어내기 때문인
- 코틀린의 by 키워드를 사용하면 구현하기도 어렵지 않다.
- 그러나 반드시 유념해야 할 데코레이터 패턴의 한계가 있음
- 데코레이터의 객체의 속을 볼 수 없다는 점 즉 속에 들어있는 객체가 무엇인지 알 길 없음
```
println(withLoggingAndValidating is LoggingGetCaptain)
//최상위 수준의 데코레이터, 여기는 문제 없음
println(withLoggingAndValidating is StarTrekRepository)
// 구현한 인터페이스 여기에도 문제 없음
println(withLoggingAndValidating is ValidatingAdd)
// 속에 들어있는 클래스. 하지만 컴파일러는 알 수 없다
println(withLoggingAndValidating is DefaultTrekRepository)
// 속에 들어있는 클래스. 하지만 컴파일러는 알 수 없다
```
- withLoggingAndValidating는 ValidatingAdd 객체를 감싸고 있고 동작도 ValidatingAdd와 같이 하지만 그 자체로 ValidatingAdd객체는 아님
- 그렇기 떄문에 캐스팅이나 타입 검사 시 주의해야 함
- 어떤 상황에서 이 패턴을 유용하게 사용할 수 있을지 궁금할 것
- java.io.* 패키지에서 Reader와 writer 인터페이스를 구현하는 클래스들을 예로 들 수 있음
- 가령 파일을 효율적으로 읽기 위해서 BufferedReader를 사용할 수 있는데, 이 클래스는 생성자에서 또 다른 Reader객체를 인수로 받음
```
val reader = BufferedReader(FileReader("/some/file"))
```
- FileReader는 Reader 인터페이스를 구현하므로 이렇게 사용할 수 있다.
- BufferedReader도 Reader의 구현체이기 때문에 FileReader 대신 BufferedReader객체를 넘기는 것도 가능함