---
layout: post
date: 2024-06-30
title: "코틀린의 scope function - 20강"
tags: [Kotlin, Lecture, ]
categories: [Kotlin, ]
---


> 인프런에서 제공하고 있는 강의를 보고 정리한 글입니다. 



### Scope Function 이란? 


일시적인 영역을 형성하는 함수를 말한다. 



{% raw %}
```kotlin
fun printPerson(person: Perosn?) {
	if (person != null) {
		println(person.name)
		println(person.age)	
	}
}

// Refactoring
fun printPerson(person: Person?) {
	person?.let {
		println(it.name)
		println(it.age)
	}
}
```
{% endraw %}


1. Safe Call (?.) 을 사용했다. person 이 null 이 아닐 때에 let 을 호출한다.
2. let 은 scope function 의 한 종류이다. let 은 확장함수로, 람다를 받아서, 람다 결과를 반환한다.

> 💡 결론적으로, 람다를 사용해 일시적인 영역을 만들어서 코드를 더 간결하게 만들거나, method chaning 에 사용하는 것을 말한다. 



### Scope Function 의 분류


let, run, also, apply, with 가 있다. with 는 확장함수는 아니다. 

1. let, run 은 람다의 결과를 반환한다.
2. also, apply 는 객체 그 자체를 반환한다.
3. let, also, with 는 scope 내에서 `it` 을 사용한다.
4. run, apply 는 `this` 를 사용한다.


{% raw %}
```kotlin
val value1 = person.let {
	it.age
}

val value2 = person.run {
	this.age
}

val value3 = person.also {
	it.age
}

val value4 = person.apply {
	this.age
}
```
{% endraw %}



this 는 생략이 가능한 대신, 다른 이름을 붙일 수 없고, it 은 생략이 불가능한 대신, 다른 이름을 붙일 수 있다. 이런 차이는 사실, 코틀린의 문법 차이 때문에 발생한다. 


with 는 다음과 같다. 



{% raw %}
```kotlin
val person = Person("Jack", 100)
with(person) {
	println(name)
	println(age)
}
```
{% endraw %}



정리하자면 다음과 같다. 


|         | `it` 사용 | `this` 사용 |
| ------- | ------- | --------- |
| 람다의 결과  | let     | run       |
| 객체 그 자체 | also    | apply     |
|         | with    |           |

undefined

### 언제 어떤 Scope function 을 사용해야 하는가? 



#### Let


하나 이상의 함수를 call chain 결과로 호출할 때 사용한다. 



{% raw %}
```kotlin
val strings = listOf("APPLE", "CAR")
strings.map { it.length }
	.filter { it > 3 }
	.let(::println)
	
// 다음 코드랑 동일하다.
strings.map { it.length }
	.filter { it > 3 }
	.let { lengths -> println(lengths) }
```
{% endraw %}



non-null 값에 대해서만 code block 을 실행시킬 때 사용한다. 



{% raw %}
```kotlin
val length = str?.let {
	println(it.uppercase())
	it.length
}
```
{% endraw %}



일회성으로 제한된 영역에 지역 변수를 만들 때 사용한다.



{% raw %}
```kotlin
val numbers = listOf("one", "two", "three", "four")
val modifiedFirstItem = numbers.first()
	.let { firstItem ->
		if ( firstItem.length >= 5) firstItem else "!$firstItem!"
	}.uppercase()
	
println(modifiedFirstItem)
```
{% endraw %}




#### Run


객체 초기화와 반환 값을 동시에 해야할 때. 예를 들어서 객체를 만들어서 DB 에 바로 저장하고, 그 인스턴스를 활용할 때 사용한다. 



{% raw %}
```kotlin
val person = Person("Jack", 100).run(personRepository::java)
```
{% endraw %}




#### apply


객체 그 자체가 반환되는 특징이 있다. 객체 설정을 할 떄, 객체를 수정하는 로직이 call chain 의 중간에 필요한 경우에 사용한다. 



{% raw %}
```kotlin
fun createPerson(
	name: String,
	age: Int,
	hobby: String,
) : Person {
	return Person(
		name = name,
		age = age,
	).apply {
		this.hobby = hobby
	}
}

// 이론상 가능은 하다. 
val person = Perosn("Jack", 100)
person.apply {this.growold()}.let { println(it) }
```
{% endraw %}




#### Also


객체 그 자체가 반환되어, 객체를 수정하는 로직이 call chain 중간에 필요할 때 사용한다. 



{% raw %}
```kotlin
mutableListOf("one", "two", "three")
	.also { println("four 추가 이전 지금 값: $it") }
	.add("four")
	
// 이거랑 같다. 
val numbers = mutableListOf("one", "two", "three")
println("The list elements before adding new one: $numbers")
numbers.add("four")
```
{% endraw %}




#### with


특정 객체를 다른 객체로 변환해야 하는데, 모듈 간의 의존성에 의해서 정적 팩토리 혹은 toClass 함수를 만들기가 어려울 때 사용한다. 



{% raw %}
```kotlin
return with(person) {
	PersonDto(
		name = name, 
		age = age,
	)
}
```
{% endraw %}




### Scope Function 의 가독성


가독성이 항상 좋을까? 숙련된 코틀린 개발자도 잘 이해하지 못하는 경우가 있을 수 있으므로, 전통적인 구현 방법이 더 가독성이 좋을 수 있다. 



{% raw %}
```kotlin
if (person != null && person.isAdult) {
	view.showPerson(person)
} else {
	view.showError()
}

// 함수형
person?.takeIf { it.isAdult }
	?.let(view::showPerson)
	?: view.showError()
```
{% endraw %}



사용 빈도가 적은 관용구는 코드를 더 복잡하게 만들고, 이런 관용구들은 한 문장 내에서 조합해 사용하면 복잡성이 증가하게 된다. 


적절한 convention 을 적용하면 유일하게 활용할 수 있다. 

