```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_BOOK_SCREEN_O01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Module STATUS_0100 OUTPUT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
MODULE status_0100 OUTPUT.
  DATA: title TYPE string.
  REFRESH fcode.
  CLEAR wa_fcode.

  wa_fcode = 'EDIT'.
  APPEND wa_fcode TO fcode.

  wa_fcode = 'DEL'.
  APPEND wa_fcode TO fcode.

  SET PF-STATUS 'S100' EXCLUDING fcode.  " 버튼/둘바 만드는 부분
  TITLE = '데이터 업로드'.

  SET TITLEBAR '100' WITH TITLE.          " 타이틀 만들기.
ENDMODULE.
```