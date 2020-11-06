---
title: "DI(의존성 주입)란 무엇일까?"
date: 2020-10-18T01:12:39+09:00
draft: false
---
&nbsp;이 글은 Dagger2를 공부하기에 앞서 DI(의존성 주입)라는게 무엇인지 도저히 모르겠는 분들을 위한(나를 위한) 글입니다.

## DI란 무엇인가?

&nbsp;DI는 의존성 주입(Dependency Injection)의 약자로, **어떤 객체의 의존성을 외부에서 주입시켜주는 프로그래밍 설계 기법**입니다. 무슨 말인지 모르겠다구요? 네, 저도 모르겠습니다. 뭘 주입한다는건지도 모르겠고, 알았다 하더라도 이걸 왜 써야하는지 모르겠단 말입니다. 영어로 보아도, 한국어로 보아도 모르겠습니다. 그래서 이 글을 썼습니다. 

&nbsp;DI 특징을 정의하기에 앞서, 일상에서의 어떤 상황을 상상해보겠습니다.

> 당신은 지금 배가 고파서 분식집에 들어갔고, 라면을 주문했습니다.

&nbsp;요리사는 라면 조리를 시작할 것입니다. 재료(준비물)로는 물, 면, 소스, 그리고 냄비가 필요합니다. 조리 과정은 "냄비에 물, 소스, 면을 넣은 후 끓인다" 가 될 것입니다.   
&nbsp;여기서 요리사는 재료들을 어디서 구해왔을까요? 당연히 외부에서 사왔을 거에요. 직접 물, 소스, 면, 냄비를 만들지 않고 밖에서 구해왔을 것입니다. 이렇게 외부에서 공급받는 것을 DI(의존성 주입)이라고 합니다.

> 요리사는 재료에 **의존**해서 라면을 만듭니다. 없으면 만들 수 없으니까요. 그리고 요리사는 이것들을 외부에서 구해옵니다. 다른말도 하면, 외부로부터 재료들을 **주입**받는다고 할 수도 있겠습니다. 그래서 **의존성 주입**이라고 하는 것입니다.

&nbsp;만약 재료(준비물)들을 사오지 않는다면 요리사가 라면을 조리하는 과정은 조금 복잡해집니다. "물을 만들고, 면을 만들고, 소스를 만들고, 냄비를 만든다. 그리고 냄비에 재료들을 넣고 끓인다" 가 조리 과정이 되겠지요.  
&nbsp;요리사가 모든걸 다 하니까 좋아보이겠지만, 생각해보면 그다지 좋은 것은 아닙니다. 요리사는 말 그대로 요리하는 사람이지, 재료를 만드는 사람은 아니니까요. 코딩도 비슷합니다. 코딩에서는 '역할의 분리'가 중요한데, 무슨 말이나면 하나의 메서드는 한 가지 일만, 클래스는 명확한 목적만을 수행하도록 만들어지는 게 좋다는 뜻입니다. 이것은 수정사항이 생겼을 때 코드의 불필요한 수정을 최소화하고, 재사용 및 테스트를 용이하게 만들기 위함입니다. 실제로 우리가 일할 때에도 위에서 이것저것 다 시키는 것 보다는 한 가지 일에만 집중할 때 일의 효율이 올라가듯요. 시키는 사람에게는 대충 한 사람이 알아서 다 해주면 처음에는 좋겠지만, 규모가 커진다면 일이 생겼을 때에 누구를 시켜야 하는지 애매해지죠. 이런저런 이유로 역할의 분리가 중요합니다.

&nbsp;이제 요리사가 라면을 조리하는 과정을 코드로 만들어보겠습니다.

---

## DI를 사용하지 않은 요리사

먼저, 재료(준비물)들을 정의하겠습니다.
```kotlin
// 재료들
class Noodle
class Sauce
class Water
class RamenPot {
    fun makeRamen(water: Water, sauce: Sauce, noodle: Noodle): Ramen {
        return Ramen(water, sauce, noodle)
    }
}

// 라면 (결과물)
class Ramen(water: Water, sauce: Sauce, noodle: Noodle)
```

