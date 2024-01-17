---
title: "[AWS] AWS Lambda에 Docker Image로 Selenium을 올리기 위한 여정"
date: 2024-01-17 22:30:00 +0900
categories: [AWS, Lambda]
tags: [aws, lambda, docker, image, selenium, python]     
# TAG names should always be lowercase
toc: true
---
## 환경
Macbook M1 Pro

python 3.9

(인터넷에 나와있는 대부분의 포스팅은 3.6 버전 기준이 많다.. AWS에서 이제 더이상 지원을 하지 않는...ㅠㅜ)

## **서술 배경**
프로젝트를 진행할 때 동적 페이지를 크롤링 할 필요가 생겨서 Python의 Selenium을 사용할 필요가 있었고, 이미 로컬에서 크롤링하는 코드는 모두 만들었지만, 실제로 람다에 올리고, Selenium을 사용하기 위한 여정이 매우 멀고도 험난하였기에 이 기나긴 여정을 서술해보고자 한다...
> 이걸 보는 사람들은 다른 고생 안 하고 잘 끝내길 바랍니다...ㅠㅜ

참고로 내가 삽질한 과정은 Docker를 올리는 것도, Lambda에 추가하는 것도 아닌, **Selenium이라는 Python 라이브러리를 정상 작동하게 하는 과정이다.** 만약 Selenium에서 고통을 겪는 사람이 아니라면, 이 사이트에서 얻지 않아도 충분히 많은 정보를 얻을 수 있을 것이다.

**본 게시글은 2024년 01월 17일을 기준으로 작성한 글이며, Chrome이나 Lambda의 정책 변경으로 인해 그대로 따라한다고 해서 동작하지 않을 수 있습니다.** ~~나도 그랬거든~~

## **AWS Lambda란 무엇인가?**
![AWS Lambda](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5c/Amazon_Lambda_architecture_logo.svg/1920px-Amazon_Lambda_architecture_logo.svg.png){: width="100" height="100"}

단순히 말하면 Serverless로 함수를 실행할 수 있는 기능이다.
EC2와 같은 서버를 구축하지 않고, 그냥 함수를 작성하고, 실행할 수 있도록 구성되어있다는 뜻이다.

> **GPT**
>- AWS Lambda는 서버를 프로비저닝하거나 관리하지 않고도 코드를 실행할 수 있게 해주는 이벤트 기반의 컴퓨팅 서비스입니다.
>- 사용자는 단순히 코드를 작성하고 업로드만 하면, AWS Lambda가 코드를 자동으로 실행하고, 관련 리소스를 자동으로 관리해줍니다.



