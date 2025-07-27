---
title: "[AWS] AWS Lambda에 Docker Image로 Selenium을 올리기 위한 여정 (2, 完)"
date: 2024-01-19 19:30:00 +0900
categories: [AWS, Lambda]
tags: [aws, lambda, docker, image, selenium, python]     
# TAG names should always be lowercase
toc: true
---
## 환경
Macbook M1 Pro

python 3.9

이전 포스팅에서 Selenium으로 로컬 환경에서 실행하고 결과값을 받아오는 것까지 진행했었고, 이제는 실제로 AWS Lambda에 올리기까지의 여정을 적어볼 것이다.

이전 포스팅은 [여기](https://nesquitto.github.io/posts/aws-lambda-docker-1)에서 확인할 수 있다.

## **Lambda에 Docker 이미지를 어떻게 올리지?**
이전 포스팅에서 Docker 이미지를 만들고 로컬에서 빌드하여 실행까지 했었는데, 이제 이 잘 돌아가는 이미지 파일을 AWS Lambda에서 실행할 차례이다.

AWS Lambda에 이미지를 올리기 위해서는 빌드된 이미지를 AWS ECR에 올려야 한다. 왜 올려야 하냐고?

[AWS ECR에 이미지를 올려야 하는 이유](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/images-create.html)를 AWS 공식 문서에서 잘 설명해주고 있다. 그냥 순서가 그렇다.

![ECR을 써야하는 이유](/assets/img/posts/2024-01-19/ECR을-써야하는-이유.png)


## **AWS ECR에 이미지를 올리려면?**
이제 이유도 알았으니 한번 올려보자.
![ECR Private Repository](/assets/img/posts/2024-01-19/ECR-Repo.png)
무작정 ECR 프라이빗 리포지토리로 들어왔다. 생성을 해보자.

![ECR 생성](/assets/img/posts/2024-01-19/ECR-생성.png)
대충 이렇게 생성하면 알아서 잘 된다.

![ECR 생성 완료](/assets/img/posts/2024-01-19/ECR-생성-완료.png)
만들어진 레포를 클릭해서 들어가보자.

![ECR 레포 내부](/assets/img/posts/2024-01-19/ECR-레포%20내부.png)
여기에 들어오면 텅텅 빈 내부가 우리를 반겨주고, 오른쪽 위에 푸시 명령 보기라는 버튼이 있다. AWS에서는 이렇게 우리가 무엇을 할건지가 확실하면 친절하게 다 알려준다.

![ECR 푸시 명령](/assets/img/posts/2024-01-19/ECR-푸시-명령.png)
진짜 엄청 친절하다. 저것만 따라하면 우리도 ECR 마스터가 될 수 있다!(Wa!)

하지만 우리가 이걸 업로드하기 위해서는 터미널에 **AWS Configure**이 등록되어있어야 할 것이다.(진짜로 필요한지는 잘 모르지만, 필자가 진행할 때에는 당연히 필요할 것 같아서 그냥 설정하고 진행했다. 하지만 당연히 aws 명령어를 통해서 ecr에 로그인하니까 계정 정보는 당연히 저장해놔야하는 것 아닐까? ㅎㅎ)

이제 저 명령어를 쓰기 전에 IAM을 세팅하러 가보자!

## **IAM을 만들어보자!**
![IAM 생성창](/assets/img/posts/2024-01-19/IAM%20생성.png)
이미 ECRAccess라는 이름을 만들어놓긴 했는데, 저걸 만드는 과정이다.

![IAM 생성창 1](/assets/img/posts/2024-01-19/IAM%20생성-1.png)
이름을 설정해주고

![IAM 생성창 2](/assets/img/posts/2024-01-19/IAM%20생성-2.png)
권한을 설정해준다. 여기 권한을 세팅해줘야 외부에서 ECR에 접근할 수 있다. 사실 저 4개 권한이 모두 필요하지도 않고, 심지어 저 중 2개는 생성될 때 에러나면서 추가되지도 않지만, 나머지 2개도 모두 필요한건지도 모르지만, 일단 권한을 얻는 것에 목적이 있기 때문에 그냥 진행했다! ~~(공부 단계니까~)~~

설정을 완료하면 액세스 키와 비밀 액세스 키 2개를 주는데, 비밀 액세스 키는 생성된 화면에서만 볼 수 있으니 꼭 확인하고, 어딘가에 복사해두자! (까먹으면 다시 만들어서 적용하면 되긴 하지만 귀찮으니까~)

## **AWS Configure을 적용해보자**
이제 생성된 IAM을 터미널에서 설정만 해주면 된다. (aws가 터미널에 설치되어있지 않다면 우선 설치해주자!)
![AWS Configure](/assets/img/posts/2024-01-19/AWS%20configure.png)
이렇게 설정해주면 된다.

``` bash
aws configure
```
명령어를 입력해주면 되는데, 이후에는
1. 공개 액세스 키
2. 비밀 액세스 키
3. 지역 (사용할 AWS Region)
4. output Format → json
순으로 입력하면 된다.

자 이제 모든 세팅이 끝났다.
진짜로 ECR에 업로드 한번 해보자!!

## **ECR에 Docker 이미지 업로드하기**

![ECR 푸시 명령](/assets/img/posts/2024-01-19/ECR-푸시-명령.png)
다시 가져온 친절한 명령어 세팅. 이것만 사용하면 나도 진짜 ECR에 업로드 할 수 있다! ~~(어디가서 자랑 가능?)~~

``` bash
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin {본인 ECR 번호}.dkr.ecr.ap-northeast-2.amazonaws.com

docker build -t {이미지 이름} .

docker tag {이미지 이름}:latest {본인 ECR 번호}.dkr.ecr.ap-northeast-2.amazonaws.com/{ECR 레포 이름}:latest

docker push {본인 ECR 번호}.dkr.ecr.ap-northeast-2.amazonaws.com/{ECR 레포 이름}:latest
```

위처럼 입력하면 된다. 우리는 이미 이전에 도커 이미지를 빌드를 완료했기 때문에 완료된 이미지를 바로 올리면 된다. (2번째 명령어는 안해도 된다는 뜻!)

![도커 올리기](/assets/img/posts/2024-01-19/도커%20올리기%201.png)
이렇게 도커에서 AWS에 로그인도 해주고

![도커 올리기 2](/assets/img/posts/2024-01-19/도커%20올리기%202.png)
만든 이미지랑 ECR 레포랑 연결도 해주고, Push를 해주게 되면! 이렇게 야무지게 이미지가 올라가는걸 볼 수 있다.

![올라간거 확인](/assets/img/posts/2024-01-19/올라간거%20확인.png)
이렇게 올라간게 완료! 되었다. (필자는 여러번의 수정을 거쳐서 계속 Push를 했기 때문에 태그를 1.1로 설정했고, 태그는 알아서 설정해주면 된다.)

## **이제 Lambda에 올리자!**
이제 람다에 올리면 모든 과정이 끝나게 된다!! 와~
![람다1](/assets/img/posts/2024-01-19/람다1.png)
람다로 넘어와서 함수를 생성해주자!!

![람다2](/assets/img/posts/2024-01-19/람다2.png)
람다에서 컨테이너 이미지를 선택하고, 함수 이름과 컨테이너 이미지를 찾아서 잘 선택해주자! 아키텍처는 x86_64를 그대로 쓰면 된다.

람다가 만들어졌으면, 이제 실행 환경을 좀 설정해줘야 한다. 함수에 들어가서 **구성**탭의 **일반 구성**을 찾아서 설정을 해주자.
![람다3](/assets/img/posts/2024-01-19/람다3.png)
위와 같이 적당히 바꿔주고, 나중에 실제 사용하는 양을 보고 조절해주면 된다.

이제 모든게 끝났다. 테스트 코드를 생성하고, 동작을 확인하면 Selenium이 동작하는걸 두 눈으로 볼 수 있다!(감격 ㅠㅜ)
![람다4](/assets/img/posts/2024-01-19/람다4.png)
잘 동작한다! (끝이다!!)

**아 여기서 Selenium에서 페이지를 불러오지 못하는 오류가 있었는데, 이 경우에는 그냥 시간이 조금 지나고 다시 테스트를 돌려보자**

필자도 페이지를 불러오지 못해서 에러를 출력하는 경우가 있었는데, Lambda에서는 VPC를 설정해주지 않아도, 기본적으로 인터넷을 탐색할 수 있기 때문에 되는게 맞다. 아무래도 생성된지 얼마 안된 함수에서는 세팅중이라 그런거같기도 하고...(사실 잘 모른다.) 뭐 그런건 AWS에서 일하는 사람들이나 알 수 있는 것 아닐까?

## **이제 Selenium을 잘 올렸으니 야무지게 사용해보자**
이 방법이 Zip으로 그냥 올리는 것보다는 훨씬 편한거같기도 하고(이미 Dockerfile이 준비되어있어서 그런거긴 하지만..) 관리하기에도 훨씬 낫다. 우리 모두 앞으로는 람다를 사용할 때 도커파일로 관리를 해보자!