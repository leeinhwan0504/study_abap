```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_SCARR_SCREEN5_I01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_0100 INPUT.
  CASE ok_code.
    WHEN 'BACK' OR 'CANC'.
      LEAVE TO SCREEN 0.
    WHEN 'EXIT'.
      LEAVE PROGRAM.
  ENDCASE.

ENDMODULE.
*&---------------------------------------------------------------------*
*&      Module  SAVE_DATA  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE save_data INPUT.
  CASE ok_code.
    WHEN 'SAVE'.
      " 라디오 버튼 조건(r1, r2)에 상관없이 무조건 업로드/저장 로직 실행
      PERFORM save_zsc_data.
  ENDCASE.
ENDMODULE.
*&---------------------------------------------------------------------*
*&      Module  EDIT_DATA  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE edit_data INPUT.
  DATA: l_valid(1) TYPE c.

  CASE ok_code.
    WHEN 'EDIT'.
      IF go_grid->is_ready_for_input( ) = 0.
        CALL METHOD go_grid->set_ready_for_input
          EXPORTING
            i_ready_for_input = 1.

      ELSE.
        CALL METHOD go_grid->check_changed_data
          IMPORTING
            e_valid = l_valid.
        IF l_valid = 'X'.
          CALL METHOD go_grid->set_ready_for_input
            EXPORTING
              i_ready_for_input = 0.

        ENDIF.
      ENDIF.
  ENDCASE.

ENDMODULE.
```