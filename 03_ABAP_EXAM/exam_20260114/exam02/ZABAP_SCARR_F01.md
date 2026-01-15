```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_SCARR_F01
*&---------------------------------------------------------------------*

FORM get_data.
  SELECT * FROM scarr INTO TABLE gt_scarr.
ENDFORM.

FORM set_fildcat.
  DEFINE _fcat.
    CLEAR gs_fcat.
    gs_fcat-fieldname = &1.
    gs_fcat-coltext   = &2.
    gs_fcat-key       = &3.
    gs_fcat-edit      = &4.
    gs_fcat-outputlen = &5.
    APPEND gs_fcat TO gt_fcat.
  END-OF-DEFINITION.

  CLEAR gt_fcat.
  _fcat: 'CARRID'   'ID'   'X' '' '5',
         'CARRNAME' '이름' ''  'X' '20',
         'CURRCODE' '통화' ''  'X' '5',
         'URL'      'URL'  ''  'X' '40'.
ENDFORM.

FORM set_layout.
  gs_layout-zebra      = 'X'.
  gs_layout-cwidth_opt = 'A'.
  gs_layout-sel_mode   = 'D'.
ENDFORM.

FORM create_object_instance.
  CREATE OBJECT go_custom
    EXPORTING
      container_name = 'CON1'.

  CREATE OBJECT go_grid
      EXPORTING
        i_parent = go_custom.
ENDFORM.

FORM display_alv_0100.
  CALL METHOD go_grid->set_table_for_first_display
    EXPORTING is_layout = gs_layout
    CHANGING  it_outtab = gt_scarr
              it_fieldcatalog = gt_fcat.

  " 이벤트 연결
  CREATE OBJECT g_event_receiver.
  SET HANDLER g_event_receiver->handle_double_click FOR go_grid.
ENDFORM.

FORM refresh_data.
  go_grid->refresh_table_display( ).
ENDFORM.
``