&nbsp;일반적인 방법(DI 안쓴방법)으로 코드를 짠다면 다음의 코드가 라면을 조리하는 요리사의 클래스 정의입니다.
```kotlin
class Chef {
    fun cookRamen(): Ramen {
        val noodle = Noodle()
        val sauce = Sauce()
        val water = Water()
        val pot = RamenPot()

        val ramen = pot.makeRamen(water, sauce, noodle)
        return ramen
    }
}
```
&nbsp; `Chef`는 `cookRamen()` 메소드로 라면을 조리하는 방법을 알고 있습니다. 그런데 이 코드대로다면 **라면을 조리하는 과정에 면, 소스, 물과 냄비를 만드는 과정이 포함**됩니다, 이상하죠. 밥을 시켰는데 쌀부터 만들어오라고 주문을 하는 것이지요. 그리고, 이 라면 만드는 과정(`cookRamen()`)을 시험하고싶다면 어떻게 되는걸까요? 이 코드대로라면, 라면을 만드는 과정 뿐만 아니라 재료를 생산하는 과정까지 테스트를 하는 꼴이 됩니다. 이상하죠, 마스터쉐프코리아에서 요리사들 평가할 때 재료를 생산하는 과정까지 평가하지는 않잖아요?

&nbsp;코딩의 관점에서 위 방법대로 구현한다면 나중에 귀찮아 질 수 있는 문제도 있습니다. 만약 라면 말고 물이 필요한 다른 요리를 하는 요리사(`Chef`)도 있다고 할 때, 물(`Water`)의 생성자에 매개변수가 추가된다면 `모든 물을 사용하는 Chef 클래스들의 물 생성 코드를 수정`해줘야합니다. 또, 뭘 만들 때마다 재료를 생산하기 때문에 객체 생성 코드(`val water = Water()` 등)를 여러곳에서 쓰는 `중복 코드`도 발생할 수 있습니다.

&nbsp;우리는 이 문제들을 4단계로 나눠 점차 DI를 적용하여 해결해볼 것입니다.

---
## DI 1단계: 재료를 외부에게서 공급받도록 한다.
&nbsp;글 처음에 써있듯이, 요리사는 라면을 만드는 재료를 만들지 않습니다. 그럼 어디서 재료를 구할까요? 우선, 사장님이 재료를 주고 요리하라고 시켰을 수 있습니다. 그럼 아래와 같이 요리사 클래스 코드를 바꿀 수 있습니다.

```kotlin
class Chef {
    fun cookRamen(water: Water, 
                  sauce: Sauce, 
                  noodle: Noodle, 
                  pot: RamenPot): Ramen {
        val ramen = pot.makeRamen(water, sauce, noodle)
        return ramen
    }
}
```
&nbsp;이제 요리사는 라면 조리를 할 때 직접 재료를 만들지 않고, 누군가가 준 재료들을 가지고 조리합니다. 조리 테스트를 할 때에도 순수하게 라면을 만드는 과정만 시험할 수 있겠네요.  
&nbsp;하지만 여전히 문제가 있습니다. 조리 하라고 시키는 누군가가 재료를 구해줘야한다는 것입니다. `cookRamen()`을 호출할 때 매개변수를 채워줘야해요. 실제로는 시키는 사람이 재료를 만들어주지는 않을텐데말이죠.

---
## DI 2단계: 재료를 공급하는 곳을 만든다.
&nbsp;그럼 재료는 어디에서 가져와야 하는걸까요? 아무래도 시장에서 사오지 않을까요? 그래서 라면 재료를 파는 시장(`Market`)을 만들어주겠습니다.

