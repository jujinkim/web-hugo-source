---
title: "안드로이드에서 Dagger2 사용하기 - 기본"
date: 2020-10-22T22:32:05+09:00
draft: true
---

&nbsp;지난 글에서는 DI란 무엇인가에 대해 라면 요리사를 예로 들면서 살펴보았습니다. 하지만 라면 만드는 요리사 하나를 구현하는데 너무 많은 코드를 작성했다고 생각되었었습니다. DI가 뭔가 발전된 코딩 패러다임인 것도 알고있고, 유지보수에 장기적으로 도움이 되는 것도 알겠는데, 그렇게 하기 위해 손이 너무 많이 가는 것 처럼 느껴집니다. 그래서 많은 개발자들이 DI를 효율적으로 적용하기 위해 다양한 프레임워크나 라이브러리를 개발해냈습니다. 그 중 가장 유명한 안드로이드용 DI 프레임워크, `Dagger2`를 살펴보겠습니다.

---

## Dagger2란?
&nbsp;Dagger2는 Google이 개발 및 유지보수중인 자바 & 안드로이드용 DI 프레임워크입니다 (왜 이름이 Dagger(단검)인지는 모르겠습니다.). 이전에 Square라는 곳에서(okhttp, retrofit등을 개발한 페이먼트 기업) Dagger1를 개발했었으나 이런저런 구조적 문제로 deprecate시키고, Google과 함께 만든 다음 버전이 Dagger2입니다.

&nbsp;Dagger2는 기본적으로 코드에 어노테이션을 붙여 사용합니다.(어노테이션은 자바나 코틀린에서 클래스&변수&함수 선언 앞에 붙이는, `@`로 시작하는 지시어를 말합니다.) 자세한건 아래 예제에서 확인하실 수 있습니다.  
&nbsp;앱을 만들면서 코드를 작성한 후, 의존성 주입이 필요하거나 객체를 제공, 유통할 클래스들에게 알맞은 어노테이션을 달아주는 식으로 사용합니다. 그럼 프로젝트를 빌드할 때에, 어노테이션을 읽고 Dagger2가 내부적으로 생성/주입 코드를 추가로 생성합니다. 실질적인 DI 코드는 Dagger2 프레임워크가 알아서 작성하고, 앱 개발자는 그저 DI가 어떻게 진행되야하는지 규칙정도만 정해주면 되는 것입니다.

&nbsp;Dagger2는 코틀린에서도 사용이 가능합니다. 다만, 코들린에는 `Koin`이라는 다른 DI 프레임워크도 있고 많은 사람들이 Koin이 더 사용하기 쉽다고 하니, 코틀린으로 개발하시는 분들은 Koin도 같이 공부해보세요.

---
## 이전의 요리사 코드에 Dagger2를 적용하기
&nbsp;DI에는 통상 오브젝트를 생산(공급)하고, 제공(유통)하고, 사용하는 클래스로 나뉩니다. 설계에 따라 다릅니다만, 아무튼 의존성을 주입하려면 주입하는 클래스와 관리해주는 클래스가 있어야하죠. Dagger2도 비슷한 구조로 시작할 수 있습니다. 지난 글에서 만든 코드를 Dagger2에 맞게 수정해보겠습니다.

&nbsp;이전의 DI 글에서 다루었던 요리사 코드를 Dagger2 프로젝트로 바꿔봅시다.

&nbsp;우선, 아래와 같이 코드를 준비하겠습니다. 지난 글에서 만들어놨던, DI를 구현한 라면 요리사 코드입니다.  
&nbsp;왜 라면 하나 만드는 요리사를 이렇게 복잡하게 구현했는지 궁금하시면 지난 글을 확인하시기 바랍니다.

재료들
```kotlin
// Noodle.kt
class Noodle

// Sauce.kt
class Sauce

// Water.kt
class Water

// RamenPot.kt
class RamenPot {
    fun makeRamen(water: Water, sauce: Sauce, noodle: Noodle): Ramen {
        return Ramen(water, sauce, noodle)
    }
}

// 결과물(라면) Ramen.kt
class Ramen(water: Water, sauce: Sauce, noodle: Noodle) {
    fun eat() = println("yum")
}
```

