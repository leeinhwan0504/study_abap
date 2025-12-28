```abap
" ALV를 출력하기 전 과정
PROCESS BEFORE OUTPUT.
 " 툴바/ 타이틀바 출력.
 MODULE STATUS_0100.
 " ALV 출력
 MODULE SET_ALV_0100.

" 입력 후 과정. 버튼 누른 후에 동작
PROCESS AFTER INPUT.
 MODULE USER_COMMAND_0100.  " 뒤로가기 나가기 취소 기본 버튼들
 MODULE SAVE_DATA.          " 저장하는 버튼
 MODULE EDIT_DATA.          " 편집 버튼
```