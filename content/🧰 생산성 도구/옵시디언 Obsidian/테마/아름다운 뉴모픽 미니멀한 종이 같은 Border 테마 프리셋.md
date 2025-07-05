---
related: 
created: 2024-02-08 03:39:34
tags:
  - 옵시디언/테마
title: 아름다운 뉴모픽 미니멀한 종이 같은 Border 테마 프리셋
dg-publish: true
aliases: 아름다운 뉴모픽 미니멀한 종이 같은 Border 테마 프리셋
modified: 2025-05-06 12:05:11
---

출처: https://forum.obsidian.md/t/my-setup-beautiful-neumorphic-minimal-paperlike-border-theme-preset/75948


![](https://i.imgur.com/V4j5bsk.jpg)

## 메모
 - Border 테마의 프리셋이다.
 - light mode에서만 사용 가능하다.
- **뉴모픽 디자인** + **종이 같은 느낌**의 디자인이다.
- "Style Settings" 플러그인 설정에서 아래 CSS 코드 스니펫을 Import 한다.
```json
{  
"Appearance-light@@card-layout-open-light": true,  
"Appearance-light@@accent-light": "#808080",  
"Appearance-light@@accent-color-override-light": true,  
"Appearance-light@@mod-left-split-background-select-light": "mod-left-split-background-customize-light",  
"Appearance-light@@background-mod-left-split-light": "#F5F5F5",  
"Appearance-light@@mod-right-split-background-select-light": "mod-right-split-background-customize-light",  
"Appearance-light@@background-mod-right-split-light": "#F5F5F5",  
"Appearance-light@@mod-root-split-background-select-light": "mod-root-split-background-transparent-light",  
"Appearance-light@@on-border-light": "#666666",  
"Appearance-light@@background-underlying-select-light": "background-underlying-Color-light", 
"Appearance-light@@background-underlying-light": "#F0F0F0",  
"Appearance-light@@card-shadow-light": "0px 2px 4px rgba(0, 0, 0, 0.1)",  
"Appearance-light@@shadow-activated-tab-header-light": "0px 2px 4px rgba(0, 0, 0, 0.1)"  
}
```

- 버튼과 하이라이트에는 보라색을 사용(Style Settings에서 Color의 Accent color 설정을 통해 변경)
- 가독성을 높이기 위해 텍스트 폰트은 'Readex Pro', 인터페이스 폰트은 'Atkinson HyperLegible'을, Code 폰트은 'MonoLisa'를 사용.
	- _지금 내 폰트 설정은 텍스트에 '마루부리', 인터페이스는 'Pretendard',  Code는 'D2Coding'이다._
