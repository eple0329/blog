---
title: "[Climeet] (Release Branch → Main Branch)에서 Squash and merge는 사용하기 어렵다"
date: 2024-03-05 11:00:00 +0900
categories: [Project, 고민]
tags: [project, climeet, github, merge, squash]     
# TAG names should always be lowercase
toc: true
---

## **상황 설명**

팀 프로젝트를 진행하면서 크게 3개의 브랜치를 사용했다.

- develop (개발용 브랜치 - Local 서버 테스트)
- release (main 반영 전 테스트 브랜치 - Staging 서버 테스트)
- main (운영용 브랜치 - Production 서버 운영)

이렇게 3개로 구성이 되어 있으며 개발중에는

1. 기능별 임시 브랜치에서 기능을 개발하고
2. 기능단위 테스트가 완료된 브랜치를 develop에 PR을 통해 Merge
3. Sprint 단위로 개발을 관리하고, 한 Sprint가 끝나면 develop 최신 commit에 release 브랜치 생성
4. release에서 테스트 결과 양호하다면 main에 Merge

의 순서로 진행했다.

각 브랜치에서 Merge 방식은 아래와 같이 진행을 하고자 했다.

- feature/기능 → develop : Squash and merge
- develop → release : 코드상 develop == release → release를 develop 위치에 다시 만듬
- release → main : Squash and merge

하지만 release → main 과정에서 문제가 발생했다.

처음에는 문제가 없었지만, 두번째 배포를 진행하려고 할 때 release에서의 모든 변경사항이 Conflict로 인식되었던 것이다.

![Conflict된 상황](/assets/img/posts/2024-03-05/Conflict.png)

v0.1.0 → v0.1.1 과정에서 생긴 Conflict이다.

![깃 커밋 내용.png](/assets/img/posts/2024-03-05/깃%20커밋%20내용.png)

위와같은 상황에서 문제가 발생했다. (develop 브랜치가 보이지만, release 브랜치도 동시에 존재하고 있다.)

## **발생 원인**

발생된 원인은 다음과 같다

- Squash and merge는 다른 브랜치에서 변경된 사항을 한 커밋으로 압축하고, 한 개의 **‘독립된’** 커밋을 머지하는 것이기 때문이다.
- release 브랜치에서 변경된 커밋**’들’**과 main 브랜치에서 변경된 커밋은 커밋 해시값이 다르다.
- 동일한 내용이 변경된 것이지만, 서로 해시값이 다르기 때문에 다른 커밋으로 인식이 된다.
- 커밋이 되는 시간 순서를 보면 release 이전 커밋 → main 커밋 → release 이후 의 순서로 진행되었다.
- 따라서 release 브랜치, main 브랜치에서 각각 다른 변경이 있었다고 보기 때문에 release 이후의 커밋을 병합하려고 할 때 Conflict가 발생하는 것이다.

[문제의 원인에 대해 잘 설명한 사이트](https://taptorestart.tistory.com/entry/Q-%EA%B9%83%ED%97%88%EB%B8%8Cgithub%EC%97%90%EC%84%9C-%EA%B9%83%ED%94%8C%EB%A1%9Cgit-flow%EC%99%80-%EC%8A%A4%EC%BF%BC%EC%8B%9C%EC%99%80-%EB%B3%91%ED%95%A9Squash-and-merge%EC%9D%84-%ED%95%A8%EA%BB%98-%EC%93%B0%EB%A9%B4-%EC%95%88-%EB%90%98%EB%8A%94-%EC%9D%B4%EC%9C%A0%EB%8A%94)

### Q. 그러면 feature/기능 → develop에서는 왜 Conflict가 발생하지 않나요?

그 이유는 위의 release → main에서의 발생 원인에서 release 이전 커밋 → main 커밋에서 차이가 있기 때문이다.

release 브랜치와 main 브랜치는 같은 변경사항에서 파생되지만, 같은 커밋으로 파생되는 것이 아니다. 하지만 feature/기능 → develop은 feature/기능 브랜치의 시작점이 develop 브랜치이기 때문에 처음 시작은 feature/기능 == develop 으로 완전히 동일하다.

따라서 시간 순서로 봤을 때 develop == feature/기능 → develop 처럼 되므로 Conflict가 발생하지 않기 때문에 Squash and merge를 사용해도 된다.

## **해결 방안**

이 문제를 해결하고 싶다면 Squash and merge를 main에서 쓰면 안된다고 생각이 든다. 같은 변경사항도 다르게 인식하기 때문에 어쩔 수 없이 계속 발생할 것이라고 본다.

따라서 release → main으로 배포를 진행할 때에는 Squash and merge가 아닌 일반 Merge를 사용하는게 제일 좋은 방법이라고 생각한다.

추가로 Rebase and merge를 추천하는 글도 있었는데, 그러면 main 브랜치의 커밋이 많아지지 않을까 하는 생각이 든다. 나중에 사용하게 되면 해보고 일반 merge가 좋은지 rebase가 좋은지 비교를 해봐야겠다.

[이 사이트에서 추천하더라](https://jangjjolkit.tistory.com/49)