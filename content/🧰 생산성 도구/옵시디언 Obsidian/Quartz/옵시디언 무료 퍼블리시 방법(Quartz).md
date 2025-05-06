---
tistoryBlogName: anpigon
tistoryTitle: "옵시디언 무료 퍼블리시 방법: Cloudflare, Quartz, Flowershow Plugin"
tistoryVisibility: "3"
tistoryCategory: "1022554"
tistorySkipModal: true
tistoryPostId: "449"
tistoryPostUrl: https://anpigon.tistory.com/449
title: Quartz를 이용해 옵시디언 노트를 웹에 게시하기
tags:
  - 옵시디언/Quartz
aliases:
  - Quartz를 이용해 옵시디언 노트를 웹에 게시하기
  - 옵시디언 무료 퍼블리시 방법
created: 2024-01-28 09:46:06
updated: 2025-05-06 12:04:27
dg-publish: true
---
옵시디언 노트를 서버에 무료로 호스팅할 수 있는 방법은 다양한데, 여기에는 GitHub Pages, Vercel, Netlify, GitLab Pages 등이 포함됩니다. 이 글에서는 Cloudflare를 이용하여 배포하는 방법에 대해 설명하겠습니다. 다른 호스팅 방법에 대한 자세한 정보는 [여기](https://quartz.jzhao.xyz/hosting)에서 확인하실 수 있습니다.

## 준비하기
1. GitHub 계정이 필요합니다. GitHub 계정이 없다면 [여기](https://github.com/signup)에서 계정을 생성할 수 있습니다.
2. Cloudflare 계정 또한 필요합니다. Cloudflare 계정이 없다면 [여기](https://dash.cloudflare.com/sign-up) 에서 계정을 생성할 수 있습니다.

## GitHub에서 새로운 Quartz 저장소 생성하기

GitHub의 [Quartz](https://github.com/jackyzha0/quartz) 페이지에 방문합니다. 그리고 'Use this template' 버튼을 클릭한 다음, 'Create a new repository'를 선택합니다.

![](https://i.imgur.com/CAm8vQT.png)

이어서 'Repository Name'을 입력하고 'Create Repository' 버튼을 클릭하여 새로운 저장소를 생성합니다.

## Cloudflare에 GitHub 연결하기
[Cloudflare 대시보드](https://dash.cloudflare.com/) 에 접속해 'Workers 및 Pages로 응용 프로그램을 빌드' 링크를 클릭합니다.

![](https://i.imgur.com/APujAJm.png)


다음 화면에서 'Pages' 탭을 선택하고, 이전에 GitHub에서 생성한 저장소를 연결합니다.

![](https://i.imgur.com/QHFNQfT.png)

프로젝트 이름을 입력하고 (입력한 이름이 도메인명이 됩니다), 프러덕션 브랜치를 확인합니다.
![](https://i.imgur.com/ZoiV2Pm.png)

빌드 설정에서 '프레임워크 미리 설정'을 '없음'으로 설정하고, '빌드 명령'에는 `npx quartz build`를 입력합니다. 그리고 '빌드 출력 디렉토리'에는 `public`를 입력합니다.

![](https://i.imgur.com/Bp14LkG.png)

## 옵시디언 노트를 웹에 게시하기

[Flowershow 플러그인](https://obsidian.md/plugins?id=flowershow)을 사용하면 옵시디언 노트를 깃허브 저장소에 업로드 할 수 있습니다. 이를 위해 플러그인을 설치하고 `Flowershow: Publish Single Note` 명령을 실행합니다. 그러면 해당 노트가 깃허브 저장소에 업로드됩니다.

Flowershow 플러그인을 사용하여 노트를 업로드한 후에는 옵시디언에 추가된 리본 아이콘 🌱을 클릭하여, 업데이트된 노트를 다시 업로드하거나 삭제할 수 있습니다. 

만약 노트 파일 경로가 변경되었다면, 리본 아이콘을 통해 해당 노트를 삭제한 후, 변경된 경로에 있는 노트를 다시 업로드해야 합니다.

![|500](https://i.imgur.com/HXePqYB.png)


> [!info] 정보
> 웹사이트에 접속했을 때 ***"404 Either this page is private or doesn't exist."*** 라는 메시지가 표시된다면, 이는 `content` 폴더 내부에 `index.md` 파일이 없는 것이 원인일 수 있습니다. 이 문제를 해결하기 위해서는 `content` 폴더 안에 `index.md` 파일을 생성해 주세요.
> 만약 방법을 모르시겠다면, [저의 블로그 깃허브 저장소](https://github.com/anpigon/anpigon-quartz/blob/v4/content/index.md)를 참고해 주세요.
 

## 마치며

%%다음은 Quartz와 Flowershow 플러그인을 이용해 만든 제 사이트입니다: [anpigon.pages.dev](https://anpigon.pages.dev/)%%

이 사이트는 Obsidian 노트를 기반으로 만들어졌으며, Quartz 템플릿과 Flowershow 플러그인을 활용해 Cloudflare를 통해 온라인에 게시되었습니다. 여기에는 앞으로 제가 작성한 노트를 공유할 예정입니다.

## 연결 문서
- [[🧰 생산성 도구/옵시디언 Obsidian/Quartz/Quartz 블로그에 댓글 기능 추가하기]]
- [[Flowershow 플러그인으로 무료로 퍼블리시하는 방법]]

## Quartz 템플릿으로 만든 블로그
- [Drill Garden](https://drillgarden.netlify.app/) 
- [DQ's garden](https://dqjeon.github.io)
- [개발자 유니의 두 번째 뇌](https://shin-jae-yoon.github.io) 
- [🌱Littleroot](https://nottaro.github.io/littleroot)