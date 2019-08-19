---
layout: post
title: 안드로이드로 AR 내비게이션 개발하기 (1)
tags: [ARCore, Android, Tmap, RecyclerView, AR내비게이션]
---

안드로이드에서 증강현실(AR)을 사용하여 보행자 내비게이션을 만들어 본 적이 있습니다. 오늘은 제가 만들었던 AR 내비게이션에 대한 전반적인 기획과 제약사항에 대해 알아보도록 하겠습니다.


### 프로젝트 기획 동기
* 세상엔 지도 보고 길 못찾는 사람이 존재한다는 사실을 깨달음
* 처음 가 본 길은 방향을 헷갈리기 쉬움
* 주변에 길치들이 너무 많았음

처음에 만들어 볼 프로젝트를 구상하는건 그렇게 오래 걸리지 않았습니다. 제 여자친구만 해도 지도 보고 길 찾는걸 어려워 했고 주변에도 길 찾기를 힘들어 하는 친구들이 많았기 때문입니다.
 
하지만 내비게이션만 만든다면 기존에 많은 내비게이션 서비스와 차별점이 없었습니다. 그래서 증강현실을 이용하여 현재 가야 할 방향을 알려준다면 훨씬 더 정확하고 편하게 목적지에 도착 할 수 있을것 같아서 증강현실을 선택하게 되었습니다. 

### 사용할 플랫폼
* 안드로이드

사실 안드로이드 말고 아이폰까지 서비스 해보고 싶었지만 Mac OS가 없어서 개발 할 수 없었습니다. React-native같은 크로스 플랫폼 개발도 고려해 보았지만 브릿지를 사용함으로써 성능하락이 예상되었기 때문에 윈도우로도 개발이 가능한 안드로이드를 선택하게 되었습니다.

### 제약사항
*  안드로이드 7.0 이상
*  사용 가능한 기기의 목록이 있음

안드로이드에서 AR을 사용할 수 있도록 구글에서 ARCore라는 SDK를 제공합니다. 하지만 ARCore를 사용하기 위해서는 제약사항이 존재 했습니다. 개발자 가이드에서는 
>ARCore require:
>* Android 7.0 or later(some models require newer versions as noted below)
>* A device that originally shipped with the Google Play Store
>* Internet access, in order to install or update Google Play Services for AR

라고 명시하고 있습니다. 자세한건 [구글 개발자 가이드](https://developers.google.com/ar/discover/supported-devices?source=post_page-----bb278c9302fb----------------------)에서 확인 할 수 있습니다.