재료를 만들어 공급하는 클래스들
```kotlin
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
재료를 유통하는 클래스
```kotlin
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
재료를 쓰는 클래스
```kotlin
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
&nbsp;코드를 수정하기 앞서, Dagger2도 외부 프레임워크이기 때문에 안드로이드 프로젝트에 적용하려면 의존성을 추가해줘야합니다. 코틀린 기준으로 아래와 같이 추가합니다.

build.gradle
```java
plugins {
    ...
    id 'kotlin-kapt'
}

dependencies {
    ...

    implementation 'com.google.dagger:dagger:2.15'
    implementation 'com.google.dagger:dagger-android:2.15'
    implementation 'com.google.dagger:dagger-android-support:2.15'
    kapt 'com.google.dagger:dagger-compiler:2.15'
    kapt 'com.google.dagger:dagger-android-processor:2.15'
}
```
&nbsp;이 글 작성 당시 최신 버전은 `2.15` 입니다. 최신 버전을 적용해주면 좋고, 버전명을 따로 변수로 빼서 지정해주는게 보편적입니다. (`.+`를 붙이는 방법은 권장하지는 않습니다. Lint에도 뜰텐데, 최신버전 가져오느라 싱크&빌드 시간을 잡아먹어요.)

---
### 공급자(모듈) 만들기
&nbsp;공급자(Maker)를 먼저 만들겠습니다. Dagger2에서는 이런 공급자들을 `모듈(Module)`이라고 부릅니다. 모듈을 만들기 위해선 클래스를 하나 만들고 `@Module` 어노테이션을 붙여줍니다. 그리고 Dagger2에 맞게 그 클래스를 구현하면 됩니다. 우리는 이미 각 Maker 클래스들을 만들어 놓은 상태이므로, 이 클래스들을 모듈로 바꿔보겠습니다.

1. `IMaker` 인터페이스는 필요하지 않습니다. 이 인터페이스는 Dagger2 안에 다른 개념으로 구현되어있습니다. 그래서 IMaker는 지울 것입니다.
2. 각 Maker 클래스들이 IMaker를 상속받는 대신, `@Module` 어노테이션을 붙여서 이 클래스들이 모듈(공급자) 임을 표시할 것입니다.
3. 보기 편하게 Maker 클래스들의 이름을 Module로 바꿔주겠습니다.
4. 클래스 내용을 Dagger2 스타일로도 맞추어주겠습니다.

&nbsp;3번은 필수는 아니지만, 협업, 유지보수 등을 위해서 이해하기 편하게 바꿔주는게 좋습니다.

```kotlin
// IMaker.kt -> 제거합니다. 
// 대신 Dagger2의 @Module을 사용할 것입니다.

// 나머지는 아래와 같이 변경합니다.

// WaterMaker.kt -> WaterModule.kt (파일 & 클래스 이름 변경)
@Module
class WaterModule {
    @Provides
    fun getItem(): Water = Water()
}

// SauceMaker.kt -> SauceModule.kt
@Module
class SauceModule {
    @Provides
    fun getItem(): Sauce = Sauce()
}

// NoodleMaker.kt -> NoodleModule.kt
@Module
class NoodleModule {
    @Provides
    fun getItem(): Noodle = Noodle()
}

