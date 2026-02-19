```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_20260219_ZSCARR_1_C01
*&---------------------------------------------------------------------*

CLASS lcl_event_receiver DEFINITION.
  PUBLIC SECTION.
    METHODS: handle_double_click
      FOR EVENT double_click OF cl_gui_alv_grid
      IMPORTING e_row e_column.
ENDCLASS.

CLASS lcl_event_receiver IMPLEMENTATION.
  METHOD handle_double_click.
    READ TABLE gt_zscarr INDEX e_row-index INTO gs_zscarr.
    IF sy-subrc = 0.
      MESSAGE i000(st) WITH gs_zscarr-carrname '를 선택함'.
    ENDIF.
  ENDMETHOD.
ENDCLASS.

DATA: g_event_receiver TYPE REF TO lcl_event_receiver.
```