## **Docker란 무엇인가?**
![Docker](https://i.namu.wiki/i/VKd93nW8oth4Ju3BmU6Ffz4knVwyAExdOtpUgzPky2w9z5AYSiKEh3hxI_0hu7M-6MTcKmn8br_WIJfW02MeyC9g61_8axL17g8K5kVP-N812-KFgsAbUb-HS5Y8WQCV65Cmi2lk6qzENov-EYi3MA.svg){: height="100"}

여러 응용 프로그램들을 격리된 환경속에서 컨테이너라는 단위로 실행을 하고, 관리할 수 있는 오픈소스 프로젝트이다.

> **GPT**
>- Docker는 소프트웨어를 개발, 배포, 실행하는데 필요한 모든 것을 포함한 컨테이너를 생성하는 플랫폼입니다.
>- 이 컨테이너는 코드, 런타임, 시스템 도구, 시스템 라이브러리 등 서버에 필요한 모든 것을 포함하고 있어서, 어디에서든 동일하게 동작하게 됩니다.

## **왜 Docker Image를 사용하여 올려야 하는가?!**
Lambda를 Docker로 사용해야 하는 이유는 크게 2가지가 있다.
### 1. 서비스의 크기가 어느정도 크더라도 사용할 수 있다.
- Lambda에서 기본적으로 제한되는 최대 용량은 250MB이다. 이는 Python 라이브러리 중 어느정도 무겁고 다양한 기능을 제공하는 것 몇가지를 넣으면 더 이상 추가할 수 없으며, 필자도 이 제한으로 상당히 불편함을 느꼈다.~~(Pytesseract, PIL 등등 넣다가 더이상 안들어가서 망연자실...)~~

### 2. 의존성 관리가 용이하다.
- Lambda에 기본적으로 내장되어있는 라이브러리는 상당히 부족하다. 기본적인 작업에서 벗어나서 뭔가를 하려고 하면 꼭 없는 라이브러리를 추가해줘야 한다. 이 점에서 Docker를 사용하게 되면, 내부에서 직접 설치하고, 사용할 수가 있기에 편리하다.

### 3. Docker Image의 버전 관리 기능으로 체계적인 관리가 가능하다.
- Git을 생각하면 편하다. 깃도 커밋 단위로 변경사항을 관리하고 사용하는데, 이와 마찬가지로 Docker Image의 버전도 관리하여 사용이 가능하다.

## **Docker Image를 만드는 과정**
> 사실 Docker Image 만들기 귀찮아서 그냥 zip으로 람다에 올렸는데, 안되고 어디서 문제인지 알기 어렵더라... 될 줄 알고 몇 일을 투자했는지...ㅠㅠ

사실 Selenium을 Lambda에 올리는 과정이 적힌 포스트를 한 10개는 본 것 같다. (더 많이 찾아봤는데, 아무리 봐도 더 찾기 어려웠다.)
그냥 Docker 이미지를 Lambda에 올리는 것을 찾아볼 수도 있는데, 굳이 Selenium을 찾은 이유는 Selenium에서 필요한 Chrome-Driver 때문이다...~~(진짜 얘가 제일 나쁜놈이야)~~
이 글을 보러 온 사람들이라면, Selenium에 대한 어느정도의 이해가 있을테니 더 설명하지는 않겠다 ㅠㅠ

그렇게 해서 찾은 한 줄기 광명

<https://uiandwe.tistory.com/1361>

이 사이트에서 순서까지 따라할 필요는 없고 (Docker 명령어를 모른다면 얌전히 따라하자) 작성자의 github 레포를 보고 감격을 할 수 밖에 없었다. 그 어떤 사이트보다 정말 필요한 파일만 딱 들어있다. 감사하며 사용하자! (감사합니다 ( _ _ ))

Git Repository를 클론하여 다운을 받자(스타도 드리자)

그대로 빌드를 하면 오류가 발생할 것이다. 한번 해보자(내가 왜 이 글을 쓰게 되었는지를 알 수 있다. 아! 도커는 알아서 받아주자.)

<https://www.docker.com/products/docker-desktop/>

도커를 받으려면 위 사이트에서 받되, 기타 설정은 다른 사이트를 참고하자.

## **무슨 에러가 발생할까**
- 아래 명령어를 입력해서 Docker Image를 만들어주자.

``` bash
docker build --platform linux/x86_64 -t selenium-lambda:0.1 -f Dockerfile .
```
입력하게 되면
![도커 이미지 빌드](/assets/img/posts/2024-01-17/이미지_빌드.png)
다음처럼 뭔가가 엄청나게 뜨면서 빌드가 진행될 것이다. 이렇게 빌드가 되는 베이스는 Dockerfile 파일이다. 무슨 순서로 진행되는지 알고싶다면, 해당 파일을 열어서 열심히 분석해보자!(쉽다!)


- 만들어진 Docker Image를 기반으로 컨테이너를 실행하자.
Lambda로 바로 넣지 않고 컨테이너를 만들어서 실행해보는 이유는 제대로 실행되는걸 넣어야 제대로 돌아갈 것이기 때문이다.(너무 당연한 말인가..? ㅎ)

``` bash
docker run --platform linux/amd64 -p 9000:8080 selenium-lambda:0.1
```

run: docker를 실행하는데

--platform linux/amd64 : linux/amd64 기반에서 실행을 할 것이고

-p 9000:8080 : 외부 포트 9000으로 접속하면 포트 포워딩을 통해 내부 포트 8080으로 전달이 될 것이며

selenium-lambda:0.1 : image는 방금 내가 빌드했던 selenium-lambda의 0.1 태그 이미지를 선택하겠다.

라는 의미이다. 실행을 해보자!

![실행완료](/assets/img/posts/2024-01-17/실행을_해보았다.png)

이렇게 나오면 제대로 실행이 되고 있다는 뜻이다. 현재 상태가 대기중에 있으며, 요청이 들어오면 그때 함수를 동작시킬 것이다.

- 이제 새로운 터미널 창을 열어서 다음 명령어를 입력해보자.
``` bash
curl "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'
```

이 명령어는 람다 함수를 실행하기 위한 명령어로 실제 AWS에서 로컬 실행시 쓰라고 제공해주는 명령어이다.

물론 docker run 명령어도 AWS에서 친절하게 알려준다.
<https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/images-test.html>

위 사이트를 참고하자.(뭐든지 공식 사이트가 가장 정확하고 확실한 법이라는것을 기억하자!!)

그럼 이제 어떤 에러가 나는지 보일 것이다.(안나오면 어캐 했는지 댓글로 부탁드립니다..)

``` json
{"errorMessage": "__init__() got an unexpected keyword argument 'executable_path'", "errorType": "TypeError", "requestId": "4c4b3cda-87df-42ef-babf-a084e93339f6", "stackTrace": ["  File \"/var/task/main.py\", line 16, in handler\n    browser = webdriver.Chrome(executable_path=\"/opt/chromedriver\", options=chrome_options)\n"]}%
```
이런 json response가 돌아오게 될 것이고, 제대로 실행되지 않았다는 것을 알 수 있다.

물론 도커를 실행했던 터미널에서도 에러 메시지를 야무지게 뱉어내고 있는것을 볼 수 있다.

## **이제 해결해보자**
자 이제 무슨 에러가 발생했는지도 알겠으니, 해결을 한번 해보자!

결론부터 말하자면 이 에러는 Chromedriver에 대한 구글 정책이 변경되어서 발생한 일이라고 한다.

[참고 사이트](https://www.inflearn.com/questions/816587/driver-webdriver-chrome-x27-chromedriver-x27-options-chrome-options)

뭐...그렇단다. 그래서 기존에 존재하던 코드에서도 변경이 필요하고, 우리가 클론해온 코드는 정책 변경이 이루어지기 전에 작성된게 아닌가 싶다.

그래서 코드를 변경하게 되면, 

``` python
    service = Service(executable_path=r'/opt/chromedriver')
    chrome_options = webdriver.ChromeOptions()
    chrome_options.binary_location = "/opt/chrome/chrome"
    chrome_options.add_argument("--headless")
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument("--single-process")
    chrome_options.add_argument("--disable-dev-shm-usage")
    chrome_options.add_argument("user-agent=Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko")
    chrome_options.add_argument('window-size=1392x1150')
    chrome_options.add_argument("disable-gpu")
    browser = webdriver.Chrome(service=service, options=chrome_options)
```

handler 함수 바로 밑부터, webdriver.Chrome 부분까지 다음과 같이 바꿔주면 된다. 코드 에러 부분을 봐도 webdriver.Chrome() 내부의 인자 중 executable_path가 알 수 없는 키라고 하니, 이 값을 바꿔주면 되는 것이다. ~~(이거 찾는데 2시간 걸린건 안비밀이다)~~

## **그래서 해결?**
이렇게 바꾸고... 다시 빌드하고... 실행하게 되면

``` bash
docker build --platform linux/x86_64 -t selenium-lambda:0.2 -f Dockerfile .
```
0.2로 태그 바꿔주고 실행했다.

``` bash
docker run --platform linux/amd64 -p 9000:8080 selenium-lambda:0.2
```
0.2 태그의 이미지를 실행하고 curl 명령어로 요청을 보내게 되면??

``` bash
curl "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'
```

``` json
{"statusCode": 200, "body": "{\"message\": \"\\ub124\\uc774\\ubc84 \\uba54\\uc778\\uc5d0\\uc11c \\ub2e4\\uc591\\ud55c \\uc815\\ubcf4\\uc640 \\uc720\\uc6a9\\ud55c \\ucee8\\ud150\\uce20\\ub97c \\ub9cc\\ub098 \\ubcf4\\uc138\\uc694\"}"}%
```

이렇게 200 statusCode와 함께 유니코드가 돌아오게 된다!! 이렇게 해결할 수 있게 되었고, 

![결과](/assets/img/posts/2024-01-17/결과적으로.png)
컨테이너 로그에서도 다음과 같은 결과를 얻을 수 있었다!

## **이제 해야 할 일**

이제 해야할 일은 실제로 람다에 올려서 제대로 동작하는지 확인을 하는 것이다.