---
layout: post
title: 데이터 크롤링 흉내내기
subtitle: 모바일게임 사이트 정보 크롤링
cover-img: /assets/img/background/gray.png
thumbnail-img: /assets/img/thumb.png
tags: [python, data-crawl]
---

내가 하고있는 모바일 게임이 한글을 지원하지 않는 게임이고 정보를 얻을 수 있는 커뮤니티가 일어와 영어로 된 커뮤니티 밖에 존재하지 않아 그냥 내가 찾아보고 싶어서 일본 위키에 정리된 내용을 크롤링 흉내를 내보았다.

단순하게 내 흥미를 위한 작업이기에 일정량 무료로 사용이 가능한 fire-store를 사용했다. 아무래도 개인 컴퓨터고 무료 서버를 이용하다보니 느리긴 하다.

1. 파이썬을 설치한다. https://www.python.org/downloads/ 설치는 당연하게도 공식 홈페이지에서 지원한다.

2. IDE는 vsCode를 사용했다. 무료라서 pyCharm은 유료다.
3. vsCode python Extension을 설치한다.

4. 디렉토리 생성 후 해당 폴더에서 vsCode 터미널로 pip install requests beautifulsoup4 googletrans==4.0.0-rc1 firebase-admin 해당 명령어 실행

5. 파이어베이스 스토어 프로젝트 생성

6.  ![image](https://github.com/sytp93/sytp93.github.io/assets/94149889/52e098b4-5c73-4f37-a006-bd693098a42b) 표시한 설정 버튼 클릭

7.  프로젝트 설정 클릭

8.  ![image](https://github.com/sytp93/sytp93.github.io/assets/94149889/38ed5853-ac5b-409e-b98e-c3f119518463) 표시한 서비스 계정 클릭

9.  ![image](https://github.com/sytp93/sytp93.github.io/assets/94149889/b14c7955-a75c-47d0-8da3-910d962946c9) 표시한 새 비공개 키 생성을 하면 json파일이 다운로드 된다.

10.  해당 파일을 나는 프로젝트 root 디렉토리에 firebase-key 라는 디렉토리를 만들고 그 안에 위치 시켰다.

11.  코드 작성
```python
import requests
from bs4 import BeautifulSoup
from googletrans import Translator
import firebase_admin
from firebase_admin import credentials, firestore

# Firebase 초기화
cred = credentials.Certificate('firebase-key/{key-file.json}')
firebase_admin.initialize_app(cred)
db = firestore.client()

# 번역기 초기화
translator = Translator()

# 웹사이트 지정
url = 'https://xn--bck3aza1a2if6kra4ee0hf.gamewith.jp/article/show/20722'
response = requests.get(url)

soup = BeautifulSoup(response.text, 'html.parser')

game_data = soup.body.find_all(class_='_n')
data_list = []

for detail in game_data:
    title = detail.text
    # content = article.find('p').get_text()
    
    # 텍스트 번역
    translated_title = translator.translate(title, src='ja', dest='ko').text
    # translated_content = translator.translate(content, src='en', dest='ko').text

    data_list.append({
        'title': title,
        'translated_title': translated_title,
    })

    # Firestore에 데이터 저장
    doc_ref = db.collection('DATA_CRAWL').document(translated_title)
    doc_ref.set({
        'title': translated_title,
    })

print("크롤링, 번역 및 Firestore 저장 완료!")
```

나는 일본어로 된 사이트의 데이터를 긁어와서 한글로 번역을 한 상태로 집어넣고 싶어서 데이터를 가져온 후 번역을 거치는 과정을 추가하였다.

단순하게 한글로 된 사이트의 데이터를 가져올때는 번역기 부분은 지워주면 된다.

![image](https://github.com/sytp93/sytp93.github.io/assets/94149889/697a1255-8174-45b6-af37-62707efceb02) 이 이미지로 결과가 fire-store에 저장된 모습을 확인할수 있다. 

이번 작업은 간단하게 한 화면에 있는 리스트의 값들을 뽑아서 저장했는데, 다음 작업으로 이제 상세내용 해당 페이지에 리스트를 클릭하고 상세 화면이 발생하면 해당 상세화면에서 내용을 긁어오는 작업을 해볼 예정이다.
