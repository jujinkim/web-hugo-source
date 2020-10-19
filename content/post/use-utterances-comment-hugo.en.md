---
title: "Use Utterances comment at Hugo"
date: 2020-10-13T22:45:58+09:00
---

&nbsp;Usually, some blog platforms such as WordPress and Blogger support their own comment system. However, since the comment system needs a server and DB, it is not able to be applied directly to a static web page. So we need a comment service that using an external server and script.  
There are a bunch of comment systems, We're gonna take a look the service that named **[utterances](https://utteranc.es)** in this post.

---
# uterrances?
&nbsp;utterances is a comment service that using the GitHub issue. It creates GitHub issue for each page or post, and display its comment on the page. utterances uses GitHub issue basically, so utterances is needed to be installed and allowed into the repo. People can also comment with their GitHub account.

---
# Why utterances?
&nbsp;There are so many comment services for static pages. Disqus, HTML Comment Box, etc. In Korea, 'LiveRe' provides the social media comment service.

&nbsp;But, Disqus is too heavy and needs a social media account. There could be also malicious comments. LiveRe also has the disadvantages of the social media comment system. HTMLCommentBox is lightweight and doesn not require social media account, but it's difficult to manage and not free from malicious comments as well.
&nbsp;Of course these services are not bad, but there are problems that mentioned above.

&nbsp;utterances could be managed easily at GitHub issue from its repo since it is a service that uses GitHub issue. And it needs a GitHub account to leave a comment, so I think there should be fewer bad comments than other services..

&nbsp;But, It requires GitHub account so people who don't have GitHub account are hard to leave comments.
&nbsp;Hmm..anyway a static blog site are normally a tech blog..right? Whatever, I recommend that this service should be applied at the tech blog which is builded by static site.(Especially the site that is hosted from GitHub pages)
   
---
# Add utterances
&nbsp;Applying utterances to your site is very easy. In fact, even those who don't know this can add into their website in about 10 minutes.
&nbsp;First of all, in this post, I'll explain based on Hugo. But it is able to apply to the other service such as Jekyll easily.

&nbsp;There are 4 steps to apply uterances.

    1. Install utterance app to the GitHub repo
    2. Set options and generate utterance code.
    3. Paste the code into the comment area of the webpage.
    4. Done!

## Install utterance
&nbsp;First, create a GitHub repo to apply utterance to. If the site is hosted on GitHub pages, you can use the repo as is.
Install the app on the repo at https://github.com/apps/utterances.

## Set up utterance
&nbsp;Go to https://utteranc.es   
&nbsp;There is a section **configuration**, Type the repo's name that is installeld utterances into here.
&nbsp;**Blog post <--> Issue Mapping** is the part that how GitHub issue should be created. Usually the default setting could used, but it is better to know the other setting to use this app flexibly.

- Issue title contains page pathname - issue title is a page's path. There will be same comments on the same path. This means, comments will be same even the page's pateh are same with different domains.
- Issue title contains page URL - issue title is page's url. If domains are different, comments are different too.
- Issue title contains page title - issue title is a page's title. Recognize and separate comments using only the page's title. If page's title is change, comments won't be recognized.
- Issue title contains page og:title - Identify pages using Open Graph title meta.
- Issue number - Use the issue that created already, not create new issue.
- Specific term - Use a custom term to identfy the page. Recommended to the webpage that is generated manually(like guest book), not posts

&nbsp;If you type a word at **Issue Label**, utterances will add the label into the generated issue. It's good to use this option if the repo's issue is used to gather the user's report.  

&nbsp;**Theme** is the theme of comoments section literally. You may notice the changes after select other theme, so pick the theme what you like.  

## Paste codes
&nbsp;A code will be generated at **Enable Utterances** section. Just copy it and pate into your site's comment section.  
&nbsp;In case of Hugo, the location to paste the comment code is demend on theme, so check each theme's documents. (PaperMod theme - paste it into /layout/partials/comments.html file. (make a file if it isn't existed))

---
# Done!
&nbsp;Wow, Everythings are done! Just upload the generated files into the server and you may see the comment section that using GitHub. And, just leave a test comment and check that there is a new issue at GitHub repo with the comment!

---
&nbsp;If your site have articles that GitHub users will visit frequently, try applying utterances now!