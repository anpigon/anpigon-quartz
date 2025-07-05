---
title: "Quartz 블로그에 댓글 기능 추가하기: Giscus 사용 방법"
tags:
  - quartz
created: 2024-01-30 02:09:56
modified: 2024-09-26 12:07:57
dg-publish: true
---

Quartz 블로그에 댓글 기능을 추가하고 싶으신가요? [Giscus](https://giscus.app/ko)를 사용하면 Quartz 블로그에 쉽게 댓글 기능을 추가할 수 있습니다. Giscus는 GitHub Discussions를 기반으로 하는 무료 댓글 시스템으로, 별도의 DB 서버가 필요 없습니다.

## Giscus 설치 준비

[Giscus 웹사이트](https://giscus.app/ko)를 방문하여 필요한 설정을 입력합니다. 여기서는 GitHub 레포지토리와 댓글을 매핑할 방법 등을 설정할 수 있습니다.

![](https://i.imgur.com/MCWkIAB.png)

설정을 모두 완료하면, Giscus에서 다음과 같은 코드가 생성됩니다.

![](https://i.imgur.com/inJLEke.png)

## 댓글 기능 설치 및 코드 수정

이제, 여러분의 블로그가 설치된 GitHub 저장소에 가서 `quartz.layout.ts` 파일을 열고 다음 코드를 추가해야 합니다.  `sharedPageComponents`의 `afterBody` 필드를 편집하여 위에서 얻은 값으로 다음 옵션을 포함하도록 합니다:

`quartz.layout.ts`
```ts
afterBody: [
  Component.Comments({
    provider: 'giscus',
    options: {
      // data-repo
      repo: 'anpigon/anpigon-quartz',
      // data-repo-id
      repoId: 'R_kgDOLKhz-Q',
      // data-category
      category: 'Announcements',
      // data-category-id
      categoryId: 'DIC_kwDOLKhz-c4CczBa',
    }
  }),
],
```

이 글을 따라하면, 블로그에 손쉽게 댓글 기능을 추가할 수 있습니다. Giscus를 통해 댓글 기능을 추가하는 과정은 간단하며, 개발자가 아니어도 쉽게 따라 할 수 있습니다.
