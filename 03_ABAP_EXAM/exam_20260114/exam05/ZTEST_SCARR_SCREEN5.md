```abap
*&---------------------------------------------------------------------*
*& Report ZTEST_SCARR_SCREEN5
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZTEST_SCARR_SCREEN5.

INCLUDE ZABAP_SCARR_SCREEN5_TOP. " 데이터 선언
INCLUDE ZABAP_SCARR_SCREEN5_SEL. " 라디오버튼
INCLUDE ZABAP_SCARR_SCREEN5_C01. " 클래스(더블클릭 이벤트)
INCLUDE ZABAP_SCARR_SCREEN5_F01. " 실제 기능(Subroutine)
INCLUDE ZABAP_SCARR_SCREEN5_O01. " PBO (화면 출력 전)
INCLUDE ZABAP_SCARR_SCREEN5_I01. " PAI (화면 입력 후)



*&=====================================================================*
*& AT SELECTION-SCREEN
*&=====================================================================*


" 화면의 특정 필드 옆에 있는 'F4 도움말' 아이콘을 클릭하거나, 필드에 커서를 두고 F4 키를 눌렀을 때 발생하는 이벤트.
" 이 이벤트가 P_FILE이라는 이름의 입력 필드에 대해서만 발생하도록 대상을 지정.
AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE.
  PERFORM GET_FILE_PATH.  " 파일 탐색기

*&=====================================================================*
*& START-OF-SELECTION
*&=====================================================================*
START-OF-SELECTION.

  PERFORM CHECK_BEFORE_PROCESS. " P_FILE 에 들어간 경로를 체크하는 함수 호출.

  PERFORM UPLOAD_FROM_EXCEL.    " 엑셀에 들어간 데이터를 여기서 선언한 테이블로 옮겨주는 작업.
  PERFORM GET_DATA.             " 엑셀 업로드할거를 오류 검증    =>> GT_ZSC를 출력하기위해서 만들고 있다.
*
*&=====================================================================*
*& END-OF-SELECTION
*&=====================================================================*
END-OF-SELECTION.




CALL SCREEN 100.       " 화면 100 호출
```