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
## 3rd step of DI: Divide classes by 'provider', 'distributer', 'user'.
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
&nbsp;Let `Market` inherit the market interface, and let the chef only knows the interface. This is called `Dependency inversion principle` or `Separation of concern` (Two concepts aren't same exactly, but they could be used for the DI). Another difficult thing came out. Separation? Dependency? In fact, I wrote the 3rd example code as intuitively as possible for easy understanding. But you know that the actual market network is much more complex. So we need to separate between markets and consumers.  
&nbsp;"Separation" means that, literally, makes two classes don't know each other. And also means that two classes don't refer each other as well. This thing is needed because the injected class knows the injecting class to much. In the real world, consumers don't know markets well. Consumers only know services of markets.

&nbsp;You'll need polymorphism as your project grows bigger. The implementation of DI is also no exception. In your real project, There will be more than four classes that provide objects(ingredients), and there will be more than one class that acts like a market. But they do similar things. Some classes do "providing", some classes go "distributing", and some classes do "using(consuming)". These common behavios must be categorized into an interface. For example, let's assume that there are two market classes which called "food market" and "electronics market". These two markets have different items definetely. However, the role(action)s are same, selling(injection). And also, provider classes such as "~~Maker" do the same behavior, provide(make). By grouping these behaviors into interfaces, the chef class doesn't need to know that how "Market" class is implemented. Just knowing an abstract market interface, and just request objects to the interface and it could be injected(got) objects that it needed.

```kotlin
// Classes that create ingredients do common action, "making(providing)".
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

// Classes that distribute ingredients do common action, "dustributing(passing)".interface IMarket {
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

// Class that consumes ingredients
class Chef(val market: IMarket) {
    lateinit var water: Water
    lateinit var sauce: Sauce
    lateinit var noodle: Noodle
    lateinit var pot: RamenPot

    init {
        // Chef doesn't know 'market' is a class "Market'
        // But only knows that is is an IMarket interface.
        market.passIngredients(this)
    }

    fun cookRamen(): Ramen {
        val ramen = pot.makeRamen(water, sauce, noodle)
        return ramen
    }
}
```

&nbsp;Awesome, isn't it? This is DI. You may noticed that DI is not for only some specific languages, but for an architecture of codes.  
&nbsp;What if this project grows up without using DI? There will be more typs of chef and ingredients. Then, every time we need to modify ingredients, we have to modify all of chefs. It will become annoying lataer on if we manage many ingredient classes and their's constructors. Y'all guys may know that maintenance is harder than developing new features.

---
## What is IoC(Inversion of Control)?
&nbsp;In the most of DI articles, there is a concept which called "IoC(Inversion of control). I wrote down this concept at the end of the post, because you(and I) wouldn't understand "inverse something controlled blahblah" before we know about above things.

&nbsp;At the last code, it shows that the control has been inversed. The chef(consumer) didn't initialize required objects itself. The chef simply called the abstraced market to "give me ingredients(objects)", and the market passed(injected) requested objects. The chef doesn't know how objects were created, whether these objects are unique or a singleton or not. All the consumer class needs is that just get objects and use them.

&nbsp;Actually, the concept of IoC is difficult to fully explain with these tiny examples. Let's learn and get to know about it together.

---
## Conclusion
- DI: In some class, request and get instances that being used even don't know how instances are generated.
- IoC: Write codes that could be called from something. The other class controls the codes(object) which should be controlled from the caller class.
- Why we use this?: Let's think that Di will gives an industrial revolution to your code.

&nbsp;In the process of the industrial revolution, there is "high-volume, low-variety production". It is neccessary to distribute works by each persons well. If somebody tries to do all things, it would be ruined. We have progressed the developed society after industrial revolution through the exmaple code above.  
&nbsp;I know it is not easy to do these all things. To make this architecture, we have to make all the DI boilerplate codes. That's why we use frameworks like Dagger2. There are already implemented codes something like IMarket in Dagger2. We'll check out Dagger2 in the next post.

&nbsp;In this post, I wrote the DI that I understood in my own way. I hope may you will get a good feeling about DI after reading this post. DI is certainly a unfamiliar design for the person who faces this first time, but it will gives you a new inspiration in your codes.