---
title: "[에러] Spring Boot/Unsupported media type 415, 400 Error - Java, Postman"
date: 2024-01-13 15:00:00 +0900
categories: [Project, Error]
tags: [spring, postman, error, multipart, form-data, json, java]     
# TAG names should always be lowercase
toc: true
---
## **상황 설명**
Spring Boot에서 열심히 API를 작성을 하던 중 파일과 Dto를 한번에 받아야해서 RequestPart를 사용해서 MultipartFile과 RouteCreateRequestDto를 한번에 받고자 했다.

``` java
    @PostMapping("/")
    public ApiResponse<String> createRoute(
        @RequestPart(value = "image") MultipartFile routeImage,
        @RequestPart(value = "routeCreatePostReq") RouteCreateRequestDto routeCreateRequestDto
    ) {
        return ApiResponse.onSuccess("성공했습니다.");
    }
```
편의상 필요한 부분만 남기고 작성하면 위와 같은 코드가 된다.
Spring을 아는 사람이라면 위 코드에는 문제가 없다는 것을 알 수 있을 것이다. 
(결론적으로 코드에는 문제가 없다...)

## **에러 설명 1**
API를 다 만들고 룰루랄라 Postman으로 확인을 해야지~ 하면서 열심히 테스트 입력값을 작성했다.

![PostmanInput](/assets/img/posts/2024-01-13/Postman_Input.png)

Dto 형식에 맞춰서 Json을 완벽하게 작성했고, 이미지 파일도 야무진걸로 가져와서 넣었다는 것을 볼 수 있다.

하지만 이 POST를 보내게 되면?

![Error](/assets/img/posts/2024-01-13/칼같이_거부_415_error.png)

이렇게 매우 짧은 시간에 거부당하는 모습을 볼 수 있다...
``` json
{
    "type": "about:blank",
    "title": "Unsupported Media Type",
    "status": 415,
    "detail": "Content-Type 'application/octet-stream' is not supported.",
    "instance": "/"
}
```
> Content-Type을 지원하지 않는다는 무자비한 말과 함께 빠꾸당했다(ㅠㅠ)

## **해결 방법 1**
이 문제의 해결 방법은 단순하다. 바로 Content-Type을 지정해주면 되는 것!

Postman에서는 따로 Content-Type을 지정해주지 않아도 자동으로 적당한걸 넣어주는 것 같은데, 이게 지원되지 않는 형식이라 생기는 문제인 것 같았다.

그래서 아래 사진처럼 Content-Type도 지정해주었다.

![Content-Type을_지정해서_수정한_Postman_Input](/assets/img/posts/2024-01-13/수정된_Postman_Input.png)

JSON 형식이니까 application/json을 넣어주고, Image니까 image/jpeg를 넣어줬다.

~~"와 이제 이건 해치웠다"~~ 라는 말을 하면서 Send를 눌렀는데... (끝이 아니었다...)

## **에러 설명 2**
두둥 탁!

![Error](/assets/img/posts/2024-01-13/칼같이_거부_400_error.png)

또 칼같이 거부당했다. (진짜 얘 왜이럼?)

``` json
{
    "type": "about:blank",
    "title": "Bad Request",
    "status": 400,
    "detail": "Failed to read request",
    "instance": "/"
}
```
> 이렇게 떠먹여줘도 왜 먹질 못하니!

해치웠나의 저주인가... 생각하면서 이리저리 구글링을 해보고 저리저리 이곳저곳을 찾아봤는데... 나와 같은 에러는 도저히 안나오더라...

## **해결 방법 2**
이게 왜 이랬는지는 나도 잘 모르겠더라...

결론을 말하자면
``` json
{“sectorId”: 3, “name”: “testRoute”, “difficulty”: 3}
```
이게 문제의 json이고

``` json
{"sectorId": 3, "name": "testRoute", "difficulty": 3}
```
이게 해결한 json이다.

지금 이걸 작성하고 있는 md에서도 문제가 있다고 하는데... (얘 왜이러니..??)

![뭐가_문제지](/assets/img/posts/2024-01-13/뭐가_문제일까.png)

자세히 보면 큰따음표가 문제였다... 나는 이 큰따음표를 내가 직접 썼는데, 어디서 썼길래 저걸로 변환이 된거지..?

하다가 해결이 안되니까 답답해서 json을 코드 에디터에서 작성하고 붙여넣었더니 성공했다더라...

![이번에는_성공](/assets/img/posts/2024-01-13/이번에는_성공_Vv.png)

여러분은 이런 에러 마주치지 마세요...ㅠㅠ
