- 코틀린 디자인패턴 책

## 1장 코틀린 시작하기

### 명명규칙
- 우선 코틀린은 .kt 로 끝나는 소스이다
- 파일명은 클래스명과 동일하게 짓는다 패턴은 그뒤에
- 파일명에는 카멜 케이스를 사용한다(책에서 나온 내용임)
  * 카멜케이스란? : 맨 앞 단어기준문자제외 그이후 단어단위 앞쪽에 대문자를 사용하여 이름을 읽기 쉽게 지음
- 메인은 대부분 Main.kt 임

## 패키지
- 비슷한 목적 또는 도메인을 공유하는 파일과 클래스의 묶음
- 자바랑 비슷하게 아래와 같이 키워드를 시용
```
//package 대단위.중단위.소단위
```
- 자바와 동일하게 간다고 생각하면 됨
- Main.kt 에서는 패키지를 선언하지 않아도 됨

### 주석
// 한줄
/**/ 여러줄

### Hello Kotlin
```
//국룰 소스코드

fun main(){ // 코틀린은 fun 키워드로 함수 사용
	println(“hello kotlin”) // 자바처럼 앞 패키지 선언은 안해도 된다
}
```
### 감싸는 클래스가 없음
- 코틀린에는 패키지 수준 함수라는 개념이 있음
- 클래스 속성에 접근할 필요가 없는 함수라면 굳이 클래스로 감싸지 않아도 된다는 것
- 책의 뒷부분에서 패키지 수준 함수에 대해 더 자세히 살펴볼 것

### 명령줄 인수가 없음
```
public static void main(String [] args) {…} // 명령줄 인수 선언코드
```

### static 키워드가 없음
- 어떤 언어에서 클래스를 인스턴스화 하지 않고도 실행할 수 있는 함수를 나타내기 위해 static을 사용
- 코틀린에서는 상태를 갖지 않는 함수는 클래스 밖에 두면 됨
- 그래서 코틀린에는 static 키워드가 없음

### 더 간결한 출력함수
- println() //ㅈㄱㄴ

### 세미 콜론이 없음
- 자바나 다른 언어에서는 세미콜론 사용하는 빈도가 많다
코틀린은 세미콜론 없음