```kotlin
class Market {
    fun getNoodle(): Noodle = Noodle()
    fun getSauce(): Sauce = Sauce()
    fun getWater(): Water = Water()
    fun getRamenPot(): RamenPot = RamenPot()
}
```
&nbsp;이제 시장이 생겼으니 요리사는 라면 주문을 받으면 재료를 구해와서 조리하면 됩니다.
```kotlin
class Chef(val market: Market) {
    fun cookRamen(): Ramen {
        val water = market.getWater()
        val sauce = market.getSuace()
        val noodle = market.getNoodle()
        val pot = markget.getRamenPot()

        val ramen = Ramen(water, sauce, noodle, pot)
        return ramen
    }
}
```
&nbsp;요리사가 재료를 구해오든지, 아니면 `cookRamen()`에 매개변수를 추가해서 재료를 받든지 아무튼 요리사나 사장님은 더 이상 재료를 직접 만들 필요는 없어졌습니다. 시장(Market)에서 재료를 받으면 되니까요.

&nbsp;그래도 조금 어색합니다. **보통 시장은 자기가 파는 물건을 직접 만들지는 않잖아요? 납품받아서 팔지.** 시장은 유통해줄 뿐, 제품을 생산하는 곳은 따로 있습니다. 그리고 자주 시장에서 물건을 사다 보면 다음부터는 일일이 항목별로 사지 않고, 아예 묶어서 구입하기도 합니다.

---
## DI 3단계: 재료를 만드는곳, 유통하는 곳, 사용하는 곳으로 나눈다.
&nbsp;시장은 물건을 유통하는 곳입니다. 만드는 곳은 따로 있습니다. 그러므로 시장도 다른 곳에서 물건을 제공받도록 수정해줘야합니다. 만드는 곳, 파는 곳, 쓰는 곳으로 역할을 나눠서 코드를 다시 작성해보겠습니다. 

```kotlin
// 각 재료를 만드는(공급하는) 클래스
class WaterMaker() {
    fun getWater(): Water = Water()
}

class SauceMaker() {
    fun getSauce(): Sauce = Sauce()
}

class NoodleMaker() {
    fun getNoodle(): Noodle = Noodle()
}

class PotMaker() {
    fun getRamenPot(): RamenPot = RamenPot()
}
```
```kotlin
// 재료를 유통하는 클래스
class Market {
    val waterMaker = WaterMaker()
    val sauceMaker = SauceMaker()
    val noodleMaker = NoodleMaker()
    val potMaker = PotMaker()

    fun passRamenIngredients(visitorChef: Chef) {
        visitorChef.sauce = sauceMaker.getSauce()
        visitorChef.noodle = noodleMaker.getNoodle()
        visitorChef.water = waterMaker.getWater()
        visitorChef.pot = potMakger.getRamenPot()
    }
}
```
```kotlin
// 재료를 쓰는 클래스
class Chef(val market: Market) {
    lateinit var water: Water
    lateinit var sauce: Sauce
    lateinit var noodle: Noodle
    lateinit var pot: RamenPot

    init {
        market.passRamenIngredients(this)
    }

    fun cookRamen(): Ramen {
        val ramen = pot.makeRamen(water, sauce, noodle)
        return ramen
    }
}
```
&nbsp;이제야 산업혁명 후 발전된 현대 사회의 모습이 코드에서 보이네요.

- 각 재료의 생산자(공급자)들이 있다.
- 유통자 역할을 하는 시장이 있다. 시장은 재료를 생산자에게서 납품받아서 소비자에게 제공한다.
- 고객이 단골인가보다. 일일이 재료를 구할 필요 없이 "평소 사던거요" 같은 주문을 한다.
- 요리사는 이제 "라면을 만들라"는 지시에 정말 "라면을 조리하는" 한 가지 일만을 할 수 있다.

&nbsp;이제 남은 것은 소비자(요리사)가 시장을 "시장"으로 인식하게 하는 것입니다. 무슨말이냐면, 지금 구현된 `Market`은 시장이 아니고 그냥 "Market"이라는 이름을 가진 재료를 주는 녀석에 불과합니다. 하지만 "Market"은 엄연히 시장이므로, 인터페이스를 통해 시장만이 할 수 있는 동작을 지정해주고, 소비자는 Market이 아닌 "시장"에게 물건을 주문하도록 해보겠습니다.

