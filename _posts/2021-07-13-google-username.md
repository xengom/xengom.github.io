---
title: google 로그인 시 username이 사용자 컴퓨터 이름으로 나오는 문제
toc: false
category: troubleShooting
tags:
- spirng
- springboot
- Mustache
---

![사용자이름](/assets/images/7/username.PNG)
제목 그대로 google 로그인을 만들었을 때 제대로 넣어줘도, 사용자 컴퓨터 이름으로 보이는... 이상한 현상이 생겼다.

진짜 별의 별거 다해봐도 안되서 찾아봤는데

밑의 링크에서 해결책을 찾았다.

[깃헙링크](https://github.com/jojoldu/freelec-springboot2-webservice/issues/549)



username이 어디선가 사용되서... 인걸로...

user로 변경하니 잘 된다.

병주좌 감사합니다..



-> user는 나중에 sessionUser에서 사용하니  googleUser로 변경

~~~java
@GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user){
        model.addAttribute("posts",postsService.findAllDesc());
        if(user!=null){
            model.addAttribute("googleUser", user.getName());
        }
        return "index";
    }
~~~



~~~html
{{>layout/header}}
    <h1>스트링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <div class="row">
            <div class=col-md-6">
                <a href="posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#googleUser}}
                    Logged in as <span id="user">{{googleUser}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/googleUser}}
                {{^googleUser}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
                {{/googleUser}}
            </div>
        </div>
        <br>
        <!-- 목록 출력 영역 -->
        <table class="table table-horizontal table-bordered">
            <thead class="thead-strong">
            <tr>
                <th>게시글번호</th>
                <th>제목</th>
                <th>작성자</th>
                <th>최종수정일</th>
            </tr>
            </thead>
            <tbody id="tbody">
            {{#posts}}
                <tr>
                    <td>{{id}}</td>
                    <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                    <td>{{author}}</td>
                    <td>{{modifiedDate}}</td>
                </tr>
            {{/posts}}
            </tbody>
        </table>
    </div>
{{>layout/footer}}
~~~





참조 : [깃헙링크](https://github.com/jojoldu/freelec-springboot2-webservice/issues/549)
