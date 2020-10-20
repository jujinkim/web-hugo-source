---
title: "What is DI(Dependency Injection)?"
date: 2020-10-18T01:12:39+09:00
draft: true
---
&nbsp;This post is for those(especially me) who don't really know what DI (Dependency Injection) is before studying Dagger2.

## What is DI?

&nbsp;DI stands for Dependency Injection, and is a programming design technique that **injects the dependency of an object from the outside**. Yes, maybe you don't understand like me. I don't understand what to inject, why to use it. So I wrote this post.

&nbsp;Before difining the DI features, le'ts imagine a tiny situation in our daily life.

> You are hungry now, so you entered into a snack bar and ordered remen.

&nbsp;The chef will start cooking the remen. Ingredients(preperation) are water, noodles, sauce, and a pot. The cooking process will be "Pour water in the pot, put sauce and noodles, and boil".  
&nbsp;Where did the chef get ingredients? Of cource these were brought from outside. The chef didn't make water, sauces, noodles, or pots. Like this, being supplied objectives from outside is called DI (Dependency Injection).

> The chef relies on ingredients to make ramen. Without these, the chef can't make ramen. And the chef gets these from the outside. In other words, it can be said that ingredients are injected from the outside to the chef. That's what we call dependency injection.

&nbsp;If the chef doesn't buy ingredients, the process of cooking ramen becomes a little complicated. "Make water, make noodles, make sauce, make a pot. And put all ingredients into the pot, and boil it" will be the cooking process.  
&nbsp;It might look good because the chef does everything, but it's not that good actually. Because the chef is literally a person who cooks, not makes ingredients. Coding is similar. In codes, 'seperation of roles' is important, which means that a method should be made to do only one work, and a class to be for a specific purpose. This is to minimize unneccessary modifications of code and to make possible to reuse and test. In real world, even when we work, the effeciency of work increases when we focus on only one thing rather than focus on many things. For some reasons, seperation of roles is important.

&nbsp;Now let's write codes that the chef cooks ramen.

---
## The chef who doesn't use DI
First, let's define ingredients.

```kotlin
// Ingredients and pot
class Noodle
class Sauce
class Water
class RamenBowl {
    fun makeRamen(water: Water, sauce: Sauce, noodle: Noodle): Ramen {
        return Ramen(water, sauce, noodle)
    }
}

// Ramen (result)
class Ramen(water: Water, sauce: Sauce, noodle: Noodle)
```

&nbsp;In the normal way(without DI), this code is a difinition of Chef class that cook ramen.

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

&nbsp;`Chef` knows how to cook ramen with `cookRamen()`. But, in this code, the process of cooking ramen includes making noodles, sauce, water and a pot. It's strange. It's like I order rice and the chef starts farming. And, what's happened if `cookRamen()` method is being tested? It tests not only cooking ramen, but also making ingredients. It's strange, even chefs are not tested  making ingredients at Master Chef.

&nbsp;At the sight of codes, there are other problems if codes are written like above. If there is another chef who cooks other foods that needs water, when parameters of water's constructor is changed, `all codes that generates water at all Chef classes should be modified as well`. In addition, there could be duplicated object generation codes(`val water = Water()`) in many classes.

&nbsp;We are going to fix this problems using DI by 4 steps.

---
## 1st step of DI: Provides ingredients from outside.
&nbsp;As mentioned at above, the chef didnt't make ingredients. So where did the chef get these? Perhaps the boss gave them and ordered the chef to cook.

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

&nbsp;Now the chef doesn't make ingredients, but cooks with ingredients which is given by someone. Only the way of cooking could be tested when testing the method. But there's still a problem. Somebody that orders to cook should got and gave ingredients. Parameters of `cookRamen()` must be filled when it is called. Actually, the person who orders the chef won't make ingredients.

---
## 2nd step of DI: Create a class that provides ingredients.
&nbsp;So where should ingredients taken from? Maybe these were taken from the market. So we will create a market that provides ramen ingredients.

```kotlin
class Market {
    fun getNoodle(): Noodle = Noodle()
    fun getSauce(): Sauce = Sauce()
    fun getWater(): Water = Water()
    fun getRamenPot(): RamenPot = RamenPot()
}
```

