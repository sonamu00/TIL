# Local storage
- key(키):value(값)을 String 타입으로 저장하는 기능만 있다. 
- 최대 저장용량은 5~10MB로 제한하여 간단한 텍스트만 담을 수 있지만 만료기간 없이 저장이 가능하다.
- 자동 로그인에 사용된다.
## 경로

### 크롬
Chrome은 Local Storage 디렉토리 내의 별도 파일에 저장한다.

#### Linux 저장 위치
```~/.config/google-chrome/Default/Local Storage/```
#### Window 저장 위치
```%LocalAppData%\Google\Chrome\User Data\Default\Local Storage\```
### 파이어폭스
Firefox는 프로필 폴더의 webappsstore.sqlite파일에 localstorage를 저장한다.
#### Linux 저장 위치
```~/.mozilla/firefox/<profile folder>/webappsstore.sqlit```
#### Window 저장 위치
```
C:\Users\<Windows login/user name>\AppData\Roaming\Mozilla\Firefox\Profiles\<profile folder>\webappsstore.sqlite
```


# session storage
- 만료기간이 있어 브라우저를 종료하거나 새탭을 열 때 데이터가 초기화가 된다. (같은 탭에서 새로고침 시에는 데이터가 유지됨)
- 자동 임시 저장 용도로 쓰인다. 
- 입력폼 정보 저장, 비 로그인 장바구니, 글쓰기 도중 내용 복구하기에 사용된다.

# Reference
- https://cyworld.tistory.com/3128
- https://iancoding.tistory.com/171