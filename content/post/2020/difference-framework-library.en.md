---
title: "Difference between Framework and Library"
date: 2020-10-26T00:50:14+09:00
draft: false
---


## Framework
&nbsp;`Framework` is a tool that wraps up whole project codes. When you create some statues, you may make a skeleton(framework) first and build the rest upon the skeleton. Projects are also not different, writing codes to fit the framework. It's like writing codes into the framework.

&nbsp;`Framework` decides the architecture or style, development plan of the project. Developers are programming within the framework. Some project codes invoke methods of the framework, but, usually, projects are developed using methods that are provided by interfaces from the framework. It can be seen that 'inversion of control(IoC)' is applied in that the framework controls project codes.

&nbsp;When the `framework` is decided once, it is hard to be replaced with the other framework. It needs to modify whole codes to apply to replace to the new framework.

Dagger2, .Net Framework, express.js, Spring, react, vue.js, etc.

Call direction: `Framework` &#8594; `Project codes`

---

## Library
&nbsp;`Library` is a tool that called by developers when they need it. In a real library, you can refer any books while you are studying, researching, or working on something. Like a real library, you can call and use features from the library whan you need.

&nbsp;It doesn't need to modify whole project's structure, architecture or style while libraries are used. They are just other codes(groups of features or methods) to implement some features in the project.

&nbsp;`Library` can be replaced anytime if it needed. If new libraries which are more fit to the project, then libraries can be replaced easily. It is not a big deal that replacing libraries. Just replacing some codes that calling or applying the library codes from some modules or methods is all things to replace a library.

Retrofit, Volley, Glide, etc.

Call direction: `Library` &#8592; `Project codes`