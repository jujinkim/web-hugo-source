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


---
## 2nd step of DI: Create a class that provides ingredients.

---
## 3rd step of DI: Divide classes that 'provide', 'distribute', 'use' ingredients.

---
## 4th step of DI: Make 'Market' a real 'Market'

---
## What is IoC(Inversion of Control)?

---
## Conclusion
- DI: In some class, request and get instances that being used even don't know how instances are generated.
- IoC: Write codes that could be called from something. The other class controls the codes which should be controlled from the caller originally.

