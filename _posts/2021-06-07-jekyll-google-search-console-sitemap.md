---
title: Jekyll 블로그 구글 서치 콘솔(Google Search Console) Sitemap 가져올 수 없음 문제 해결
category: Gitblog
tags:
- Github
- Page
- sitemap
- google
- search
- xml
- jekyll
- blog
---

## 문제

![가져올 수 없음](/assets/images/6/sitemap1.PNG)

새로 도메인을 구매한 후, 구글 서치 콘솔에 등록하려하니 Sitemap을 가져올 수 없음 문제가 발생했다. 하지만 검색을 아무리 해도, 죽었다 깨어나도 해결방법을 찾을 수 없었다.

구글링해서 나오는 검색결과는 Tistory 블로그 기반이라 Tistory에서 해결하는 방법(뭐 구 에디터를 써라? 이런거)을 알려주는 게 태반이였고 나머지는 본인 도메인/sitemap 을 주소창에 입력했을 때, 애초에 sitemap.xml이 출력되지 않는 경우였다.

그치만... 난 xengom.com/sitemap치면 잘만 나오는데....라고 구시렁거리면서 3시간을 삽질한 결과, 원인을 찾았다.



## 해결

원인은 sitemap에 써있는 URL.

지금은 xengom.com을 쓰지만 커스텀 도메인을 도입하기전에 xengom.github.io를 사용한지라 _config.yml에 github.io가 들어가 있었고, gemfile을 통해 sitemap을 자동생성하는 방식이라 _config.yml에 있는 URL을 기반으로 sitemap.xml이 생성되어 sitemap이 제대로 생성이 안된거였다.

물론 주소창에 xengom.com/stiemap을 치면 자동으로 xengom.github.io/sitemap페이지를 불러다 주기때문에 문제파악을 못한거였고....

![성공](/assets/images/6/sitemap2.PNG)

![결과](/assets/images/6/result.PNG)

허무 그 자체...