// PotMaker.kt -> PotModule.kt
@Module
class PotModule {
    @Provides
    fun getItem(): RamenPot = RamenPot()
}
```

&nbsp;`@Module`이라는 어노테이션으로 클래스가 `Module`(공급자)이라는 것을 표시합니다. 인터페이스를 쓸 땐 IMarket 를 붙였듯이, Dagger2에서는 어노테이션을 붙이면 됩니다.

&nbsp;공급자를 만들었으면 생산하는 방법도 만들어야겠지요. `@Provides`는 인스턴스를 만드는 메소드에 붙여줍니다. 이 때, 메소드 이름은 중요하지 않습니다. 메소드의 반환 타입과 매개변수, 내용이 중요합니다. 위 코드에서 `PotModule`에는 `RamenPot`을 반환하는 메소드 `getItem()`이 있습니다. 이 메소드에 @Provides를 붙여주면, Dagger2 컴포넌트는 자동으로 RamenPot을 공급받아야 할 때 이 메소드를 호출할 것입니다.

---
### 유통자(컴포넌트) 만들기
&nbsp;유통하는 클래스들도 Dagger2 에 맞게 변경해주겠습니다. Dagger2에서는 마켓처럼 유통하는 요소들을 `Component`라고 부릅니다. Component는 사용자로부터 요청이 오면 필요한 인스턴스들을 Module로부터 공급받아서 사용자에게 넘겨주는(주입해주는) 역할을 하는 것입니다.  
&nbsp;마찬가지로 우리는 이미 Market 클래스를 만들어 놓았으므로 얘를 Component로 바꾸겠습니다.

1. `IMarket`은 불필요합니다. Maker와 마찬가지로, IMarket 대신 `@Component`를 사용할 것입니다.
2. Component가 사용할 모듈들을 Component에 연결해줍니다.
3. Component에 의존성 주입(객체 전달) 메소드를 만들어줍니다. 사용자 클래스들은 컴포넌트의 주입 메소드를 호출해서 객체를 전달(의존성을 주입)받을 것입니다.

```kotlin
// IMarket.kt -> 제거합니다.
// 대신 Dagger2의 @Component를 사용할 것입니다.

// Market.kt -> MarketComponent.kt 이름 변경, 내용 변경
@Component(
        modules = [
            NoodleModule::class,
            PotModule::class,
            SauceModule::class,
            WaterModule::class
        ])
interface MarketComponent {
    fun inject(chef: Chef)
}
```
&nbsp;`Component`는 interface 또는 abstract class에 만듭니다. 그래서 위 코드에서도 `interface MarketComponent`로 만들었습니다. 이건 그냥 Dagger2가 그렇게 만들어졌기 때문에 외우면 됩니다.

&nbsp;`@Component` 어노테이션을 붙여서 이 인터페이스가 컴포넌트(유통자)임을 표시합니다. 그리고 어노테이션 매개변수로 이 컴포넌트가 사용할 모듈들을 적어서 연결해줍니다. 이 컴포넌트가 사용자 클래스로부터 주입 요청을 받으면, 필요한 객체들을 여기에 적어준 모듈들로부터 찾을 것입니다.

&nbsp;인터페이스 안에 `inject()`라는 메소드를 만들어줬습니다. 이 메소드는 매개변수로 전달 된 객체에다가 의존성을 주입해줍니다. 즉, 매개변수로 들어온 객체가 필요로하는 다른 객체들을 모듈로부터 찾아서 넘겨주는 메소드입니다. 이 메소드는 컴포넌트에 정의만 해놓으면, 실제 구현은 Dagger2가 알아서 할 겁니다.

---
### 사용자 만들기
&nbsp;마지막으로 사용자 클래스들을 만들어주겠습니다. 사실 사용자 클래스들은 만든다기보단 Dagger2로부터 의존성을 주입받을 수 있도록 수정해준다고 보시면 됩니다. 사용자 클래스들은 실제 프로젝트에서 사용하는 프로젝트 코드이기 때문에, 프로젝트 설계에 따라 자유롭게 평소처럼 만드시면 됩니다. 대신, 클래스 안에서 사용하는 객체들은 외부에서 주입받을 것이므로 이 객체 필드들 정의만 조금 수정해주면 됩니다.

현재 예제 코드에서는 `Chef`가 사용자 클래스가 되겠습니다.

```kotlin
// Chef.kt -> 아래와 같이 Dagger2로부터 주입받도록 수정합니다.
class Chef {
    @Inject lateinit var water: Water
    @Inject lateinit var sauce: Sauce
    @Inject lateinit var noodle: Noodle
    @Inject lateinit var pot: RamenPot

    init {
        DaggerMarketComponent.create().inject(this)
    }

