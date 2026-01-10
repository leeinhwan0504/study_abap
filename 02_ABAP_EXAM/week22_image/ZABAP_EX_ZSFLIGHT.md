```abap
*&---------------------------------------------------------------------*
*& Report ZABAP_EX_ZSFLIGHT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZABAP_EX_ZSFLIGHT.


INCLUDE ZABAP_EX_ZSFLIGHT_TOP.    " 수정 1~7
INCLUDE ZABAP_EX_ZSFLIGHT_SEL.
INCLUDE ZABAP_EX_ZSFLIGHT_C01.    " 수정 8~12
INCLUDE ZABAP_EX_ZSFLIGHT_F01.
INCLUDE ZABAP_EX_ZSFLIGHT_I01.
INCLUDE ZABAP_EX_ZSFLIGHT_O01.


*&=====================================================================*
*& INITIALIZATION
*&=====================================================================*
INITIALIZATION.
  PERFORM SET_FUNCTION_KEY.
  SY-TITLE = '엑셀 업로드 프로그램'.


*&=====================================================================*
*& AT SELECTION-SCREEN
*&=====================================================================*
AT SELECTION-SCREEN.
  PERFORM ACT_FUNCTION_KEY.     " 여기서 부터 클릭하면서 수정. 수정 13~15

AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE.
  PERFORM GET_FILE_PATH.

*&=====================================================================*
*& START-OF-SELECTION
*&=====================================================================*
START-OF-SELECTION.

IF r1 = 'X'.
    PERFORM CHECK_BEFORE_PROCESS.
* 파일 업로드 진행
    PERFORM UPLOAD_FROM_EXCEL.  " 여기서 부터 클릭하면서 수정.
    PERFORM GET_DATA.           " 여기서 부터 클릭하면서 수정. 수정 17~18 19~23
ELSEIF r2 = 'X'.
    PERFORM GET_NEEDED_DATA.    " 여기서 부터 클릭하면서 수정.
ELSEIF r3 = 'X'.
    PERFORM DEL_DATA.           " 여기서 부터 클릭하면서 수정. 수정24~25
ENDIF.

*&=====================================================================*
*& END-OF-SELECTION
*&=====================================================================*
END-OF-SELECTION.
IF r1 ='X'.
  CALL SCREEN 100.              " 여기서 부터 클릭하면서 수정.
ELSEIF r2 = 'X'.
  IF GT_TABLE IS NOT INITIAL.
  CALL SCREEN 100.
  ELSE.
    MESSAGE '조회할 데이터가 없습니다.' TYPE 'I'.
  ENDIF.


ENDIF.
```
