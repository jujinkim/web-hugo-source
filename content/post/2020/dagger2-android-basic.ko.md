---
title: "안드로이드에서 Dagger2 사용하기 - 기본"
date: 2020-10-22T22:32:05+09:00
draft: true
---

&nbsp;지난 글에서는 DI란 무엇인가에 대해 라면 요리사를 예로 들면서 살펴보았습니다. 하지만 라면 만드는 요리사 하나를 구현하는데 너무 많은 코드를 작성했다고 생각되었었습니다. DI가 뭔가 발전된 코딩 패러다임인 것도 알고있고, 유지보수에 장기적으로 도움이 되는 것도 알겠는데, 그렇게 하기 위해 손이 너무 많이 가는 것 처럼 느껴집니다. 그래서 많은 개발자들이 DI를 효율적으로 적용하기 위해 다양한 프레임워크나 라이브러리를 개발해냈습니다. 그 중 가장 유명한 안드로이드용 DI 프레임워크, `Dagger2`를 살펴보겠습니다.

---

## Dagger2란?
&nbsp;Dagger2는 Google이 개발 및 유지보수중인 자바 & 안드로이드용 DI 프레임워크입니다 (왜 이름이 Dagger(단검)인지는 모르겠습니다.). 이전에 Square라는 곳에서(okhttp, retrofit등을 개발한 페이먼트 기업) Dagger1를 개발했었으나 이런저런 구조적 문제로 deprecate시키고, Google과 함께 만든 다음 버전이 Dagger2입니다.

&nbsp;Dagger2는 기본적으로 코드에 어노테이션을 붙여 사용합니다. 어노테이션은 자바나 코틀린에서 클래스&변수&함수 선언 앞에 붙이는, `@`로 시작하는 지시어를 말합니다. 자세한건 아래 예제에서 확인하실 수 있습니다.  
&nbsp;앱을 만들면서 코드를 작성한 후, 의존성 주입이 필요하거나 객체를 제공, 유통할 클래스들에게 알맞은 어노테이션을 달아주는 식으로 사용합니다. 그럼 프로젝트를 빌드할 때에, 어노테이션을 읽고 Dagger2가 내부적으로 생성/주입 코드를 추가로 생성합니다. 실질적인 DI 코드는 Dagger2 프레임워크가 알아서 작성하고, 앱 개발자는 그저 DI가 어떻게 진행되야하는지 규칙정도만 정해주면 되는 것입니다.

&nbsp;Dagger2는 코틀린에서도 사용이 가능합니다. 다만, 코들린에는 `Koin`이라는 다른 DI 프레임워크도 있고 많은 사람들이 Koin이 더 사용하기 쉽다고 하니, 코틀린으로 개발하시는 분들은 Koin도 같이 공부해보세요.

---
## 이전의 요리사 코드에 Dagger2를 적용하기
이전의 DI 글에서 다루었던 요리사 코드를 Dagger2 프로젝트로 바꿔보겠습니다.

우선, 아래와 같이 코드를 나누겠습니다.

```kotlin
/// 
```

```kotlin
// 재료를 만드는 클래스들

// IMaker.kt
interface IMaker<T> {
    fun getItem(): T
}

// WaterMaker.kt
class WaterMaker(): IMaker<Water> {
    override fun getItem(): Water = Water()
}

// SauceMaker.kt
class SauceMaker(): IMaker<Sauce> {
    override fun getItem(): Sauce = Sauce()
}

// NoodleMaker.kt
class NoodleMaker(): IMaker<Noodle> {
    override fun getItem(): Noodle = Noodle()
}

// PotMaker.kt
class PotMaker(): IMaker<RamenPot> {
    override fun getItem(): RamenPot = RamenPot()
}
```
```kotlin
// 재료를 유통하는 클래스

// IMarket.kt
interface IMarket {
    fun passIngredients(visitor: Any)
}

// Market.kt
class Market: IMarket {
    val waterMaker = WaterMaker()
    val sauceMaker = SauceMaker()
    val noodleMaker = NoodleMaker()
    val potMaker = PotMaker()

    override fun passIngredients(visitor: Any) {
        if (visitor is Chef) {
            visitor.sauce = sauceMaker.getItem()
            visitor.noodle = noodleMaker.getItem()
            visitor.water = waterMaker.getItem()
            visitor.pot = potMaker.getItem()
        }
    }
}
```
```kotlin
// 재료를 쓰는 클래스

// Chef.kt
class Chef(val market: IMarket) {
    lateinit var water: Water
    lateinit var sauce: Sauce
    lateinit var noodle: Noodle
    lateinit var pot: RamenPot

    init {
        market.passIngredients(this)
    }

    fun cookRamen(): Ramen {
        val ramen = pot.makeRamen(water, sauce, noodle)
        return ramen
    }
}
```
---
### Dagger2를 Android 프로젝트에 적용하기
&nbsp;Dagger2도 외부 프레임워크이기 때문에 안드로이드 프로젝트에 적용하려면 의존성을 추가해줘야합니다.

build.gradle
```gradle
dependencies {
    ...

    implementation 'com.google.dagger:dagger:2.15'
    implementation 'com.google.dagger:dagger-android:2.15'
    implementation 'com.google.dagger:dagger-android-support:2.15'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.15'
    annotationProcessor 'com.google.dagger:dagger-android-processor:2.15'
}
```
이 글 작성 당시 최신 버전은 `2.15` 입니다.


### 공급자(모듈) 만들기

### 유통자(컴포넌트) 만들기

### 사용자 만들기

---
