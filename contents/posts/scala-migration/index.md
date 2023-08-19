---
title: "Scala Migration to 2.13"
description: "Scala 2.11에서 2.13으로 마이그레이션 하는 과정"
date: 2023-07-15
update: 2023-07-15
tags:
  - scala
  - migration
---

## 왜 마이그레이션을 해야 하는가?

현재 프로젝트에서 사용하고 있는 Scala 버전은 2.11.8이다. 시간이 흐르면서 지원이 점점 끊기는 구버전이 되고 있는 상황이고, 성능 및 편의가 개선된 상위 버전을 사용하는 것이 더 이로운 일이다. 하지만 바로 최신 버전으로 가기에는 부담이 되기 때문에 마이너 버전을 하나씩 올리는 방식으로 진행하였다. 목표 버전은 2.13.x이다.

## Scala 2.12.x

```
1. Maven 라이브러리 호환성
2. Future 상속 시에 오버라이딩 구현
```

프로젝트에는 Scala 2.11.8이 적용되어 있었다. 1차적으로는 2.12.x로 마이그레이션 해야 했기 때문에 위의 두가지 목표를 가지고 진행했다. 2번의 경우에는 미리 다른 프로젝트에서 마이그레이션 작업을 진행하신 팀장님께서 공유해주신 사항을 참고하였다.

Scala 2.11.18로 설정하고 각 라이브러리 버전을 확인하여 맞는 버전으로 업그레이드 진행했다. 인텔리제이에서 빌드했을 때 별다른 오류 없이 성공했지만 Jenkins에서 빌드를 하니 실패했다. 원인은 Maven에서 Scala compile이 실패됐기 때문이었다.

해당 문제에 대해 확인해보니 scala-maven-plugin의 버전 업그레이드가 필요했다. 4.2.4로 업그레이드 한 후 정상적으로 compile이 완료됐다. 로컬에서 간단한 테스트 후에 개발서버에 배포했지만 또 에러 로그를 발견했다. 에러 내용은 NullPointerException 이었다.

해당 케이스 클래스의 타입은 Option이었기 때문에 NullPointerException이 발생하면 안됐지만, 디버깅 결과 Some(null) 형태로 들어가있는 데이터를 확인했다. 역직렬화가 되는 프로세스를 정확히 알지 못해 Hazelcast 호환성 문제일까 싶어서 버전을 변경한 후에 재확인을 했다. 하지만 결과는 그대로였다.

검색과 테스트를 반복하다보니 Scala 2.12에서 직렬화 된 데이터를 2.13에서 역직렬화 하는 경우 Option 타입의 값을 정확히 객체화 시키지 못하고 Some(null)이 되는 경우가 있었다. 2.13에서 직렬화 한 후에 2.12에서 역직렬화 하는 경우에도 같은 증상을 확인했다.

하위 호환성을 지원해야 올바르지만 해당 데이터는 캐시 데이터여서 리셋 후에 2.13 버전으로 직렬화 된 데이터를 쌓도록 했다. 개발 서버에 배포 후 정상 동작을 확인하고, Scala 버전을 최종적으로 2.12.15로 변경했다.

## Scala 2.13.x

곧바로 2.13으로 마이그레이션을 진행하기로 하였다. 2.12 마이그레이션과는 달리 코드를 변경해줘야 하는 부분이 많아서 작업량이 이전보다 많았다. Deprecated 예정인 API도 되도록 수정하는 방향으로 진행했다. Scala 버전은 다른 프로젝트에서 이미 적용된 2.13.8로 하였다.

주요 변경 사항을 아래와 같다.

```
1. scala.collection.JavaConvnerters -> scala.jdk.CollectionConverters
2. x.filterKeys(..).map(identity) -> x.view.filterKeys(..).toMap
3. x.mapValues(..).map(identity) -> x.view.mapValues(..).toMap
4. x.onSuccess -> x.foreach
5. x.onFailure -> x.failed.foreach
6. x.tryCompleteWith -> x.completeWith
7. x.toIterator -> x.iterator
8. StringBuilder.newBuilder -> new StringBuilder()
```

이 외에 import 시에 package 경로를 명확하게 정해줘야 한다.

Scala 2.13 버전부터 지원이 되지 않는 라이브러리가 있어서 교체가 필요했다.

```
1. scala-logging 추가
2. scalamock-core, scalamock-scalatest-support 삭제
3. scalamock 추가
4. postgresql-async -> quill-async-postgresql 교체
```

수정 작업을 완료한 후에 maven에서 scala.compile이 정상적으로 되지 않아 scala-maven-plugin 버전을 최신 버전으로 올렸다. 이 후에는 빌드 및 실행 모두 정상적으로 되었고 개발 환경에 배포하면서 마이그레이션 작업을 마무리 했다.

## 마무리

버전 호환성만 확인하면 될거라고 생각했으나 몇가지 걸림돌이 숨겨져 있었다. 최신 버전이 아니고 마이너한 영역이라 레퍼런스의 도움을 받기도 쉽지 않았다. 문제를 예상하고 테스트 코드로 확인하는 과정을 필히 거치는 것이 좋을 것이라 생각하게 됐다.

작업양이 많아서 모든 Deprecated를 걷어내지는 못했지만 추후 작업을 하면서 2.13 버전에 맞도록 계속 수정이 필요해보인다. 그리고 2.13에서 변화점이 많아서 유용한 API 등에 대해 학습이 필요하다.