---
## DI 4단계: 시장을 진짜 시장으로 만든다.
&nbsp;`Market`을 시장 인터페이스를 상속받게 하고, 요리사는 시장 인터페이스만 알도록 할 것입니다. 이것을 인터페이스를 통해 의존관계를 분리한다고 합니다. 어려운게 또 나왔습니다. 관계를 분리시킨다니. 사실 3단계까지는 쉬운 이해를 위해 최대한 직관적으로 예시 코드를 작성했습니다. 그러나 실제 유통망은 훨씬 복잡하다는걸 다들 아실겁니다. 그래서 제공자와 피제공자간 관계를 분리시켜야 합니다.  
&nbsp;분리시킨다는건 말 그대로 두 클래스가 서로를 모르게 한다는 것입니다. 코드상에서 두 클래스가 서로에 대해 참조를 안한다는 뜻이기도 하지요. 이게 왜 나왔냐하면, 사실 3번까지의 과정은 의존성을 주입시키기는 하지만, 주입되는 클래스가 주입하는 클래스를 너무 잘 알고있습니다. 소비자가 시장이 제공하는 모든 것을 알지는 않잖아요. **소비자는 시장이 제공하는 서비스**만 알고있을 뿐입니다.

&nbsp;프로젝트가 커지고 다양한 클래스들이 필요해질 경우 다형성도 필요하게됩니다. DI의 구현도 예외는 아닙니다. 객체를 제공하는 클래스도 세 개 보다는 많을 것이고, 객체를 유통하는 클래스가 `Market` 하나만 있지는 않을것입니다. 하지만 이들이 하는 행동은 비슷합니다. 제공 클래스들은 "제공"하고, 유통 클래스들은 "유통"하고, 나머지는 "사용"합니다. 이런 공통적인 행동들을 `Interface`로 묶어야합니다. 예를들어, 유통 역할을 할 클래스가 "식재료 마켓"과 "전자상가" 두 개가 있다고 하면, 두 시장은 분명히 취급하는 물건도, 방법도 다를 것입니다. 하지만 `물건을 납품받고, 소비자에게 제공(주입)한다` 는 역할(행동)은 동일하지요. 또, 제공 클래스들은 `물건을 제공한다`가 공통 행동입니다. 이런 행동들을 인터페이스로 묶는다면, 요리사는 Market 클래스가 정확히 어떻게 구현됐는지 알 필요가 없습니다. 그냥 추상화된 시장 인터페이스만 알고있고, 그 시장에 객체를 요청하기만 하면 필요한 객체를 주입받을 수 있을테니까요.

```kotlin
// 재료를 만드는 클래스들은 공통적으로 "만든다"라는 행동을 합니다.
interface IMaker<T> {
    fun getItem(): T
}

class WaterMaker(): IMaker<Water> {
    override fun getItem(): Water = Water()
}

class SauceMaker(): IMaker<Sauce> {
    override fun getItem(): Sauce = Sauce()
}

class NoodleMaker(): IMaker<Noodle> {
    override fun getItem(): Noodle = Noodle()
}

class PotMaker(): IMaker<RamenPot> {
    override fun getItem(): RamenPot = RamenPot()
}
```
```kotlin
// 재료를 유통하는 클래스들은 공통적으로 "유통한다"라는 행동을 합니다.
interface IMarket {
    fun passIngredients(visitor: Any)
}

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
// 재료를 쓰는 녀석
class Chef(val market: IMarket) {
    lateinit var water: Water
    lateinit var sauce: Sauce
    lateinit var noodle: Noodle
    lateinit var pot: RamenPot

    init {
        // Chef는 market이 Market클래스인지는 모릅니다.
        // 그저 IMarket(시장)이라는것만 알 뿐입니다
        market.passIngredients(this)
    }

    fun cookRamen(): Ramen {
        val ramen = pot.makeRamen(water, sauce, noodle)
        return ramen
    }
}
```

