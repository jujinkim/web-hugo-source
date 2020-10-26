---
title: "Difference between Framework and Library"
date: 2020-10-26T00:50:14+09:00
draft: true
---


## Framework
&nbsp;`Framework` is a tool that wraps up whole project codes. When you create some statues, you may make a skeleton(framework) first and build the rest upon the skeleton. Projects are also not different, writing codes to fit the framework. It's like writing codes into the framework.

&nbsp;`Framework` decides the architecture or style, development plan of the project. Developers are programming within the framework. Some project codes invoke methods of the framework, but, usually, projects are developed using methods that are provided by interfaces from the framework. It can be seen that 'inversion of control(IoC)' is applied in that the framework controls project codes.

&nbsp;When the `framework` is decided once, it is hard to be replaced with the other framework. It needs to modify whole codes to apply to replace to the new framework.

Dagger2, .Net Framework, express.js, Spring, react, vue.js, etc.

호출 방향: `Framework` &#8594; `Project codes`

---

## Library
&nbsp;`라이브러리`는 개발자가 필요에 의해 호출하는 도구입니다. 한국어로는 `도서관`입니다. 공부, 연구, 작업 등을 하다가 필요한 문서나 자료가 필요할 경우 책을 가져다가 참고하듯, 필요할 때 꺼내어 사용한다고 생각하면 됩니다.

&nbsp;`라이브러리`를 호출한다고 해서 프로젝트 코드의 전반적인 설계나 스타일이 바뀌지는 않습니다. 그저 프로젝트에서 어떤 기능을 구현하기 위해 사용되는 다른 코드일 뿐입니다.

&nbsp;`라이브러리`는 필요에 따라 다른 것으로 대체할 수 있습니다. 프로젝트에서 필요한 기능에 더욱 부합하는 라이브러리가 나온다면 얼마든지 대체가 가능하며, 해당 기능 또는 모듈에서의 라이브러리 호출&적용 코드만 수정해주면 보통은 문제가 없습니다.

Retrofit, Volley, Glide, etc.

호출 방향: `Library` &#8592; `Project codes`