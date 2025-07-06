---
title: 자체 호스팅 라이브 동기화
aliases: []
tags: [옵시디언/플러그인]
related:
dg-publish:
created: 2025-06-22 12:22:19
modified: 2025-07-05 11:42:33
---

> 원문: https://github.com/vrtmrz/obsidian-livesync/blob/main/README.md

# **자체 호스팅 라이브 동기화**

**자체 호스팅 라이브 동기화**는 모든 Obsidian 호환 플랫폼에서 사용 가능한 커뮤니티 개발 동기화 플러그인입니다. 강력한 서버 솔루션(CouchDB 및 MinIO, S3, R2 같은 객체 스토리지)을 활용해 신뢰할 수 있는 데이터 동기화를 보장합니다.

또한, 실험적으로 WebRTC 기반 **피어 투 피어 동기화**를 지원하여 서버 없이도 기기 간 직접 노트 동기화가 가능합니다.
![동기화 데모](https://user-images.githubusercontent.com/45774780/137355323-f57a8b09-abf2-4501-836c-8cb7d2ff24a3.gif)
> [!IMPORTANT]
> 본 플러그인은 공식 "옵시디언 동기화"와 **호환되지 않으며**, 해당 서비스와의 동기화는 불가능합니다.

## 주요 기능
- **최소 트래픽**으로 볼트 동기화
- **충돌 관리**: 단순 변경 사항 자동 병합
- **오픈소스 서버 지원**: 호환 솔루션 사용 가능
- **종단 간 암호화** 적용
- [커스터마이징 동기화(Beta)](#customization-sync) 또는 [숨김 파일 동기화](#hiddenfilesync)로 설정/스니펫/테마/플러그인 동기화
- **WebRTC 피어 투 피어 동기화**(실험적): `호스트` 불필요
  - **주의**: 실험 단계 기능으로 안정성 보장되지 않음
  - 최소 **1대의 기기가 항상 온라인** 상태여야 함
  - 대체 솔루션:
    - [livesync-serverpeer](https://github.com/vrtmrz/livesync-serverpeer): 서버 기반 가상 클라이언트
    - [webpeer](https://github.com/vrtmrz/livesync-commonlib/tree/main/apps/webpeer): 기기 간 데이터 중계기
    - 실시간 인스턴스: [fancy-syncing.vrtmrz.net/webpeer](https://fancy-syncing.vrtmrz.net/webpeer/)
  - 상세 설명: [영문](https://fancy-syncing.vrtmrz.net/blog/0034-p2p-sync-en.html) / [일본어](https://fancy-syncing.vrtmrz.net/blog/0034-p2p-sync)

보안상 **자체 호스팅 필수** 연구자/엔지니어나 **프라이버시 중시** 사용자에게 적합합니다.
> [!IMPORTANT]
> - 설치/업그레이드 전 **볼트 백업 필수**
> - iCloud, 공식 동기화 등 **다른 동기화 도구와 병행 사용 금지**
> - 백업 플러그인: [Differential ZIP Backup](https://github.com/vrtmrz/diffzip)

## 사용 방법
### ✔️ 3분 설정 - fly.io 기반 CouchDB
**초보자 추천**

[![LiveSync Setup onto Fly.io SpeedRun 2024 using Google Colab](https://img.youtube.com/vi/7sa_I1832Xc/0.jpg)](https://www.youtube.com/watch?v=7sa_I1832Xc)

1. [fly.io에 CouchDB 설치](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_flyio.md)
2. [빠른 설정](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/quick_setup.md)에서 플러그인 구성

### ✔️ 수동 설정
1. 서버 구성:
   - [fly.io에 CouchDB 설치](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_flyio.md)
   - [자체 CouchDB 설정](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_own_server.md)
2. [빠른 설정](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/quick_setup.md)에서 플러그인 구성

> [!TIP]
> **무료 대안**:
> - Fly.io 유료화 → IBM Cloudant 사용 ([설치 가이드](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_cloudant.md))
> - 서버 없는 **피어 투 피어 동기화**
> - 무료 Cloudflare R2(객체 스토리지)
> - **라즈베리 파이**에 CouchDB 구축 가능 (※ 서버 보안 주의)

## 상태바 정보
동기화 상태를 아이콘으로 표시:

- **활동 표시기**
  `📲` 네트워크 요청
- **상태**
  `⏹️` 중지됨 | `💤` 대기 중 | `⚡️` 동기화 중 | `⚠️` 오류 발생
- **통계**
  `↑` 업로드 청크/메타데이터 | `↓` 다운로드 청크/메타데이터
- **진행 표시기**
  `📥` 미처리 항목 | `📄` DB 작업 | `💾` 쓰기 처리 | `⏳` 읽기 처리 |
  `🛫` 대기 중 읽기 | `📬` 일괄 읽기 | `⚙️` 숨김 파일 처리 |
  `🧩` 청크 대기 | `🔌` 커스터마이징 항목

> **중요**: 파일/DB 손상 방지를 위해 모든 진행 표시기 **사라질 때까지** Obsidian 종료 지양. []
> ※ 특히 파일 삭제/이름 변경 시

## 문제 해결
문제 발생 시: [문제 해결 가이드](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/troubleshooting.md) 참조
