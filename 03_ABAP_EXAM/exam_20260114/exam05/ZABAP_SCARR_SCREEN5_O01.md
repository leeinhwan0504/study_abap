```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_SCARR_SCREEN5_O01
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
  TITLE = 'ZSCARR 업로드'.

  SET TITLEBAR '100' WITH TITLE.          " 타이틀 만들기.
ENDMODULE.
*&---------------------------------------------------------------------*
*& Module SET_ALV_0100 OUTPUT
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
MODULE set_alv_0100 OUTPUT.
  IF go_custom IS INITIAL.
    PERFORM create_object_instance.
    PERFORM set_fildcat.
    PERFORM set_layout.
    PERFORM display_alv_0100.
  ELSE.
    PERFORM refresh_data.
  ENDIF.
ENDMODULE.
```