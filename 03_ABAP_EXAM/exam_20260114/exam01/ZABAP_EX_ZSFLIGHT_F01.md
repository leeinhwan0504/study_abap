```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_EX_ZSFLIGHT_F01
*&---------------------------------------------------------------------*

*&---------------------------------------------------------------------*
*& Form SET_FUNCTION_KEY
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_function_key .
*SMPL
  G_FUNCTION_KEY-ICON_ID   = ICON_XLS.
  G_FUNCTION_KEY-ICON_TEXT = 'SAMPLE다운'.
  G_FUNCTION_KEY-TEXT      = 'SAMPLE다운'.
  SSCRFIELDS-FUNCTXT_01    = G_FUNCTION_KEY.

  CLEAR G_FUNCTION_KEY.
  G_FUNCTION_KEY-ICON_ID   = ICON_CREATE_NOTE.
  G_FUNCTION_KEY-ICON_TEXT = '노트생성'.
  G_FUNCTION_KEY-TEXT      = '노트를 생성합니다'.
  SSCRFIELDS-FUNCTXT_03    = G_FUNCTION_KEY.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form ACT_FUNCTION_KEY
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM act_function_key .
  CASE SSCRFIELDS-UCOMM.
    WHEN 'FC01'.
      PERFORM EXCEL_DOWN_SMPL.  " 여기 클릭해서 수정하러 가기. 수정 13~15
  ENDCASE.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form GET_FILE_PATH
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_file_path .
* 선택된 파일의 주소를 P_FILE 입력칸에 할당
* METHOD 사용
  DATA : LT_FILE TYPE FILETABLE,
         LS_FILE TYPE FILE_TABLE,
         LV_RC   TYPE I.

  CALL METHOD CL_GUI_FRONTEND_SERVICES=>FILE_OPEN_DIALOG
    CHANGING
      FILE_TABLE = LT_FILE
      RC         = LV_RC.

  READ TABLE LT_FILE INTO LS_FILE INDEX 1.
  IF SY-SUBRC = 0.
    P_FILE = LS_FILE.
  ENDIF.

* FUNCTION 사용시: CALL FUNCTION 'F4_FILENAME'
ENDFORM.
*&---------------------------------------------------------------------*
*& Form CHECK_BEFORE_PROCESS
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM check_before_process .
* 파일 주소 확인
  IF P_FILE EQ SPACE OR P_FILE = 'C:\'.
    MESSAGE '경로를 입력하세요' TYPE 'I'.
    LEAVE LIST-PROCESSING.
  ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form UPLOAD_FROM_EXCEL
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM upload_from_excel .
  DATA : lv_filename      TYPE string,
         lt_records       TYPE solix_tab,
         lv_headerxstring TYPE xstring,
         lv_filelength    TYPE i.

  lv_filename = p_file.



  CALL FUNCTION 'GUI_UPLOAD'
    EXPORTING
      filename                = lv_filename
      filetype                = 'BIN'
    IMPORTING
      filelength              = lv_filelength
      header                  = lv_headerxstring
    TABLES
      data_tab                = lt_records
    EXCEPTIONS
      file_open_error         = 1
      file_read_error         = 2
      no_batch                = 3
      gui_refuse_filetransfer = 4
      invalid_type            = 5
      no_authority            = 6
      unknown_error           = 7
      bad_data_format         = 8
      header_not_allowed      = 9
      separator_not_allowed   = 10
      header_too_long         = 11
      unknown_dp_error        = 12
      access_denied           = 13
      dp_out_of_memory        = 14
      disk_full               = 15
      dp_timeout              = 16
      OTHERS                  = 17.

  "convert binary data to xstring
  "if you are using cl_fdt_xl_spreadsheet in odata then skips this step
  "as excel file will already be in xstring
  CALL FUNCTION 'SCMS_BINARY_TO_XSTRING'
    EXPORTING
      input_length = lv_filelength
    IMPORTING
      buffer       = lv_headerxstring
    TABLES
      binary_tab   = lt_records
    EXCEPTIONS
      failed       = 1
      OTHERS       = 2.

  IF sy-subrc <> 0.
    "Implement suitable error handling here
  ENDIF.

  DATA : lo_excel_ref TYPE REF TO cl_fdt_xl_spreadsheet .

  TRY .
      lo_excel_ref = NEW cl_fdt_xl_spreadsheet(
                              document_name = lv_filename
                              xdocument     = lv_headerxstring ) .
    CATCH cx_fdt_excel_core.
      "Implement suitable error handling here
  ENDTRY .

  "Get List of Worksheets
  lo_excel_ref->if_fdt_doc_spreadsheet~get_worksheet_names(
    IMPORTING
      worksheet_names = DATA(lt_worksheets) ).

  IF NOT lt_worksheets IS INITIAL.
    READ TABLE lt_worksheets INTO DATA(lv_woksheetname) INDEX 1.

    DATA(lo_data_ref) = lo_excel_ref->if_fdt_doc_spreadsheet~get_itab_from_worksheet(
                                             lv_woksheetname ).
    "now you have excel work sheet data in dyanmic internal table
    ASSIGN lo_data_ref->* TO <gt_data>.
  ENDIF.          .

  DATA : lv_numberofcolumns   TYPE i,
         lv_date_string       TYPE string,
         lv_target_date_field TYPE datum.


  FIELD-SYMBOLS : <ls_data>  TYPE any,
                  <lv_field> TYPE any.

  FIELD-SYMBOLS : <ex_field> TYPE any.

  "you could find out number of columns dynamically from table <gt_data>


  " 수정16
  SELECT COUNT( DISTINCT FIELDNAME ) FROM DD03L WHERE TABNAME = 'ZSFLIGHT' INTO @lv_numberofcolumns.


*  LOOP AT <gt_data> ASSIGNING <ls_data> FROM 2 .
*
*
**    "processing columns
*    DO lv_numberofcolumns TIMES.
*      ASSIGN COMPONENT sy-index OF STRUCTURE <ls_data> TO <lv_field> .
*      ASSIGN COMPONENT sy-index OF STRUCTURE gs_EXCEL TO <ex_field> .
*      <ex_field> = <lv_field>.
*
*    ENDDO .
*    APPEND gs_EXCEL TO gt_EXCEL.
*    CLEAR gs_EXCEL.
**    NEW-LINE .
*  ENDLOOP .



" 여기는 완료하고 다시 수정했다.  여기는 다하고 수정할 것. 수정36
" ------------------------------------------------------------------------------------------
  TYPES : BEGIN OF t_dd03l,
          FIELDNAME TYPE DD03L-FIELDNAME,
          POSITION  TYPE DD03L-POSITION,
          INTTYPE   TYPE DD03L-INTTYPE,
          DECIMALS  TYPE DD03L-DECIMALS,
          END OF t_dd03l.

  DATA : it_dd03l TYPE TABLE OF t_dd03l,
         wa_dd03l TYPE t_dd03l.

  SELECT FIELDNAME, POSITION, INTTYPE, DECIMALS FROM DD03L
    WHERE TABNAME = 'ZSFLIGHT'          " 수정
    INTO TABLE @it_dd03l.

  SORT it_dd03l BY POSITION.


  LOOP AT <gt_data> ASSIGNING <ls_data> FROM 2 .


*    "processing columns
    DO lv_numberofcolumns TIMES.
      ASSIGN COMPONENT sy-index OF STRUCTURE <ls_data> TO <lv_field> .
      ASSIGN COMPONENT sy-index OF STRUCTURE gs_EXCEL  TO <ex_field> .

      READ TABLE it_dd03l INDEX sy-index INTO wa_dd03l.

      CASE wa_dd03l-inttype.
        WHEN 'T'.
          REPLACE ALL OCCURRENCES OF ':' IN: <lv_field> WITH ''.
          DATA : ZTIME(6) TYPE N.
          ZTIME = <lv_field>.

          IF ZTIME+2(2) = 60.
            DATA : ZHH TYPE I.

            ZHH = ZTIME+0(2).
            ZHH = ZHH + 1.
            ZTIME+0(2) = ZHH.
            ZTIME+2(2) = '00'.
          ENDIF.
            <ex_field> = ZTIME.

        WHEN 'D'.                                                   " 여기 추가 수정
           REPLACE ALL OCCURRENCES OF '-' IN: <lv_field> WITH ''.
           REPLACE ALL OCCURRENCES OF '.' IN: <lv_field> WITH ''.
           REPLACE ALL OCCURRENCES OF '/' IN: <lv_field> WITH ''.
           <ex_field> = <lv_field>.

        WHEN OTHERS.
            <ex_field> = <lv_field>.

*        WHEN 'P'.
*          <ex_field> = <lv_field>.
*          DATA P_DISTANCE(30) TYPE C.
*          P_DISTANCE = <ex_field>.
*
*          DATA v_off  TYPE I.
*          DATA lv_cnt TYPE I.
*          DATA lv_int TYPE I.
*          DATA lv_string TYPE I.
*
*          v_off = wa_dd03l-DECIMALS.
*          lv_cnt  = strlen( P_DISTANCE ).
*          lv_int = lv_cnt - v_off.
*          move P_DISTANCE+lv_int(v_off)  to lv_string.
*
*          IF lv_string <> 0.    " 0000이 아닐 때. TOP에 가서 DATA : DECI TYPE C. 이것을 넣을 것.
*            DECI = 'X'.
*          ENDIF.
      ENDCASE.

      " 여기도 추가 수정
      IF sy-index = 10.
*            IF gs_EXCEL-CURRENCY ='JPY' OR gs_EXCEL-CURRENCY ='KRW' OR gs_EXCEL-CURRENCY ='VND'.
*              gs_EXCEL-PRICE = gs_EXCEL-PRICE / 100.
*              gs_EXCEL-PAYMENTSUM = gs_EXCEL-PAYMENTSUM / 100.
*            ENDIF.

        DATA : WAERS TYPE TCURC-WAERS.
        DATA : BAPICURR TYPE BAPICURR-BAPICURR.

        WAERS = gs_EXCEL-CURRENCY.
        BAPICURR = gs_EXCEL-PRICE.

        CALL FUNCTION 'BAPI_CURRENCY_CONV_TO_INTERNAL'
          EXPORTING
            currency                   = WAERS
            amount_external            = BAPICURR
            max_number_of_digits       = 15
         	IMPORTING
            AMOUNT_INTERNAL            = gs_EXCEL-PRICE
*           RETURN                     =
                  .
        BAPICURR = gs_EXCEL-PAYMENTSUM.

        CALL FUNCTION 'BAPI_CURRENCY_CONV_TO_INTERNAL'
          EXPORTING
            currency                   = WAERS
            amount_external            = BAPICURR
            max_number_of_digits       = 15
          IMPORTING
            AMOUNT_INTERNAL            = gs_EXCEL-PAYMENTSUM
*           RETURN                     =
                  .
       ENDIF.

    ENDDO .

    APPEND gs_EXCEL TO gt_EXCEL.
    CLEAR gs_EXCEL.
*    NEW-LINE .
  ENDLOOP .

*  "you could find out number of columns dynamically from table <gt_data>
*  lv_numberofcolumns = 13 .
*
*  LOOP AT <gt_data> ASSIGNING <ls_data> FROM 2 .
*
*
**    "processing columns
*    DO lv_numberofcolumns TIMES.
*      ASSIGN COMPONENT sy-index OF STRUCTURE <ls_data> TO <lv_field> .
*      IF sy-subrc = 0 .
*        CASE sy-index .
*          when 1 .
*            gs_EXCEL-MANDT = <lv_field>.
*          when 2 .
*            gs_EXCEL-AGENCYNUM = <lv_field>.
*          when 3 .
*            gs_EXCEL-NAME = <lv_field>.
*          when 4 .
*            gs_EXCEL-STREET = <lv_field>.
*          when 5 .
*            gs_EXCEL-POSTBOX = <lv_field>.
*          when 6 .
*            gs_EXCEL-POSTCODE = <lv_field>.
*          when 7 .
*            gs_EXCEL-CITY = <lv_field>.
*          when 8 .
*            gs_EXCEL-COUNTRY = <lv_field>.
*          when 9 .
*            gs_EXCEL-REGION = <lv_field>.
*          when 10 .
*            gs_EXCEL-TELEPHONE = <lv_field>.
*          when 11 .
*            gs_EXCEL-URL = <lv_field>.
*          when 12 .
*            gs_EXCEL-LANGU = <lv_field>.
*          when 13 .
*            gs_EXCEL-CURRENCY = <lv_field>.
*
**          WHEN 10 .
**            lv_date_string = <lv_field> .
**            PERFORM date_convert USING lv_date_string CHANGING lv_target_date_field .
**            WRITE lv_target_date_field .
*          WHEN OTHERS.
**            WRITE : <lv_field> .
*        ENDCASE .
*
*      ENDIF.
*    ENDDO .
*    APPEND gs_EXCEL TO gt_EXCEL.
*    CLEAR gs_EXCEL.
**    NEW-LINE .
*  ENDLOOP .


** TAB으로 구분된 내용을 잘라서 ITAB에 APPEND 한다.
*  LOOP AT lt_intern.
*    SPLIT lt_intern
*    AT cl_abap_char_utilities=>horizontal_tab
*    INTO gs_EXCEL-CARRID gs_EXCEL-CARRNAME gs_EXCEL-CURRCODE gs_EXCEL-URL.
*    gs_EXCEL-MANDT = '001'.
*    APPEND gs_EXCEL TO gt_EXCEL.
*    CLEAR gs_EXCEL.
*  ENDLOOP.
*
**HEADER LINE 삭제
*  IF gt_excel IS NOT INITIAL.
*    DELETE gt_excel INDEX 1. "상단에 적힌 필드명을 삭제시켜줌
*  ELSE.
*    MESSAGE '데이터가 존재하지 않습니다' TYPE 'E'.
*  ENDIF.


ENDFORM.


*&---------------------------------------------------------------------*
*& Form GET_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_data .
* 필요 데이터 취합
  PERFORM GET_NEEDED_DATA.      " 여기 클릭해서 수정. 수정17 수정18
* 업로드 조건에 따라 ALV 출력 데이터 취합
  PERFORM GET_ZSC_DATA.         " 여기 클릭해서 수정. 수정19 수정20 수정21 수정22 수정23

ENDFORM.
*&---------------------------------------------------------------------*
*& Form EXCEL_DOWN_SMPL
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM excel_down_smpl .
* 다운로드 양식 선택
*    LS_KEY-OBJID = 'ZTEST14_EXCEL01'.
*    LS_KEY-RELID = 'MI'.
  DATA : FNAME(40) TYPE C.
  FNAME = '엑셀 업로드 샘플 양식'.

* 파일 경로 조회
  PERFORM SET_DIRECTORY.

* 엑셀 다운
  PERFORM DOWNLOAD_EXCEL_SMPL USING FNAME.    " 여기 클릭해서 수정하러 가기. 수정 13~15


ENDFORM.
*&---------------------------------------------------------------------*
*& Form SET_DIRECTORY
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> LS_KEY_OBJID
*&---------------------------------------------------------------------*
FORM set_directory.
*  CLEAR GV_FILE.
  IF OBJFILE IS INITIAL.
    CREATE OBJECT OBJFILE.
  ENDIF.
  CLEAR GV_INITIAL_DIR.
  CLEAR GV_DIRECTORY.

*  IF GV_FILE IS NOT INITIAL.
*    GV_INITIAL_DIR = GV_FILE.
*  ELSE.
*    OBJFILE->GET_TEMP_DIRECTORY( CHANGING     TEMP_DIR = GV_INITIAL_DIR
*                                 EXCEPTIONS   CNTL_ERROR           = 1
*                                              ERROR_NO_GUI         = 2
*                                              NOT_SUPPORTED_BY_GUI = 3 ).
*  ENDIF.
*  GV_INITIAL_DIR = 'C:\Temp'.

  OBJFILE->DIRECTORY_BROWSE( EXPORTING  INITIAL_FOLDER = GV_INITIAL_DIR
                             CHANGING   SELECTED_FOLDER = GV_DIRECTORY
                             EXCEPTIONS CNTL_ERROR      = 1
                                        ERROR_NO_GUI    = 2
                                        NOT_SUPPORTED_BY_GUI = 3 ).
  IF SY-SUBRC = 0.
*    GV_FILE = GV_DIRECTORY && '\' && LS_KEY-OBJID && '.xlsx'.
  ELSE.
    MESSAGE '파일경로에러' TYPE 'E'.
    LEAVE LIST-PROCESSING.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form DOWNLOAD_EXCEL_SMPL
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> LS_KEY_OBJID
*&---------------------------------------------------------------------*
FORM download_excel_smpl  USING    p_fname.
* OLE OBJECT 생성 & 실행
  CREATE OBJECT GO_APPLICATION 'Excel.Application'.

* 화면 DISPLAY 설정 (1을 설정하면 DISPLAY)
  SET PROPERTY OF GO_APPLICATION 'Visible' = 1.

* WORKBOOK 및 WORKBOOK 설정 & OPEN
  CALL METHOD OF GO_APPLICATION 'Workbooks' = GO_WBOOK.

*  SET PROPERTY OF GO_APPLICATION 'SheetsInNewWorkbook' = 2.

  CALL METHOD OF GO_WBOOK 'Add'.

* 최초 실행 SHEET는 첫번째
  CALL METHOD OF GO_APPLICATION 'Worksheets' = GO_SHEET
    EXPORTING
      #1 = 1.
  CALL METHOD OF GO_SHEET 'Activate'.
  SET PROPERTY OF GO_SHEET 'Name' = 'TABLE'.
  GET PROPERTY OF GO_APPLICATION 'ActiveWorkbook' = GO_WBOOK.

*  PERFORM column_format USING 1 '@'.
*  PERFORM column_format USING 2 '@'.
*  PERFORM column_format USING 3 '@'.
*  PERFORM column_format USING 4 '@'.
*  PERFORM column_format USING 5 '@'.
*  PERFORM column_format USING 6 '@'.
*  PERFORM column_format USING 7 '@'.
*  PERFORM column_format USING 8 '@'.
*  PERFORM column_format USING 9 '@'.
*  PERFORM column_format USING 10 '@'.
*  PERFORM column_format USING 11 '@'.
*  PERFORM column_format USING 12 '@'.
*  PERFORM column_format USING 13 '@'.

  TYPES : BEGIN OF T_DD03L,
    TABNAME   TYPE DD03L-TABNAME,
    FIELDNAME TYPE DD03L-FIELDNAME,
    POSITION  TYPE DD03L-POSITION,
  END OF T_DD03L.

  DATA : FTAB TYPE TABLE OF T_DD03L.
  DATA : S_FTAB TYPE T_DD03L.
  DATA : FLINE TYPE I.

  SELECT TABNAME FIELDNAME POSITION
    FROM DD03L
    INTO TABLE FTAB
    WHERE TABNAME = 'ZSFLIGHT'.   " 수정 13

  DESCRIBE TABLE FTAB LINES FLINE.

  DO FLINE TIMES.
    PERFORM column_format USING sy-index '@'.
  ENDDO.

  SORT FTAB BY POSITION.

*  DATA : POS TYPE I.
*  DATA : FNAM TYPE DD03L-FIELDNAME.


*  LOOP AT FTAB INTO S_FTAB.
*    POS = S_FTAB-POSITION.
*    FNAM = S_FTAB-FIELDNAME.
*    PERFORM FILL_CELL USING GO_APPLICATION 01 POS FNAM.
*  ENDLOOP.


** 데이터 입력
*  PERFORM FILL_CELL USING GO_APPLICATION 01: 01 'MANDT',
*                                             02 'AGENCYNUM',
*                                             03 'NAME',
*                                             04 'STREET',
*                                             05 'POSTBOX',
*                                             06 'POSTCODE',
*                                             07 'CITY',
*                                             08 'COUNTRY',
*                                             09 'REGION',
*                                             10 'TELEPHONE',
*                                             11 'URL',
*                                             12 'LANGU',
*                                             13 'CURRENCY'.


  DATA : TAB1 TYPE TABLE OF SFLIGHT WITH HEADER LINE.   " 수정14 SFLIGHT 로 할 것
  DATA : RNUM TYPE I.
  DATA : CNUM TYPE I.

  DATA: lv_deli(1) TYPE c value cl_bcs_convert=>gc_tab.
  DATA lv_rc      TYPE i.

  TYPES: lv_data(9999) TYPE c,
         ty            TYPE TABLE OF lv_data.
  DATA: lt_clip TYPE ty WITH HEADER LINE.
  FIELD-SYMBOLS : <tfield> TYPE ANY.

  DATA: CFIELD(30) TYPE C.    " 여기는 다 하고 수정할 것. 수정 34

  LOOP AT FTAB INTO S_FTAB.
    if sy-tabix = 1.
      lt_clip = S_FTAB-FIELDNAME.
    ELSE.
      CONCATENATE  lt_clip S_FTAB-FIELDNAME INTO lt_clip SEPARATED BY lv_deli.
    ENDIF.
  ENDLOOP.

  APPEND lt_clip.
  CLEAR lt_clip.

  SELECT * FROM SFLIGHT INTO TABLE TAB1.    " SFLIGHT 로 할 것. 수정15
*     UP TO 2 ROWS.


  " 여기는 완료하고 다시 수정했다.  여기는 다하고 수정할 것. 수정35
  LOOP AT TAB1.
    RNUM = sy-tabix + 1.

    DO FLINE TIMES.
      ASSIGN COMPONENT sy-index OF STRUCTURE TAB1 TO <tfield>.

      CFIELD = <tfield>.          " 이것이 추가 됨

      CONDENSE CFIELD NO-GAPS.    " 공백제거

      IF sy-index = 1.
        lt_clip = CFIELD.
      ELSE.
        CONCATENATE  lt_clip CFIELD INTO lt_clip SEPARATED BY lv_deli.
      ENDIF.
    ENDDO.

    APPEND lt_clip.
    CLEAR lt_clip.

  ENDLOOP.


  CALL METHOD cl_gui_frontend_services=>clipboard_export
    IMPORTING
      data                 = lt_clip[]
    CHANGING
      rc                   = lv_rc
    EXCEPTIONS
      cntl_error           = 1
      error_no_gui         = 2
      not_supported_by_gui = 3
      OTHERS               = 4.



   DATA:e_cell1 TYPE ole2_object,
        e_cell2 TYPE ole2_object,
        e_range TYPE ole2_object.

  CALL METHOD OF GO_APPLICATION 'Cells' = e_cell1
    EXPORTING
      #1 = 1
      #2 = 1.


  CALL METHOD OF GO_APPLICATION 'Cells' = e_cell2
    EXPORTING
      #1 = RNUM
      #2 = FLINE.


  CALL METHOD OF GO_APPLICATION 'Range' = e_range
    EXPORTING
      #1 = e_cell1
      #2 = e_cell2.

  CALL METHOD OF e_range 'Select'.

  CALL METHOD OF GO_SHEET 'Paste'.



*  LOOP AT TAB1 INTO TAB1.
*    RNUM = sy-tabix + 1.
*    PERFORM FILL_CELL USING GO_APPLICATION RNUM: 01 TAB1-MANDT,
*                                                 02 TAB1-AGENCYNUM,
*                                                 03 TAB1-NAME,
*                                                 04 TAB1-STREET,
*                                                 05 TAB1-POSTBOX,
*                                                 06 TAB1-POSTCODE,
*                                                 07 TAB1-CITY,
*                                                 08 TAB1-COUNTRY,
*                                                 09 TAB1-REGION,
*                                                 10 TAB1-TELEPHONE,
*                                                 11 TAB1-URL,
*                                                 12 TAB1-LANGU,
*                                                 13 TAB1-CURRENCY.
*  ENDLOOP.

DATA: lo_columns TYPE ole2_object.
CALL METHOD OF go_application 'Columns' = lo_columns.
CALL METHOD OF lo_columns 'Autofit'.
*  PERFORM column_width USING 1 10.
*  PERFORM column_width USING 2 10.
*  PERFORM column_width USING 3 20.
*  PERFORM column_width USING 4 10.
*  PERFORM column_width USING 5 40.

  DATA : BORDERS TYPE OLE2_OBJECT.
  DATA : RANGE TYPE OLE2_OBJECT.
  DATA : CELL1 TYPE OLE2_OBJECT.
  DATA : CELL2 TYPE OLE2_OBJECT.

  CALL METHOD OF GO_APPLICATION 'CELLS' = CELL1  " gs_cell2 에 2열 5행을 대입하고
    EXPORTING
    #1 = 1
    #2 = 1.

  CALL METHOD OF GO_APPLICATION 'Cells' = CELL2  " gs_cell2 에 3열 15행을 대입하고
    EXPORTING
    #1 = 1
    #2 = FLINE.

  CALL METHOD OF GO_APPLICATION 'Range' = RANGE
    EXPORTING
    #1 = CELL1
    #2 = CELL2.

  DATA: lo_interior TYPE ole2_object.

  CALL METHOD OF RANGE 'Interior' = lo_interior.
  SET PROPERTY OF lo_interior 'Color' = 65535.


  CALL METHOD OF GO_APPLICATION 'Cells' = CELL2  " gs_cell2 에 3열 15행을 대입하고
    EXPORTING
    #1 = RNUM
    #2 = FLINE.
  CALL METHOD OF GO_APPLICATION 'Range' = RANGE
    EXPORTING
    #1 = CELL1
    #2 = CELL2.

  CALL METHOD OF RANGE 'borders' = borders
    EXPORTING #1 = 7.
  SET PROPERTY OF borders 'Linestyle' = 1.

  CALL METHOD OF RANGE 'borders' = borders
    EXPORTING #1 = 8.
  SET PROPERTY OF borders 'Linestyle' = 1.

  CALL METHOD OF RANGE 'borders' = borders
    EXPORTING #1 = 9.
  SET PROPERTY OF borders 'Linestyle' = 1.

  CALL METHOD OF RANGE 'borders' = borders
    EXPORTING #1 = 10.
  SET PROPERTY OF borders 'Linestyle' = 1.

  CALL METHOD OF RANGE 'borders' = borders
    EXPORTING #1 = 11.
  SET PROPERTY OF borders 'Linestyle' = 1.

  CALL METHOD OF RANGE 'borders' = borders
    EXPORTING #1 = 12.
  SET PROPERTY OF borders 'Linestyle' = 1.

*CALL METHOD OF RANGE 'Merge'.

* Align:
  CONSTANTS:
    c_center TYPE i VALUE -4108,
    c_left   TYPE i VALUE -4131,
    c_right  TYPE i VALUE -4152.

  CALL METHOD OF RANGE 'select'.
  SET PROPERTY OF RANGE 'HorizontalAlignment' = c_center.

*  CALL METHOD OF GO_APPLICATION 'Worksheets' = GO_SHEET
*    EXPORTING
*      #1 = 2.
*  CALL METHOD OF GO_SHEET 'Activate'.
*  SET PROPERTY OF GO_SHEET 'Name' = 'ZSCARR2'.

* 파일명 설정
  CONCATENATE GV_DIRECTORY '\' p_fname '.xlsx' INTO GV_PATH.

* 실행 파일 저장
  CALL METHOD OF GO_WBOOK 'SaveAs' EXPORTING #1 = GV_PATH.


  IF SY-SUBRC = 0.
    MESSAGE '엑셀정상다운' TYPE 'S'.
  ELSE.
    MESSAGE '엑셀다운에러' TYPE 'S'.
  ENDIF.

*  CALL METHOD OF GO_WBOOK 'Close'.
*  CALL METHOD OF GO_APPLICATION 'Quit'.


ENDFORM.


*&---------------------------------------------------------------------*
*& Form FILL_CELL
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> GO_APPLICATION
*&      --> P_01
*&      --> P_01
*&      --> P_
*&---------------------------------------------------------------------*
FORM FILL_CELL  USING    PV_APPLICATION
                         PV_ROW
                         PV_COL
                         PV_VALUE.

  DATA: LV_ECELL TYPE OLE2_OBJECT.

  CALL METHOD OF PV_APPLICATION 'Cells' = LV_ECELL
    EXPORTING
      #1 = PV_ROW
      #2 = PV_COL.

  SET PROPERTY OF LV_ECELL 'Value' = PV_VALUE.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form GET_NEEDED_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_needed_data .
* 입력 데이터 점검을 위해 사용할 DB 데이터
  SELECT *
    FROM ZSFLIGHT
    INTO CORRESPONDING FIELDS OF TABLE GT_TABLE.
    SORT GT_TABLE BY CARRID CONNID FLDATE.        " 수정17 수정18

  IF r2 = 'X'.
    it_cp[] = GT_TABLE[].
  ENDIF.
*  IF GT_ZSCARR IS NOT INITIAL.
*    DELETE FROM ZSCARR.
*  ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form GET_ZSC_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM get_zsc_data .


*  LOOP AT GT_EXCEL INTO GS_EXCEL.
*    MOVE-CORRESPONDING GS_EXCEL TO GS_ZSC.
*
*    IF GS_ZSC-CARRID IS INITIAL OR GS_ZSC-CONNID IS INITIAL OR GS_ZSC-FLDATE IS INITIAL.  " 수정19
*      GS_ZSC-ZSTATUS = ICON_LED_RED.
*      GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[키값이 없습니다.]'.
*    ELSE.
*      SORT GT_TABLE BY CARRID CONNID FLDATE.                        " 수정20
*      READ TABLE GT_TABLE INTO GS_TABLE
*                           WITH KEY CARRID = GS_ZSC-CARRID          " 수정21
*                                    CONNID = GS_ZSC-CONNID          " 수정22
*                                    FLDATE = GS_ZSC-FLDATE          " 수정23
*                           BINARY SEARCH.
*      IF SY-SUBRC = 0.
*         GS_ZSC-ZSTATUS = ICON_LED_RED.
*         GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[키값이 이미 들어있습니다.]'.
*      ENDIF.
*    ENDIF.
*
*    IF GS_ZSC-ZRESULT IS INITIAL.
*      GS_ZSC-ZSTATUS = ICON_LED_YELLOW.
*    ENDIF.
*    APPEND GS_ZSC TO GT_ZSC.
*    CLEAR: GS_EXCEL, GS_ZSC.
*  ENDLOOP.


  LOOP AT GT_EXCEL INTO GS_EXCEL.                                     " 수업시간에 수정
    MOVE-CORRESPONDING GS_EXCEL TO GS_ZSC.

    IF GS_ZSC-CARRID IS INITIAL OR GS_ZSC-CONNID IS INITIAL OR GS_ZSC-FLDATE IS INITIAL.
      GS_ZSC-ZSTATUS = ICON_LED_RED.
      GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[키값이 없습니다.]'.
    ELSE.
      SORT GT_TABLE BY CARRID CONNID FLDATE.
      READ TABLE GT_TABLE INTO GS_TABLE
                           WITH KEY CARRID = GS_ZSC-CARRID
                                    CONNID = GS_ZSC-CONNID
                                    FLDATE = GS_ZSC-FLDATE
                           BINARY SEARCH.
      IF SY-SUBRC = 0.
         GS_ZSC-ZSTATUS = ICON_LED_RED.
         GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[키값이 이미 들어있습니다.]'.
      ENDIF.

    ENDIF.

    IF GS_ZSC-CARRID IS NOT INITIAL AND GS_ZSC-CONNID IS NOT INITIAL.
      SELECT SINGLE * FROM SPFLI INTO @DATA(S_SPFLI) WHERE CARRID = @GS_ZSC-CARRID AND CONNID = @GS_ZSC-CONNID.
      IF SY-SUBRC = 0.
        GS_ZSC-CITYFROM = S_SPFLI-CITYFROM.
        GS_ZSC-CITYTO = S_SPFLI-CITYTO.
      ELSE.
         GS_ZSC-ZSTATUS = ICON_LED_RED.
         GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[존재하지않는 비행기번호]'.
      ENDIF.

    ENDIF.
    IF GS_ZSC-CURRENCY IS NOT INITIAL.
      SELECT SINGLE * FROM SCURX INTO @DATA(S_SCURX) WHERE CURRKEY = @GS_ZSC-CURRENCY.
      IF SY-SUBRC = 0.
      ELSE.
         GS_ZSC-ZSTATUS = ICON_LED_RED.
         GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[존재하지않는 통화단위]'.
      ENDIF.
    ENDIF.
    IF GS_ZSC-PLANETYPE IS NOT INITIAL.
      SELECT SINGLE * FROM SAPLANE INTO @DATA(S_SAPLANE) WHERE PLANETYPE = @GS_ZSC-PLANETYPE.
      IF SY-SUBRC = 0.
      ELSE.
         GS_ZSC-ZSTATUS = ICON_LED_RED.
         GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[존재하지않는 비행기타입번호]'.
      ENDIF.
    ENDIF.


    IF GS_ZSC-ZRESULT IS INITIAL.
      GS_ZSC-ZSTATUS = ICON_LED_YELLOW.
    ENDIF.
    APPEND GS_ZSC TO GT_ZSC.
    CLEAR: GS_EXCEL, GS_ZSC.
  ENDLOOP.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form CREATE_OBJECT_INSTANCE
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM create_object_instance .
  CREATE OBJECT GO_DOCKING
    EXPORTING
      SIDE         = CL_GUI_DOCKING_CONTAINER=>DOCK_AT_LEFT
      EXTENSION    = 3000.

  CREATE OBJECT GO_GRID
    EXPORTING
      I_PARENT     = GO_DOCKING.

*CREATE OBJECT GO_CUSTOM
*  EXPORTING
*   CONTAINER_NAME = 'CON1'.
*
*CREATE OBJECT GO_GRID
*  EXPORTING
*    I_PARENT = GO_CUSTOM.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form SET_FILDCAT
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_fildcat .
  DEFINE _FCAT.
    CLEAR: GS_FCAT.
    GS_FCAT-FIELDNAME = &1.
    GS_FCAT-COLTEXT   = &2.
    GS_FCAT-KEY       = &3.
    GS_FCAT-EDIT       = &4.
    GS_FCAT-OUTPUTLEN  = &5.
    GS_FCAT-LOWERCASE  = &6.
    APPEND GS_FCAT TO GT_FCAT.
  END-OF-DEFINITION.

  CLEAR: GT_FCAT.

  TYPES : BEGIN OF T_DD03L,
    FIELDNAME TYPE DD03L-FIELDNAME,
    POSITION TYPE DD03L-POSITION,
    KEYFLAG TYPE DD03L-KEYFLAG,
    LENG TYPE DD03L-LENG,
  END OF T_DD03L.
  DATA : FTAB TYPE TABLE OF T_DD03L.
  DATA : S_FTAB TYPE T_DD03L.

  SELECT FIELDNAME POSITION KEYFLAG LENG
    FROM DD03L
    INTO TABLE FTAB
    WHERE TABNAME = 'ZSFLIGHT'.   " 수정26

  SORT FTAB BY POSITION.

  DATA : POS TYPE I.
  DATA : FNAM TYPE LVC_S_FCAT-FIELDNAME.
  DATA : CTXT TYPE LVC_S_FCAT-COLTEXT.
  DATA : KEY  TYPE LVC_S_FCAT-KEY.
  DATA : EDIT TYPE LVC_S_FCAT-EDIT.
  DATA : LENG TYPE LVC_S_FCAT-OUTPUTLEN.
  DATA : LOW  TYPE LVC_S_FCAT-LOWERCASE.

  IF r1 = 'X'.
    _FCAT: 'ZSTATUS' '상태' 'X' '' '3' ''.
     LOOP AT FTAB INTO S_FTAB.
        FNAM = S_FTAB-FIELDNAME.
        LENG = S_FTAB-LENG.
        KEY =  S_FTAB-KEYFLAG.
        CASE sy-tabix.                " 수정27
          WHEN 1.
            CTXT = '클라이언트'.
          WHEN 2.
            CTXT = '항공사 코드'.
          WHEN 3.
            CTXT = '항공편 노선'.
          WHEN 4.
            CTXT = '운항날짜'.
          WHEN 5.
            CTXT = '항공편 금액'.
          WHEN 6.
            CTXT = '통화단위 '.
          WHEN 7.
            CTXT = '항공기 기종'.
          WHEN 8.
            CTXT = '이코노미석의 최대 좌석 수'.
          WHEN 9.
            CTXT = '이코노미석의 점유된 좌석 수'.
          WHEN 10.
            CTXT = '현재 까지 총 예약 금액'.
          WHEN 11.
            CTXT = '비즈니스석의 최대 좌석 수'.
          WHEN 12.
            CTXT = '비즈니스석의 점유된 좌석 수'.
          WHEN 13.
            CTXT = '퍼스트클래스 석의 좌석 수'.
          WHEN 14.
            CTXT = '퍼스트클래스 석의 점유된 좌석 수'.
        ENDCASE.
        _FCAT: FNAM CTXT KEY '' LENG ''.
     ENDLOOP.

    _FCAT: 'CITYFROM' '출발도시' '' '' '20' ''.   " 수업시간에 수정
    _FCAT: 'CITYTO' '도착도시' '' '' '20' ''.     " 수업시간에 수정
    _FCAT: 'ZRESULT' '비고' '' '' '50' ''.
  ELSEIF r2 = 'X'.
     LOOP AT FTAB INTO S_FTAB.
       CLEAR : FNAM, LENG, KEY, EDIT, LOW.
        FNAM = S_FTAB-FIELDNAME.
        LENG = S_FTAB-LENG.
        KEY =  S_FTAB-KEYFLAG.
        IF KEY IS INITIAL.
          EDIT = 'X'.
        ENDIF.

*        CASE FNAM.
*          WHEN 'NAME' OR 'STREET' OR 'CITY' OR 'URL'.
*            LOW = 'X'.
*        ENDCASE.

        CASE sy-tabix.                " 수정28
          WHEN 1.
            CTXT = '클라이언트'.
          WHEN 2.
            CTXT = '항공사 코드'.
          WHEN 3.
            CTXT = '항공편 노선'.
          WHEN 4.
            CTXT = '운항날짜'.
          WHEN 5.
            CTXT = '항공편 금액'.
          WHEN 6.
            CTXT = '통화단위 '.
          WHEN 7.
            CTXT = '항공기 기종'.
          WHEN 8.
            CTXT = '이코노미석의 최대 좌석 수'.
          WHEN 9.
            CTXT = '이코노미석의 점유된 좌석 수'.
          WHEN 10.
            CTXT = '현재 까지 총 예약 금액'.
          WHEN 11.
            CTXT = '비즈니스석의 최대 좌석 수'.
          WHEN 12.
            CTXT = '비즈니스석의 점유된 좌석 수'.
          WHEN 13.
            CTXT = '퍼스트클래스 석의 좌석 수'.
          WHEN 14.
            CTXT = '퍼스트클래스 석의 점유된 좌석 수'.
        ENDCASE.
        _FCAT: FNAM CTXT KEY EDIT LENG LOW.
     ENDLOOP.
  ENDIF.


*IF r1 = 'X'.
*    _FCAT: 'ZSTATUS' '상태' 'X' '' '3' '',
*           'MANDT' '클라이언트' 'X' '' '3' '',
*           'AGENCYNUM' '여행사번호' 'X' '' '8' '',
*           'NAME' '이름' '' '' '25' '',
*           'STREET' '도로명' '' '' '30' '',
*           'POSTBOX' '우편박스' '' '' '10' '',
*           'POSTCODE' '우편코드' '' '' '10' '',
*           'CITY' '도시' '' '' '25' '',
*           'COUNTRY' '나라키' '' '' '3' '',
*           'REGION' '지역' '' '' '3' '',
*           'TELEPHONE' '전화번호' '' '' '30' '',
*           'URL' '사이트주소' '' '' '255' '',
*           'LANGU' '언어' '' '' '1' '',
*           'CURRENCY' '통화' '' '' '5' '',
*           'ZRESULT' '비고' '' '' '50' ''.
*ELSEIF r2 = 'X'.
*    _FCAT: 'MANDT' '클라이언트' 'X' '' '3' '',
*           'AGENCYNUM' '여행사번호' 'X' '' '8' '',
*           'NAME' '이름' '' 'X' '25' 'X',
*           'STREET' '도로명' '' 'X' '30' 'X',
*           'POSTBOX' '우편박스' '' 'X' '10' '',
*           'POSTCODE' '우편코드' '' 'X' '10' '',
*           'CITY' '도시' '' 'X' '25' 'X',
*           'COUNTRY' '나라키' '' 'X' '3' '',
*           'REGION' '지역' '' 'X' '3' '',
*           'TELEPHONE' '전화번호' '' 'X' '30' '',
*           'URL' '사이트주소' '' 'X' '255' 'X',
*           'LANGU' '언어' '' 'X' '1' '',
*           'CURRENCY' '통화' '' 'X' '5' ''.
*ENDIF.

  LOOP AT GT_FCAT INTO GS_FCAT WHERE FIELDNAME = 'CONNID'.    " 수정29
    GS_FCAT-LZERO   = 'X'.
    MODIFY GT_FCAT FROM GS_FCAT.
  ENDLOOP.


*  " 여기는 다하고 마지막에 하기. 수정39
  LOOP AT GT_FCAT INTO GS_FCAT WHERE FIELDNAME = 'PRICE'.
*    GS_FCAT-REF_TABLE = 'ZSFLIGHT'.
*    GS_FCAT-REF_FIELD = 'PRICE'.
      GS_FCAT-CFIELDNAME = 'CURRENCY'.
      " 통화단위를 참조할 필드이름. 소수점이 안나와야 할 것들의 소수점 조정. 원본데이터 SFLIGHT 필드를 보고한다. KRW 등과 같이.
      " 소수점을 없애는 대신에 .00이 붙게되는 경우도 있어서 이것도 수정해야한다.
      MODIFY GT_FCAT FROM GS_FCAT.
    ENDLOOP.

  LOOP AT GT_FCAT INTO GS_FCAT WHERE FIELDNAME = 'PAYMENTSUM'.
*  GS_FCAT-REF_TABLE = 'ZSFLIGHT'.
*  GS_FCAT-REF_FIELD = 'PAYMENTSUM'.
    GS_FCAT-CFIELDNAME = 'CURRENCY'.
    MODIFY GT_FCAT FROM GS_FCAT.
  ENDLOOP.

  LOOP AT gt_fcat INTO gs_fcat.
    CASE gs_fcat-fieldname.
      WHEN 'CURRENCY' OR 'PLANETYPE'.
        gs_fcat-emphasize = 'C500'.
        MODIFY gt_fcat FROM gs_fcat.
    ENDCASE.
  ENDLOOP.

ENDFORM.

*&---------------------------------------------------------------------*
*& Form SET_LAYOUT
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM set_layout .
  GS_LAYOUT-ZEBRA = 'X'.
  GS_LAYOUT-CWIDTH_OPT = 'A'.
  GS_LAYOUT-SEL_MODE   = 'D'.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form DISPLAY_ALV_0100
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM display_alv_0100 .

IF r1 ='X'.
 CALL METHOD GO_GRID->SET_TABLE_FOR_FIRST_DISPLAY
    EXPORTING
      IS_LAYOUT                     = GS_LAYOUT
    CHANGING
      IT_OUTTAB                     = GT_ZSC
      IT_FIELDCATALOG               = GT_FCAT
          .
 ELSEIF r2 ='X'.
*   !!! IMPORTANT !!!
*   We register the ENTER event so the manual changes
*   are propagated back to GT_DATA
* go_grid->register_edit_event( i_event_id = cl_gui_alv_grid=>mc_evt_enter ).

 CALL METHOD GO_GRID->SET_TABLE_FOR_FIRST_DISPLAY
    EXPORTING
      IS_LAYOUT                     = GS_LAYOUT
    CHANGING
      IT_OUTTAB                     = GT_TABLE
      IT_FIELDCATALOG               = GT_FCAT.

  CALL METHOD go_grid->set_ready_for_input
          EXPORTING
            i_ready_for_input = 0.
 ENDIF.
* 이거 안써주면 더블클릭 안먹힘!!!
CREATE OBJECT g_event_receiver.
SET HANDLER g_event_receiver->handle_double_click FOR GO_GRID.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form REFRESH_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM refresh_data .
CALL METHOD GO_GRID->REFRESH_TABLE_DISPLAY.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form SAVE_ZSC_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM save_zsc_data .
    LOOP AT GT_ZSC ASSIGNING FIELD-SYMBOL(<FS_ZSC>).
      PERFORM SAVE_ZSC_FINAL CHANGING <FS_ZSC>.         " 여기서 부터 클릭하면서 수정. " 수정30
    ENDLOOP.


*    LOOP AT GT_ZSC INTO GS_ZSC.
*     IF GS_ZSC-ZSTATUS = ICON_LED_YELLOW.
*     MOVE-CORRESPONDING GS_ZSC TO GS_ZSCARR.
*     GS_ZSCARR-MANDT = SY-MANDT.
*     INSERT INTO ZSCARR VALUES GS_ZSCARR.
*
*     IF SY-SUBRC = 0.
*         GS_ZSC-ZSTATUS = ICON_LED_GREEN.
*         GS_ZSC-ZRESULT = '저장 성공'.
*       COMMIT WORK AND WAIT.
*       MESSAGE S006.
*     ELSEIF SY-SUBRC <> 0.
*       MESSAGE E007.
*     ENDIF.
*  ENDIF.
*    ENDLOOP.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form SAVE_ZSC_FINAL
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      <-- <FS_ZSC>
*&---------------------------------------------------------------------*
FORM save_zsc_final  CHANGING GS_ZSC LIKE GS_ZSC.
IF GS_ZSC-ZSTATUS = ICON_LED_YELLOW.
     MOVE-CORRESPONDING GS_ZSC TO GS_TABLE.
     GS_TABLE-MANDT = SY-MANDT.
     INSERT INTO ZSFLIGHT VALUES GS_TABLE.      " 수정30

     IF SY-SUBRC = 0.
         GS_ZSC-ZSTATUS = ICON_LED_GREEN.
         GS_ZSC-ZRESULT = '저장 성공'.
       COMMIT WORK AND WAIT.
*       MESSAGE S006.
     ELSEIF SY-SUBRC <> 0.
         GS_ZSC-ZSTATUS = ICON_LED_RED.
         GS_ZSC-ZRESULT = '저장 실패'.
*       MESSAGE E007.
     ENDIF.
  ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form DEL_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM del_data .
* 입력 데이터 점검을 위해 사용할 DB 데이터
  SELECT *
    FROM ZSFLIGHT
    INTO CORRESPONDING FIELDS OF TABLE GT_TABLE.    " 수정24


  IF GT_TABLE IS NOT INITIAL.
    DELETE FROM ZSFLIGHT.       " 수정25

    IF SY-SUBRC = 0.
     MESSAGE '정상적으로 테이블 데이터가 전체 삭제되었습니다.' TYPE 'S'.
    ELSE.
      MESSAGE '데이터 삭제중에 문제가 생겼습니다.' TYPE 'E'.
    ENDIF.
  ELSE.
*     MESSAGE '데이터 삭제중에 문제가 생겼습니다.' TYPE 'E'.
    MESSAGE '삭제할 데이터가 없습니다.' TYPE 'I'.
  ENDIF.



ENDFORM.
*&---------------------------------------------------------------------*
*& Form SAVE_MODIFY
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM save_modify .
*
*        IF go_grid->is_ready_for_input( ) = 0.
*        CALL METHOD go_grid->set_ready_for_input
*          EXPORTING
*            i_ready_for_input = 1.
*
*      ELSE.
*        CALL METHOD go_grid->check_changed_data
*          IMPORTING
*            e_valid = l_valid.
*        IF l_valid = 'X'.
*          CALL METHOD go_grid->set_ready_for_input
*            EXPORTING
*              i_ready_for_input = 0.
*
*        ENDIF.
*      ENDIF.
*
  DATA: l_valid(1) TYPE c.
  CALL METHOD go_grid->check_changed_data
          IMPORTING
            e_valid = l_valid.

  IF l_valid = 'X'.
  DATA : p_confirm TYPE c.
  DATA : p_button TYPE c.
*      IF togl = 'X'.
       IF go_grid->is_ready_for_input( ) = 1.
        p_button = 'S'.
        IF it_cp[] NE gt_table[].
          PERFORM popup_confirm USING p_button CHANGING p_confirm.
          IF p_confirm = '1'.
            PERFORM f_save_data.      " 여기 클릭해서 수정. 수정31 32 33
            PERFORM GET_NEEDED_DATA.
            PERFORM refresh_data.
*            p_selfield-refresh = 'X'.
*            p_selfield-row_stable = 'X'.
*            p_selfield-col_stable = 'X'.
          ELSE.
            LEAVE SCREEN.
          ENDIF.
        ELSE.
          MESSAGE '변경할 데이터가 없습니다.' TYPE 'I'.
        ENDIF.
      ELSE.
        MESSAGE '조회모드입니다.' TYPE 'I'.
      ENDIF.

   ELSE.
      MESSAGE '변경할 수 없는 상태입니다.' TYPE 'I'.
   ENDIF.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form F_SAVE_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM f_save_data .
  DATA: wa_cp   TYPE ZSFLIGHT,    " 수정31
        wa_tmp  TYPE ZSFLIGHT.    " 수정32

  LOOP AT gt_table INTO gs_table.
    READ TABLE it_cp INTO wa_cp INDEX sy-tabix.
    IF wa_cp NE gs_table.

      MOVE-CORRESPONDING gs_table TO wa_tmp.

      MODIFY ZSFLIGHT FROM wa_tmp.  " 수정33
      IF sy-subrc = 0.
       APPEND gs_table TO it_changes.
      ELSE.
        MESSAGE '테이블 저장 중 에러가 발생했습니다.' TYPE 'I'.
        LEAVE SCREEN.
      ENDIF.
    ENDIF.

    CLEAR wa_cp.
  ENDLOOP.
*MESSAGE '데이터가 정상적으로 저장되었습니다.' TYPE 'S'.
  DESCRIBE TABLE it_changes LINES DATA(lines).
  MESSAGE lines && '건의 데이터가 저장되었습니다.' TYPE 'S'.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form POPUP_CONFIRM
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> P_BUTTON
*&      <-- P_CONFIRM
*&---------------------------------------------------------------------*
FORM popup_confirm USING p_button CHANGING p_confirm.     "POPUP 함수
  DATA: text_q  TYPE string,
        text_b1 TYPE string.
  IF p_button = 'S'.
    text_q = '변경된 데이터가 있습니다.저장하시겠습니까?'.
    text_b1 = '저장'.
  ELSEIF p_button = 'D'.
    text_q = '정말 삭제하시겠습니까?'.
    text_b1 = '삭제'.
  ENDIF.

  CALL FUNCTION 'POPUP_TO_CONFIRM'
    EXPORTING
      titlebar              = 'POPUP'
      text_question         = text_q
      text_button_1         = text_b1
      icon_button_1         = 'ICON_CHECKED'
      text_button_2         = '취소'
      icon_button_2         = 'ICON_INCOMPLETE'
      default_button        = '1'
      display_cancel_button = space
    IMPORTING
      answer                = p_confirm. "1:Continew / 2:Cancel
ENDFORM.
*&---------------------------------------------------------------------*
*& Form COLUMN_WIDTH
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> P_1
*&      --> P_10
*&---------------------------------------------------------------------*
FORM column_width  USING p_column TYPE i
                         p_width  TYPE i.

DATA:
lo_selection  TYPE ole2_object,
lo_column      TYPE ole2_object.

* Select the Column
CALL METHOD OF GO_SHEET 'Columns' = lo_column
EXPORTING
#1 = p_column.

CALL METHOD OF lo_column 'select'.
CALL METHOD OF go_application 'selection' = lo_selection.

SET PROPERTY OF lo_column 'ColumnWidth' = p_width.

ENDFORM.                    "Column_width
*&---------------------------------------------------------------------*
*& Form COLUMN_FORMAT
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> P_1
*&      --> P_
*&---------------------------------------------------------------------*
FORM column_format  USING p_column TYPE i
                          p_format.

DATA:
lo_selection  TYPE ole2_object,
lo_column      TYPE ole2_object.

* Select the Column
CALL METHOD OF GO_SHEET 'Columns' = lo_column
EXPORTING
#1 = p_column.

CALL METHOD OF lo_column 'select'.
CALL METHOD OF go_application 'selection' = lo_selection.

*SET PROPERTY OF lo_column 'ColumnWidth' = p_width.
SET PROPERTY OF lo_column 'NumberFormatLocal' = p_format.


ENDFORM.
```