    fun cookRamen(): Ramen {
        val ramen = pot.makeRamen(water, sauce, noodle)
        return ramen
    }
}
```
&nbsp;`@Inject`는 주입받을 필드를 명시할 때 사용합니다. 이 어노테이션이 붙은 필드들은 Component로부터 자동으로 주입받아서 할당되게 됩니다.

&nbsp;`init`에서 보면, Component를 만들고 `inject`를 호출하면서 매개변수로 `this`를 넘겨주고 있습니다. 이 메소드를 호출하게 되면 사용자 클래스가 사용할 멤버 객체들을 주입받게 됩니다. 이 것으로 기본적인 Dagger2 DI 구현이 완성됩니다.

&nbsp;여기서 `DaggerMarketComponent`는 만든적이 없는데 어디서 나왔나 궁금하실텐데요, 얘가 바로 Dagger2가 알아서 만든 클래스입니다. 위에서 MarketComponent 인터페이스만 만들고 실제 구현은 하지 않았다는걸 기억하실겁니다. 이 인터페이스를 구현한 클래스가 DaggerMarketComponent 입니다. 실제 구현 된(자동으로 생성된) 컴포넌트들은 이름 앞에 "Dagger"가 붙습니다. 우리는 Dagger2에게 컴포넌트 설계도만 만들어주면, 실제 구현은 Dagger2가 한다고 생각하시면 됩니다.  
&nbsp;DaggerMarketComponent와 같은 컴포넌트들은 싱글톤이 아닙니다. 그래서 create()를 통해 만들어줘야합니다. 이 create()는 static method로, 각종 모듈들이 함께 초기화된 컴포넌트를 만들어서 반환합니다.  
&nbsp;만약 찾을 수 없는 클래스로 나온다면, 프로젝트 전체 빌드를 한번 해주세요. 빌드되면서 Dagger2가 어노테이션들을 읽고 자동으로 이런 클래스들을 생성합니다. 

---
&nbsp;MainActivity에서 Chef를 만들고 바로 cookRamen을 호출하면 라면이 만들어집니다. Chef클래스 안에서 라면 재료들을 만들지 않았지만, `Component.inject`를 통해 재료(의존성)를 공급(주입)받았기 때문에 바로 `cookRamen()`이 가능한 것입니다. 설계에 따라서는 MainActivity의 Chef 인스턴스도 Dagger2를 통해 패턴으로 제공받을 수 있겠지요

---
## Dagger2 Graph
&nbsp;전체적인 DI 흐름이 보이셨나요? 요약하자면 다음과 같습니다.
> 사용자 클래스가 컴포넌트의 `inject(this)`를 호출함으로써, 컴포넌트에 본인 객체를 넘겨줍니다. 그럼 컴포넌트는 넘어온 객체에서 `@Inject`가 달린 필드를 읽고, 이 필드의 타입과 맞는 객체(의존성)를 연결된 모듈로부터 찾아서 할당(주입)합니다. 이것이 전체적인 Dagger2의 의존성 주입 과정입니다.

&nbsp;이런 일련의 흐름들을 `Dagger2 Graph`라고 합니다. Dagger2를 쓸 때에는 모듈들, 컴포넌트들, 그리고 실제 사용할 클래스들의 관계와 흐름(그래프)을 잘 설계해야합니다. 그리고 그 설계를 그림을 그려보는 것도 좋습니다. 이번 예제 코드에서의 그래프를 그림으로 나타내면 아래처럼 그릴 수 있겠네요.

```
그림~~~~
```

---
&nbsp;이번 글에서는 Dagger2의 기본적인 사용 방법을 알아보았습니다. 하지만 실제 프로젝트에서는 이보다 다양한 방법과 복잡한 조건으로 객체를 만들고 관리해야합니다. 예를 들면 싱글톤 인스턴스라던지, 아니면 클래스 생성자에 매개변수가 있다던지요. 물론 Dagger2는 이런 복잡한 상황들을 모두 포함할 수 있습니다. 실제로 어노테이션의 종류도 훨씬 많구요. 더 다양하고 세밀하게 DI를 설계하고 그래프를 그릴 수 있습니다.  

&nbsp;이번 글에서는 '아 Dagger2는 이런식으로 쓰는거구나~` 정도만 알아두고, 다음 글에서 조금 더 다양한 Dagger2의 사용법을 알아보겠습니다.