---
layout: post
title:  "heroku에 socket.io앱 deploy실패기"
author: "lastgleam"
---

# heroku에 socket.io앱 deploy실패기

## 먼저 heroku란,

간단히 레포지토리를 업로드하는 것만으로도 웹서버의 역할 + DNS서버의 역할을 자동으로 설정해주는 PaaS이다. 한개의 앱 당 500mb의 저장공간을 제공해주어서 미니프로젝트를 업로드 해서 공개하는 경우에 많이 쓰이는 편인 듯. 특히 node.js 프로젝트를 잘 지원해주고 있는 듯하다.

## build, deploy success 그러나 에러
`socket.io`를 익힐 겸 간단히 프로그램을 만들었는데, heroku를 이용해서 빠르게 자랑(?)해보고자 하였다.  지난주 참가했던 [`LINE Developer Bootcamp - Node.jsでChatbot開発超入門`](https://line.connpass.com/event/78432/)에서 heroku를 써봤었기 때문에 사용법은 알고 있던 터였다. 이러저러 설정을 거친 후 `git push heroku master`로 deploy, 생성된 URL로 접속해보니!, 다음과 같은 에러가발생했다. 역시나 한큐에 잘되는 건 없는 것이었다...

![에러..](/assets/images/2018-02-27-heroku-error.png)

에러가 발생했을 때, 나는 
1. 공식홈페이지의 도큐멘트를 찾아 읽고 스스로 해결하기 
2. 스택 오버플로우의 초록색체크 답변대로 해보기 
3. 개발자들이 기술 블로그에 써놓은 안내대로 해보기
4. 사수에게 직접 상황을 공유하고, 해결방법을 구하기

순으로 에러를 해결하려고 하는 편이다.

먼저 heroku 공식가이드를 살펴보았다.
[Using WebSockets on Heroku with Node.js - **Option 2: Socket.io**](https://devcenter.heroku.com/articles/node-websockets#option-2-socket-io)

`package.json`을 만드는 것부터 차례차례 자세히 작성되어 있으나, 딱히 내 프로젝트와 다른 방법으로 구현된 부분은 없어보였다.

불현듯, heroku의 로그를 보면 에러 로그가 남아있지 않을까 해서 `heroku logs`로 확인해보니 다음과 같은 로그가 남아있었다...!
```
.
.
.
2018-02-27T04:12:10.000000+00:00 app[api]: Build succeeded
2018-02-27T04:12:38.434381+00:00 heroku[web.1]: Starting process with command `npm start`
2018-02-27T04:12:40.543488+00:00 heroku[web.1]: Process exited with status 1
2018-02-27T04:12:40.481510+00:00 app[web.1]: npm ERR! missing script: start
2018-02-27T04:12:40.488160+00:00 app[web.1]: 
2018-02-27T04:12:40.488390+00:00 app[web.1]: npm ERR! A complete log of this run can be found in:
.
.
.
```
## Procfile!
start커맨드가 `node index.js`여야 하는데 `npm start`로 시작되고 있던 것. 바로 'web : node index.js'라고 작성한 `Procfile`을 root디렉토리에 생성 후 다시 deploy했다.

결과, 

![heroku-success](/assets/images/2018-02-27-heroku-success.png)

제대로 동작하고 있는 게 확인 되었다.

드디어 아래의 URL로 전세계 누구나 액세스 가능하게 되었다!

https://socket-chat-lastgleam.herokuapp.com/

## 에필로그

처음에 heroku의 웹사이트에서 직접 빈 프로젝트를 작성하면 `heroku remote`부터 시작하는 deploy절차가 표시된다. 그 절차에는 start script가 `npm start`일 때를 디폴트로 갖고 있어서, Procfile을 생성하여 설정을 바꿔줄 필요가 있던 것.
Procfile은 heroku의 start command를 변경하는 방법이니까, `package.json`의 `scripts.start`안에 `web : node index.js`라고 작성해두면 굳이 heroku에서 설정때문에 고생할 필요도 없고, heroku가 아닌 다른 방식으로 배포할 때에도 문제될게 없을 듯 하다.
아무래도 아직 node.js와 javascript에 대한 경험이 별로 없어서 겪은 일이라고 생각된다. 이런 사소한 착오가 앞으로도 몇번 더 있을 것 같다.

