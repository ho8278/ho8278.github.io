---
layout: post
title: 안드로이드로 AR 내비게이션 개발하기 (2)
tags: [ARCore, Android, T map, RecyclerView, AR내비게이션]
---
저번 포스트에서는 프로젝트의 전반적인 개요를 살펴보았습니다. 이번 포스트에서는 핵심 기능 중 하나인 T map을 사용한 장소 검색 및 경로 탐색 기능에 대해 포스팅 해보려고 합니다. 내용에 들어가기에 앞서 먼저 왜 다른 내비게이션 API 말고 T map을 사용했는지에 대해 알려드리겠습니다.

**프로젝트에 사용하는 T map을 세팅하는 방법은 이번 포스트에서 포함 하지 않았습니다. 안드로이드 프로젝트에 T map을 추가하고 싶으신 분은 [이곳]()을 참고 하시면 될것 같습니다.**

### 기존 내비게이션 API 분석
1. [카카오내비](https://developers.kakao.com/docs/android/kakaonavi-api)
카카오 내비게이션 API를 사용하지 못했던 가장 큰 이유는 내비게이션 API를 사용하여 경로를 받아오고 나서 실제 내비게이션 서비스를 사용할 때 API를 사용하고 있는 앱이 아닌 카카오내비 앱을 통해서 내비게이션 서비스를 실행했기 때문입니다. 또한 저희 프로젝트의 기획인 보행자용 내비게이션이 아닌 운전자용 내비게이션이었기 때문에 사용하지 못하였습니다.
2. [네이버 지도](https://developers.naver.com/forum/posts/13)
네이버는 위 링크에서도 알 수 있듯이 내비게이션 API를 제공하지 않기 때문에 사용하지 못하였습니다.
3. [T map](http://tmapapi.sktelecom.com/main.html#android/guide/androidGuide.sample1)
T map의 경우에는 카카오내비와 동일하게 장소 검색과 경로검색 기능을 제공하지만 T map은 운전자용 경로뿐만 아니라 보행자용 경로까지 제공하고 있기 때문에 T map을 사용하게 되었습니다.


### 장소 검색 기능
<p align="center">
  <img width="40%" src="/img/ar_navigation_test_1.gif"/>
</p>
장소 검색 기능은 다음의 두 화면으로 구성하였습니다.

1. 장소 입력 화면
2. 장소검색 결과 출력 화면

먼저 출발지 검색을 하는 코드입니다.
```java
startText.setOnEditorActionListener(new TextView.OnEditorActionListener() {
    @Override
    public boolean onEditorAction(TextView textView, int i, KeyEvent keyEvent) {
        if (i == EditorInfo.IME_ACTION_SEARCH) {
            Intent intent = new Intent(SearchActivity.this, SearchResultActivity.class);
            intent.putExtra("name", startText.getText().toString());
            check = true;
            startActivityForResult(intent, 1);
            return true;
        }
        return false;
    }
});
```

따로 검색 버튼을 만들지 않았기 때문에 SoftKeyboard의 ImeOptions을 이용하여 검색기능을 구현 하였습니다. startActivityForResult를 사용한 이유는 두 번째 화면에서 선택된 정보를 가져와 마커를 찍기 위해 사용했습니다.

다음은 Tmap SDK에서 제공하는 클래스인 TmapData클래스를 사용한 장소 검색 코드 입니다.
```java
tMapData.findAllPOI(search, new TMapData.FindAllPOIListenerCallback() {
    @Override
    public void onFindAllPOI(ArrayList poiItem) {
        result.clear();
        for (int i = 0; i < poiItem.size(); i++) {
            TMapPOIItem item = (TMapPOIItem) poiItem.get(i);
            result.add(new SearchResult(item.getPOIName(), item.getPOIAddress().replace("null", ""), item.getPOIPoint()));
        }
        handler.post(new Runnable() {
            @Override
            public void run() {
                searchResultAdapter.notifyDataSetChanged();
            }
        });
    }
});
```
TMapData 클래스의 findAllPOI함수를 사용하여 장소를 검색하고 콜백을 통해 넘어온 결과 값을 Recyclerview에 출력해 주었습니다. findAllPOI함수는 내부적으로 쓰레드를 생성하여 네트워크 통신을 하고 콜백을 부르기 때문에 Handler를 사용하여 UI쓰레드에 접근 후 Recyclerview를 갱신해주었습니다.

### 경로 탐색 기능
<p align="center">
  <img width="40%" src="/img/ar_navigation_test_2.gif"/>
</p>

경로 탐색 기능은 TMap SDK에서 제공하는 TMapPolyLine 클래스를 사용하여 경로를 표시 할 수 있었습니다. TMapData의 findPathData함수를 사용하면 TMapPolyLine 클래스를 반환시켜 주는데 이 함수 역시 내부적으로 네트워크 접속을 하기 때문에 메인 쓰레드에서 사용할 수 없습니다. 그래서 AsyncTask를 사용하여 네트워크 통신을 하고 Handler를 이용해 메인 쓰레드에 접근하게 하였습니다.
```java
 class tempTask extends AsyncTask<Void, Void, Void> {
    ...
    @Override
    protected void onPostExecute(Void aVoid) {
        super.onPostExecute(aVoid);
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.mapicon);
        Bitmap resize = Bitmap.createScaledBitmap(bitmap, (bitmap.getWidth() * 100) / bitmap.getHeight(), 100, true);
        //for(int i=0; i<savePoint.size(); i++){
        markeritem = new TMapMarkerItem();
        markeritem.setIcon(resize);
        markeritem.setPosition(0.5f, 1.0f);

        markeritem.setTMapPoint(new TMapPoint(startLat, startLon));
        String id = "markeritem_start";
        tMapView.addMarkerItem(id, markeritem);
        TMapMarkerItem markeritem2 = new TMapMarkerItem();
        markeritem2.setIcon(resize);
        markeritem2.setPosition(0.5f, 1.0f);
        markeritem2.setTMapPoint(new TMapPoint(endLat, endLon));
        String id2 = "markeritem_end";
        tMapView.addMarkerItem(id2, markeritem2);
    }
    @Override
    protected Void doInBackground(Void... voids) {
        try {
            ...
            TMapPolyLine tMapPolyLine = new TMapData().findPathData(new TMapPoint(startLat, startLon), new TMapPoint(endLat, endLon));
            tMapPolyLine.setLineColor(Color.RED);
            tMapPolyLine.setLineWidth(2);
            handler.post(new Runnable() {
                @Override
                public void run() {
                    tMapView.addTMapPolyLine("Line1", tMapPolyLine);
                    tMapView.setCenterPoint(endLon,endLat);
                    Calendar currentCal = Calendar.getInstance();
                    DateFormat dateFormat = new SimpleDateFormat("a hh시 mm분");
                    currentCal.add(Calendar.MINUTE, (int) totalTime);
                    tTime.setText(dateFormat.format(currentCal.getTime()));
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

이상으로 AR 내비게이션의 기능 중 하나인 장소 검색과 경로 탐색기능에 대해 알아보았습니다. 다음 포스트에서는 AR을 사용한 내이게이션 기능에 대해 알아보도록 하겠습니다.
감사합니다.😄