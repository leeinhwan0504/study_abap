```abap
*&---------------------------------------------------------------------*
*& Report ZABAP_EX01_ZSTRAVELAG
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZABAP_EX01_ZSTRAVELAG.


INCLUDE ZABAP_EX_ZSTRAVELAG01_TOP.
INCLUDE ZABAP_EX_ZSTRAVELAG01_SEL.
INCLUDE ZABAP_EX_ZSTRAVELAG01_C01.
INCLUDE ZABAP_EX_ZSTRAVELAG01_F01.
INCLUDE ZABAP_EX_ZSTRAVELAG01_I01.
INCLUDE ZABAP_EX_ZSTRAVELAG01_O01.

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
  PERFORM ACT_FUNCTION_KEY.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE.
  PERFORM GET_FILE_PATH.

*&=====================================================================*
*& START-OF-SELECTION
*&=====================================================================*
START-OF-SELECTION.

IF r1 = 'X'.
    PERFORM CHECK_BEFORE_PROCESS.
* 파일 업로드 진행
    PERFORM UPLOAD_FROM_EXCEL.
    PERFORM GET_DATA.
ELSEIF r2 = 'X'.
    PERFORM GET_NEEDED_DATA.
ELSEIF r3 = 'X'.
    PERFORM DEL_DATA.
ENDIF.

*&=====================================================================*
*& END-OF-SELECTION
*&=====================================================================*
END-OF-SELECTION.
IF r1 ='X'.
  CALL SCREEN 100.
ELSEIF r2 = 'X'.
  IF GT_TABLE IS NOT INITIAL.
  CALL SCREEN 100.
  ELSE.
    MESSAGE '조회할 데이터가 없습니다.' TYPE 'I'.
  ENDIF.


ENDIF.
```

