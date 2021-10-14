## 이벤트
### event 객체 속성 값

|속성|내용|
|---|---|
|event.currentTarget|이벤트 발생한 bubbling범위 내 이벤트핸들러의 this와 <br/>같은 값을 가짐|
|event.data|이벤트 핸들러로부터 받아오는 값|
|event.deledateTarget|현재 호출된 이벤트핸들러에 연결된 객체 리턴|
|event.namespace|사용자지정 이벤트명 리턴|
|event.pageX|왼쪽기준 마우스위치 리턴|
|event.pageY|위쪽기준 마우스위치 리턴|
|event.relatedTarget|마우스가 요소에 들어오거나 나오는 요소리턴|
|event.result|이벤트핸들러가 반환한 값을 보관|
|event.target|이벤트가 발생한 요소 리턴|
|event.timestamp|이벤트가 발생한 시간, 밀리세컨초로 리턴|
|event.type|발생한 이벤트를 리턴|
|event.which|키/마우스를 눌렀을 때 그 키의 정수 값 리턴|


### event 객체 메소드
|메소드|내용|
|---|---|
|event.isDefaultPrevented()|preventDefault()가 호출되었는지 확인|
|event.isImmediatePropagationStopped()|stopImmediatePropadation()이 호출되었는지 확인|
|event.isPropagationStopped()|stopPropagation()호출됐는지 확인|
|event.preventDefault()|기본이벤트 발생 방지|
|event.isImmediatePropagation()|실행된 한 개 이벤트 제외하고 나머지는 동일한 이벤트 중단|
|event.stopPropagation()|bubbling을 막는 메소드|

### 이벤트 연결 메소드
- 요소 객체에 이벤트 발생 시 연결된 이벤트 핸들러 지정하는 메소드<br/>

|메소드|내용|
|---|---|
|$(‘s’).bind(‘이벤트명 [이벤트명]’,function( ){ })|지정 이벤트에 이벤트 핸들러 지정<br/>다수 이벤트 지정 시 띄어쓰기로 구분
|$(‘s’).bind(‘이벤트명 [이벤트명]’,{ 객체 },function(){})|이벤트처리 객체를 인자로 넣어서 처리<br/>event객체 data에 들어감|
|$(‘s’).on(‘이벤트명[,이벤트명],’function( ){ })|지정 이벤트에 이벤트 핸들러 지정|
|$(‘s’).on({ 객체 }) | 이벤트처리 객체를 인자로 넣어서 처리<br/>event객체 data에 들어감|
|$(‘s’).on(‘이벤트’, ’선택자’,function(){})|선택된 요소 중 새로 동적으로 생성되는<br>선택자에 이벤트를 적용 동적적용|

### one()
- 이벤트를 연결하여 한 번만 실행하는 메소드

|메소드|내용|
|---|---|
|$(‘s’).one(‘이벤트명 [이벤트명]’,‘핸들러’)|핸들러에 있는 기능을 한 번만 실행<br/>이벤트 명을 다수로 설정하면 각 이벤트 별 한 번씩만 실행|

### 연결 이벤트
- 사용 방법
```jsx
$(‘s’).method(function(event){});
```
|메소드|내용|
|---|---|
|blur|요소가 focus해제 시|
|focus|요소가 focus받을 때|
|focusin|요소, child가 focus받을 때|
|focusout|요소,child가 focus해제 시|
|resize|윈도우 크기 변경 시|
|mouseover|마우스가 요소에 있을 때|
|mouseout|마우스가 요소,child에서 나갈 때|
|mouseleave|마우스가 요소에서 나갈 때|
|select|텍스트가 선택 됐을 때(textarea,filed)|
|keydown|키를 눌렀을 때|
|scroll|스크롤을 움직일 때|
|click|클릭했을때|
|dbclick|더블 클릭 했을 때|
|mousedown|마우스 왼쪽 버튼 누를 때|
|mouseup|마우스 왼쪽 버튼 뗄 때|
|mousemove|마우스가 요소, child에서 움직일 때|
|mouseenter|마우스가 요소에 들어올 때|
|change|요소의 값이 변경 되었을 때|
|submit|form이 전송 되었을 때|
|keypress|키를 눌렀을 때(alt, ctrl, shift, esc인식X)|
