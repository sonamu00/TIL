## jQuery

### jQuery란?

- 존 레식에 의해 개발된 경량 JavaScript 라이브러리
- 복잡한 JavaScript 코드를 손쉽게 구현하기 위해 개발됨

### jQuery의 장점

1. DOM과 관련된 처리를 쉽게 구현할 수 있음
2. ajax통신, 이벤트 처리 등을 폭넒게 지원함
3. 별도의 플러그인을 통해 차트, 슬라이드쇼, 테이블 등을 간단히 구현 가능

### jQuery 연결

- head 요소안에 script 태그를 이용해서 jQuery 파일을 포함 시켜줌
    1. JQuery 홈페이지에서 .js 파일을 다운로드 받아 연결하는 방법

        ```jsx
        <script src='../파일위치/'jquery-3.5.1.min.js'></script>
        ```

    2. CDN(Content Delivery Network)를 통한 연결

        ```jsx
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
        ```

### jQuery 시작

- (document).ready()
    - 페이지를 모두 로드 한 후 ready 메소드를 실행
    - 자바스크립트의 document.onload와 같은 개념의 메소드
- 사용법 1번

    ```jsx
    JQuery(document).ready(function(){ // .ready는 전체 코드 다 해석하고 실행
    	$('#p3').css("background-color","yellow");
    });
    ```

- 사용법 2번

    ```jsx
    $(document).ready(function(){
    	$('#p4').css("background-color", "orange");
    });
    ```

- 사용법 3번

    ```jsx
    $(function(){ // 코드 로딩 되자마자 실행
    	$('p').css("color","white");
    });
    ```
### JQuery 기본 선택자
- 전체 선택자
|선택자|기능|사용예시|
|---|---|---|
|```*```|HTML 페이지에 있는 모든 문서 객체를 선택|``` $('*').css("color","red"); ```|

- 아이디 선택자
|선택자|기능|사용예시|
|---|---|---|
|```#```|특정 아이디 속성을 가진 문서 객체를 선택|``` $('*').css("color","red"); ```|

- 클래스 선택자
|선택자|기능|사용예시|
|---|---|---|
|```.```|특정한 클래스 속성을 가진 문서 객체를 선택|``` $('.testItem').css("background-color","orange");```|

- 자식 선택자와 후손 선택자
|선택자|기능|사용예시|
|---|---|---|
|```>```|부모 선택자의 바로 아래 객체 선택|```$('div > h3').css("color","black");```|
|```띄어쓰기```|부모 선택자 아래 모든 객체|```$('div h3').css("background-color","yellow");```|

- 속성 선택자
|선택자|기능|사용예시|
|```요소[속성]```|특정 속성을 가진 객체 선택|``` $('input[class]').css("background-color","green");```|
|```요소[속성 = 값]```|속성 안의 값이 특정 값과 같은 객체 선택|```$('input[type=text]').val('change value');```|
|```요소[속성 ~= 값]```|속성 안의 값이 특정 값을 단어로써 포함하는 객체 선택|``` $('input[class ~= test]').val('1234');```|
|```요소[속성 $= 값]```|속성 안의 값이 특정 값으로 끝나는 객체 선택|```$('input[type $= box]').attr('ckecked',true);```|
|```요소[속성 *= 값]```|속성 안의 값이 특정 값을 포함하는 객체 선택|```$('input[type *= box]').css('width','100px')```|
|```요소[속성 ^= 값]```|속성 안에 값이 특정 단어로 시작하는 객체 선택|```$('input[type ^= ra]').attr('ckecked',true);```|

