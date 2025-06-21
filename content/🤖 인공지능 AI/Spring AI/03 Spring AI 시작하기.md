---
created: 2024-10-04 08:52:16
updated: 2024-10-04 08:58:53
dg-publish: true
title: Spring AI 시작하기
---

이 섹션에서는 Spring AI 사용을 시작하는 방법에 대한 시작점을 제공합니다.

필요에 따라 다음 섹션의 각 단계를 따라야 합니다.

> [!Info] 노트
> Spring AI는 Spring Boot 3.2.x 및 3.3.x를 지원합니다.

## 스프링 초기화

[start.spring.io](https://start.spring.io/)로 이동하여 새 애플리케이션에 사용할 AI 모델과 벡터 스토어를 선택하세요.

## 마일스톤 및 스냅샷 리포지토리 추가하기

종속성 스니펫을 직접 추가하려면 다음 섹션의 지침을 따르세요.

마일스톤 및 스냅샷 버전을 사용하려면 빌드 파일에 Spring 마일스톤 및/또는 스냅샷 리포지토리에 대한 참조를 추가해야 합니다.

Maven의 경우 필요에 따라 다음 리포지토리 정의를 추가합니다:

```xml
<repositories>
	<repository>
		<id>spring-milestones</id>
		<name>Spring Milestones</name>
		<url>https://repo.spring.io/milestone</url>
		<snapshots>
			<enabled>false</enabled>
		</snapshots>
	</repository>
	<repository>
		<id>spring-snapshots</id>
		<name>Spring Snapshots</name>
		<url>https://repo.spring.io/snapshot</url>
		<releases>
			<enabled>false</enabled>
		</releases>
	</repository>
</repositories>
```

Gradle의 경우 필요에 따라 다음 리포지토리 정의를 추가합니다:

```gradle
repositories {
  mavenCentral()
  maven { url 'https://repo.spring.io/milestone' }
  maven { url 'https://repo.spring.io/snapshot' }
}
```

## 종속성 관리

Spring AI BOM(Bill of Materials)은 Spring AI의 특정 릴리스에서 사용되는 모든 종속성의 권장 버전을 선언합니다. 애플리케이션의 빌드 스크립트에서 BOM을 사용하면 종속성 버전을 직접 지정하고 유지 관리할 필요가 없습니다. 대신 사용 중인 BOM 버전에 따라 사용되는 종속성 버전이 결정됩니다. 또한 재정의하지 않는 한 기본적으로 지원되고 테스트된 종속성 버전을 사용하도록 보장합니다.

Maven 사용자라면 `pom.xml` 파일에 다음을 추가하여 BOM을 사용할 수 있습니다.

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Gradle 사용자는 또한 Maven BOM을 사용하여 종속성 제약 조건을 선언하기 위한 Gradle(5.0 이상) 기본 지원을 활용하여 Spring AI BOM을 사용할 수 있습니다. 이는 Gradle 빌드 스크립트의 종속성 섹션에 '플랫폼' 종속성 핸들러 메서드를 추가하여 구현됩니다. 아래 스니펫에서 볼 수 있듯이, 그 다음에는 사용하려는 하나 이상의 spring-ai 모듈에 대한 스타터 종속성을 버전 없이 선언할 수 있습니다(예: spring-ai-openai).

```gradle
dependencies {
  implementation platform("org.springframework.ai:spring-ai-bom:1.0.0-SNAPSHOT")
  // Replace the following with the starter dependencies of specific modules you wish to use
  implementation 'org.springframework.ai:spring-ai-openai'
}
```

## 특정 컴포넌트에 대한 종속성 추가

문서의 다음 각 섹션은 프로젝트 빌드 시스템에 추가해야 하는 종속성을 보여줍니다.

- [채팅 모델](https://docs.spring.io/spring-ai/reference/api/chatmodel.html)
- [임베딩 모델](https://docs.spring.io/spring-ai/reference/api/embeddings.html)
- [이미지 생성 모델](https://docs.spring.io/spring-ai/reference/api/imageclient.html)
- [트랜스크립션 모델](https://docs.spring.io/spring-ai/reference/api/audio/transcriptions.html)
- [텍스트 음성 변환(TTS) 모델](https://docs.spring.io/spring-ai/reference/api/audio/speech.html)
- [벡터 데이터베이스](https://docs.spring.io/spring-ai/reference/api/vectordbs.html)

## 샘플 프로젝트

GitHub에서 이러한 프로젝트를 복제하여 시작할 수 있습니다.

### 항공편 예약 도우미

[github.com/tzolov/playground-flight-booking](https://github.com/tzolov/playground-flight-booking)

이용약관(검색 증강 생성, RAG)에 액세스하고, 작업(함수 호출)을 수행하기 위한 도구(Java 메서드)에 액세스하며, LLM을 사용하여 사용자와 상호 작용하는 AI 기반 시스템

### OpenAI

- [github.com/rd-1-2022/ai-openai-helloworld](https://github.com/rd-1-2022/ai-openai-helloworld)

### Azure OpenAI

- [github.com/rd-1-2022/ai-azure-openai-helloworld](https://github.com/rd-1-2022/ai-azure-openai-helloworld)
- [github.com/Azure-Samples/spring-ai-azure-workshop](https://github.com/Azure-Samples/spring-ai-azure-workshop)
