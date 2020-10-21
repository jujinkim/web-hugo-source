---
title: "Utterances 덧글 적용하기 (Hugo)"
date: 2020-10-13T22:45:58+09:00
---

&nbsp;보통 네이버나 티스토리같은 블로그 플랫폼들은 자체 덧글 시스템을 지원합니다. 하지만 덧글 시스템은 서버와 DB가 필요하기 때문에, 정적 블로그 싸이트에서는 바로 적용할 수 없습니다. 대신 정적 블로그 싸이트에서 덧글 시스템을 적용하기 위해선 외부 서버의 도움을 받는 덧글 서비스를 스크립트로 적용해야합니다.  
덧글 서비스는 여러가지가 있는데, 이번 글에서 소개할 서비스는 **[utterances](https://utteranc.es)** 라는 앱입니다.

---
# uterrances?
&nbsp;uterrances는 GitHub issue기능을 활용한 정적 덧글 서비스로, 특정 페이지 또는 포스트마다 GitHub issue를 생성하고 그 issue에 달린 커멘트를 페이지에 보여주는 서비스입니다. 기본적으로 GitHub issue를 사용하기 때문에 GitHub repo가 필요하며, 그 repo에 utterances를 설치&허용해줘야합니다. 또한 GitHub 계정으로 덧글을 달 수 있습니다.

---
# 왜 utterances 인가?
&nbsp;정적 페이지에 덧글 기능을 도입하는 서비스는 많습니다. 대표적으로 Disqus가 있고, HTML Comment Box는 가벼움을 자랑하지요. 국산으로는 LiveRe를 사용해 SNS 연동이 가능합니다.   

&nbsp;하지만 Disqus는 무겁고, SNS 로그인을 따로 해줘야하며, SNS 특성상 익명성을 조장한 악성 덧글과 스팸성 덧글을 피하기 어렵습니다. LiveRe 역시 SNS 연동 덧글 시스템이 가지는 단점을 그대로 가지고 있습니다. HTMLCommentBox는 가볍고 SNS 연동도 불필요하지만, 관리가 어려우며 완전 익명이기에 스팸에서 더욱 자유롭지 못하다고 생각합니다.  
&nbsp;물론 이 덧글 서비스들이 안좋은 서비스라는건 아닙니다. 하지만 상기한 문제점 등이 있어 불만이 없을수는 없습니다.

&nbsp;utterances는 GitHub issue를 사용하는 앱(서비스)이기에, GitHub repo의 issue에서 쉽게 관리할 수 있습니다. 또한 GitHub 계정이 있어야 덧글을 달 수 있기 때문에 비교적 악성 덧글과 스팸에서 자유롭다고 생각합니다. 아무래도 GitHub 계정은 일반 SNS와는 다른 성질을 가지고 있기 때문에..

&nbsp;대신 GitHub 계정이 없는 사람은 덧글을 달기 위해 로그인을 해야하는데, GitHub 계정은 개발자가 아니면 보통은 가지고있지 않기 때문에 일반인들에게는 접근성이 조금 떨어진다는 단점이 있겠습니다.  
&nbsp;근데 정적 블로그 싸이트는 보통 개발 블로그 아닌가..? 아무튼, 정적 싸이트로 구성된 개발 블로그에 사용하기를 추천합니다. 특히 싸이트(블로그)가 GitHub Pages에서 호스팅되고 있다면 아주 좋습니다.

---
# utterances 적용하기
&nbsp;utterances를 적용하는 것은 매우 쉽습니다. 실제로 이 서비스를 모르는 사람도 10분 내외면 적용할 수 있을 정도입니다.  
&nbsp;우선, 본 글에서는 Hugo 정적 싸이트(블로그) 플랫폼을 기반으로 설명드립니다. 그런데 정말 적용이 쉬워서, Jekyll같은 다른 플랫폼을 사용하시는 분들도 이 글만 보고 바로 알아보고 적용하실 수 있을 것입니다.  

&nbsp;utterances를 적용하는 절차는 간단하게 네개로 분리됩니다. 

    1. GitHub repo에 utterances 설치
    2. utterance 설정 및 코드생성
    3. 코드를 싸이트의 comment 구현 파일에 붙여넣기
    4. 적용!

## utterance 설치하기
&nbsp;우선 utterances를 적용할 GitHub repo를 하나 만듭니다. 만약 싸이트가 GitHub pages에서 호스팅되고있다면, 그 repo를 그대로 사용해도 좋습니다.  
https://github.com/apps/utterances 에서 해당 repo에 앱 설치를 해줍니다.  

## utterance 설정하기
&nbsp;https://utteranc.es 에 접속합니다.  
&nbsp;스크롤을 내리다보면 **configuration** 부분이 있는데, 여기에다가 위에서 utterance를 설치한 repo 이름을 적어줍니다.  
&nbsp;**Blog post <--> Issue Mapping** 부분은 GitHub issue를 어떻게 만들지 정하는 부분입니다. 보통은 초기 설정을 이용하면 되지만, 알고 있다면 특정한 경우에 더 유연하게 적용이 가능할 것입니다.

- Issue title contains page pathname - issue 제목이 페이지 path입니다. 동일한 path에서는 동일한 덧글을 보여줍니다. 즉, 도메인이 달라도 같은 path면 같은 페이지로 인식합니다.
- Issue title contains page URL - issue 제목이 페이지 url입니다. 도메인이 다르면 다른 페이지로 인식합니다.
- Issue title contains page title - issue 제목이 페이지 제목입니다. 페이지 제목만 보고 덧글을 구분합니다. 페이지 제목이 달라지면 기존의 덧글을 인식하지 못합니다.
- Issue title contains page og:title - Open Graph title meta 데이터로 페이지를 구분합니다.
- Issue number - issue를 새로 안만들고 원래 있던 issue를 덧글처럼 연결합니다.
- Specific term - 직접 지정한 단어로 issue를 만들고 덧글로 씁니다. 블로그 포스트 말고 다른 정적 페이지에 넣을 때(방명록이라던지..) 이 설정을 추천합니다.

&nbsp;**Issue Label**에 단어를 지정하면, utterances가 덧글을 위해 issue를 만들 때 label을 붙입니다. GitHub repo의 issue에 utterances 외 다른 issue도 올라올 수 있다면 여기에 임의의 라벨을 붙여서 구분하기 편하게 하면 됩니다.  

&nbsp;**Theme**는 말 그대로 덧글란의 테마입니다. 테마를 선택하면 바로 미리보기가 보이니 적당한거 골라서 적용하면 되겠습니다. Preffered color scheme는 아마 웹브라우져의 다크/라이트 상태를 읽어오는게 아닐까 싶네요. 

## 코드 붙여넣기
&nbsp;위에서 설정을 다 마치면 **Enable Utterances** 에 코드가 생성이 되어있을 것입니다. 스크립트 파일이기 때문에, 해당 코드를 그대로 덧글이 들어가야 할 곳에 붙여 넣어주면 됩니다.  
&nbsp;Hugo의 경우 테마마다 다르기 때문에 각 테마 설명서를 확인해보시길 바랍니다. (PaperMod 테마의 경우 /layout/partials/comments.html 파일에(없으면 만듭니다) 넣어주면 됩니다.)

---
# 적용!
&nbsp;벌써 모든 절차가 끝났습니다. 그저 생성된 싸이트를 서버에 올려주기만 하면 그대로 적용이 됩니다. 해당 페이지로 들어가보시면 GitHub 덧글란이 생겨있는 것을 확인할 수 있으며, 테스트로 덧글 하나 달아보시면 GitHub repo에 새 issue가 생기고 거기에 덧글이 달려있는 것을 확인할 수 있습니다!

---
만약 GitHub 이용자가 자주 방문할만한 글들을 담고있다면, 지금 utterances를 적용해보세요!