&nbsp;Now there is a market, so the chef is able to get ingredients from market and cook.

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

&nbsp;The chef or the boss no longer needs to make ingredients themselves. They just need to get ingredients from the market.

&nbsp;It's still a little awekward though. Usually markets don't make products to be sold, right? Markets receive stuffs and sell them. Markets only do distributes, and there are other places that produce stuffs. Some markets also provides that 'ordering the wishlist or stuffs that ordered frequently'.   

---
## 3rd step of DI: Divide classes that 'provide', 'distribute', 'use' ingredients.
&nbsp;"Market" is a places that distributes products. Places that make products are not markets. Therefore, we need to modify `market` to receive products(ingredients) from other sources. Let's rewrite the code by dividing where to make, sell, and use.

```kotlin
// Classes that create each ingredients(or tool)
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

// Classes that distributes stuffs
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

// Class that uses stuffs
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

&nbsp;Now we see the modern society that developed after the industrial revoslution at the code.
- There are creaters of each products.
- There is a market that roles of distributor. A market receives products from creaters, and pass them to consumers.
- Maybe some consumer uses the market often. It provides that ordering stuffs at once that ordered frequently.
- The chef is now do only "cooks ramen" when instructed to make ramen.

&nbsp;The only thing that left is just make consumers recognize the market as "Market". I mean, `Market` class above is not a market, but the class which named "Market" that gives ingredients. However, since `Market` is a market, so set methods that only markets can do using interface, and make consumers order to the "Market", not a class named Market.

---
## 4th step of DI: Make 'Market' a real 'Market'
Let `Market` inherit the market interface, and let the chef only knows the interface. This is called 


---
## What is IoC(Inversion of Control)?
 많은 DI 글들을 보면 꼭 빠지지 않는 개념으로 IoC(제어의 역전)이 있습니다. 글 마지막 부분에 이 개념을 쓴 이유는, 어차피 아무것도 모른 상태로 “제어를 역전한다” 이런 말 써봐야 저부터가 이해가 안되기 때문입니다.

 위 예제의 마지막에서는 이미 제어가 역전된 듯한 모습을 보여주긴 합니다. 요리사(소비자)가 직접 필요한 객체를 초기화하지 않았습니다. 그저 요리사는 추상화된 마켓에 “객체를 달라"고 호출만 했고, 마켓에서는 요청받은 객체를 알아서 만들어서 주입해줬죠. 즉, 요리사가 필요한 객체를 제어하는 입장에서, 제어된 객체를 받는 입장이 되어버린것이지요. 요리사는 객체가 어떻게 만들어진건지, 이 객체가 유일한지 싱글톤인지도 모릅니다. 그저 받아서 쓰기만 하면 될 뿐입니다.

 사실 제어의 역전 개념은 이런 예제정도로는 완전히 설명하기 어렵습니다. 용어부터가 복잡하잖아요. 그리고 제어의 역전은 추상화가 들어가야합니다. 하위 클래스가 스스로 사용하려는 객체를 구체적으로 몰라도 되어야하기 때문입니다.


---
## Conclusion
- DI: In some class, request and get instances that being used even don't know how instances are generated.
- IoC: Write codes that could be called from something. The other class controls the codes which should be controlled from the caller originally.

 산업혁명 발전 과정 중에는 “소품종 대량생산"이 있습니다. 이것은 각자 해야 할 일을 잘 분배해야합니다. 혼자 다 하려고 하면 이도저도 아닌 결과물들만 나옵니다. 그리고 우리는 위 예제의 진행을 통해 산업혁명 이후의 발전된 유통망 사회를 작게나마 구성했습니다.
 하지만 위 귀찮은 것도 맞습니다. 이거 만들려면 DI 코드들 다 만들어야하니까요. 그래서 Dagger2같은 프레임워크를 쓰는 것입니다. 거기엔 IMarket같은게 다 만들어져 있거든요. Dagger2에 관해서는 다음 글에서 확인해보겠습니다.

 이 글에서는 제가 나름대로 이해한 DI를 적었습니다. 많은 분들이 이 글을 보고 좋는 느낌을 얻어가셨으면 좋겠습니다. DI는 확실히 손이 많이 가는 설계이지만, 알아두면 많은 부분에서 새로운 영감을 얻으실 것입니다.