&nbsp;개쩔죠. 이것이 DI입니다. 느끼셨겠지만 DI는 어느 한 언어만을 위한게 아닌, 코드의 전반적인 설계입니다. 그런데 이거 코드도 길어졌고 그냥 처음처럼 짜면 안되나 싶기도 할 수도 있어요.  
&nbsp;만약 DI 안쓰고 이 프로젝트가 커졌다고 해봅시다. 요리사 종류도 많아지고 재료 종류도 많아지겠지요. 그럼 재료 하나 수정할 때마다 모든 요라사 다 수정해줘야합니다. 요리사들에게 일일이 재료 생성자 때려박고 관리하다보면 나중엔 귀찮아질거에요. 신기능 개발보다 빡센게 유지보수인건 다들 아실겁니다.

---
## IoC(제어의 역전)?
&nbsp;많은 DI 글들을 보면 꼭 빠지지 않는 개념으로 IoC(제어의 역전)이 있습니다. 글 마지막 부분에 이 개념을 쓴 이유는, 어차피 아무것도 모른 상태로 "제어를 역전한다" 이런 말 써봐야 저부터가 이해가 안되기 때문입니다. 

&nbsp;위 예제의 마지막에서는 이미 제어가 역전된 듯한 모습을 보여주긴 합니다. 요리사(소비자)가 직접 필요한 객체를 초기화하지 않았습니다. 그저 요리사는 추상화된 마켓에 "객체를 달라"고 호출만 했고, 마켓에서는 요청받은 객체를 알아서 만들어서 주입해줬죠. 즉, 요리사가 필요한 객체를 제어하는 입장에서, 제어된 객체를 받는 입장이 되어버린것이지요. 요리사는 객체가 어떻게 만들어진건지, 이 객체가 유일한지 싱글톤인지도 모릅니다. 그저 받아서 쓰기만 하면 될 뿐입니다.

&nbsp;사실 제어의 역전 개념은 이런 예제정도로는 완전히 설명하기 어렵습니다. 용어부터가 복잡하잖아요. 그리고 제어의 역전은 추상화가 들어가야합니다. 하위 클래스가 스스로 사용하려는 객체를 구체적으로 몰라도 되어야하기 때문입니다.

---
## 요약
- DI: 어떤 클래스에서, 자기가 쓸 객체들을 외부로부터 받는다. 그것들이 어떻게 만들어지는지 모르지만 일단 다른 클래스에 요청해서 받아다가 쓴다.
- IoC:  누군가 호출할 수도 있는 코드를 작성한다. 원래라면 호출자가 제어해야 할 코드를 다른 클래스가 제어한다. (어떤 클래스에서 사용될 객체들의 생성이나 생명주기 등을 그 클래스가 아닌 다른 클래스에서 제어)
- 왜 이렇게 만들어?: 이렇게 만든다면 산업혁명이 당신의 코드에서 일어났다고 생각하자.

&nbsp;산업혁명 발전 과정 중에는 "소품종 대량생산"이 있습니다. 이것은 각자 해야 할 일을 잘 분배해야합니다. 혼자 다 하려고 하면 이도저도 아닌 결과물들만 나옵니다. 그리고 우리는 위 예제의 진행을 통해 산업혁명 이후의 발전된 유통망 사회를 작게나마 구성했습니다.  
&nbsp;하지만 위 방법이 귀찮은 것도 맞습니다. 이거 만들려면 DI 코드들 다 만들어야하니까요. 그래서 `Dagger2`같은 프레임워크를 쓰는 것입니다. 거기엔 IMarket같은게 다 만들어져 있거든요. Dagger2에 관해서는 [다음 글](../dagger2-android-basic/)에서 확인해보겠습니다.

&nbsp;이 글에서는 제가 나름대로 이해한 DI를 적었습니다. 많은 분들이 이 글을 보고 좋는 느낌을 얻어가셨으면 좋겠습니다. DI는 확실히 손이 많이 가는 설계이지만, 알아두면 많은 부분에서 새로운 영감을 얻으실 것입니다.