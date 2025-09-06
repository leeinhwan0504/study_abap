
```abap
*&---------------------------------------------------------------------*
*& Report ZWEEK5_02
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZWEEK5_02.


TABLES:     spfli.

TYPE-POOLS: slis.

* 데이터 정의
*-----------------------------------------------------------------------*
TYPES: BEGIN OF t_spfli,
  carrid    TYPE spfli-carrid,    " 항공사 코드(AA: American Airlines)
  connid    TYPE spfli-connid,    " 항공편 연결 번호
  countryfr TYPE spfli-countryfr, " 비행기가 출발하는 국가 코드
  cityfrom  TYPE spfli-cityfrom,  " 비행기가 출발하는 도시의 이름
  deptime   TYPE spfli-deptime,   " 출발시간
  arrtime   TYPE spfli-arrtime,   " 도착시간
  distance  TYPE spfli-distance,  " 비행거리(출발지에서 도착지 까지 거리)
  distid    TYPE spfli-distid,    " 거리 단위(KM-킬로미터, MI-마일)
END OF t_spfli.

DATA: it_spfli TYPE STANDARD TABLE OF t_spfli INITIAL SIZE 0.

*ALV data declarations
DATA: fieldcatalog TYPE slis_t_fieldcat_alv WITH HEADER LINE,
      gd_tab_group TYPE slis_t_sp_group_alv,
      gd_layout    TYPE slis_layout_alv,
      gd_repid     LIKE sy-repid.

*-----------------------------------------------------------------------*

SELECTION-SCREEN BEGIN OF BLOCK part1 WITH FRAME TITLE text-001.
  SELECT-OPTIONS s_carrid FOR spfli-carrid.
  SELECT-OPTIONS s_connid FOR spfli-connid.
  SELECT-OPTIONS s_cntyfr FOR spfli-countryfr.  " 이름이 8글자
  SELECT-OPTIONS s_cityfr FOR spfli-cityfrom.
selection-SCREEN END OF BLOCK part1.

*Start-of-selection. 실행했을 때
START-OF-SELECTION.

  PERFORM data_retrieval.
  PERFORM build_fieldcatalog.
  PERFORM build_layout.
  PERFORM display_alv_report.



*&---------------------------------------------------------------------*
*&      Form  BUILD_FIELDCATALOG
*&---------------------------------------------------------------------*
*       Build Fieldcatalog for ALV Report
*----------------------------------------------------------------------*
FORM build_fieldcatalog.

  fieldcatalog-fieldname   = 'CARRID'.
  fieldcatalog-seltext_m   = 'Airline Code'.
  fieldcatalog-col_pos     = 0.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'CONNID'.
  fieldcatalog-seltext_m   = 'Flight Connection Number'.
  fieldcatalog-col_pos     = 1.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'COUNTRYFR'.
  fieldcatalog-seltext_m   = 'Country Key'.
  fieldcatalog-col_pos     = 2.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'CITYFROM'.
  fieldcatalog-seltext_m   = 'Departure city'.
  fieldcatalog-col_pos     = 3.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.


  fieldcatalog-fieldname   = 'DEPTIME'.
  fieldcatalog-seltext_m   = 'Departure time'.
  fieldcatalog-col_pos     = 4.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'ARRTIME'.
  fieldcatalog-seltext_m   = 'Arrival time'.
  fieldcatalog-col_pos     = 5.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'DISTANCE'.
  fieldcatalog-seltext_m   = 'Distance'.
  fieldcatalog-col_pos     = 6.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'DISTID'.
  fieldcatalog-seltext_m   = 'Mass unit of distance (kms, miles)'.
  fieldcatalog-col_pos     = 7.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

ENDFORM.                    " BUILD_FIELDCATALOG


*&---------------------------------------------------------------------*
*&      Form  BUILD_LAYOUT
*&---------------------------------------------------------------------*
*       Build layout for ALV grid report
*----------------------------------------------------------------------*
FORM build_layout.

  gd_layout-no_input          = 'X'.
  gd_layout-colwidth_optimize = 'X'.
  gd_layout-zebra = 'X'.

ENDFORM.                    " BUILD_LAYOUT


*&---------------------------------------------------------------------*
*&      Form  DISPLAY_ALV_REPORT
*&---------------------------------------------------------------------*
*       Display report using ALV grid
*----------------------------------------------------------------------*
FORM display_alv_report.
  gd_repid = sy-repid.
  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
    EXPORTING
      i_callback_program = gd_repid
      is_layout          = gd_layout
      it_fieldcat        = fieldcatalog[]
      i_save             = 'X'
    TABLES
      t_outtab           = it_spfli
    EXCEPTIONS
      program_error      = 1
      OTHERS             = 2.
  IF sy-subrc <> 0.
* MESSAGE ID SY-MSGID TYPE SY-MSGTY NUMBER SY-MSGNO
*         WITH SY-MSGV1 SY-MSGV2 SY-MSGV3 SY-MSGV4.
  ENDIF.

ENDFORM.

*&---------------------------------------------------------------------*
*&      Form  DATA_RETRIEVAL
*&---------------------------------------------------------------------*
*       Retrieve data form SCARR table and populate itab it_scarr
*----------------------------------------------------------------------*
FORM data_retrieval.

  SELECT carrid connid countryfr cityfrom deptime arrtime distance distid
    FROM spfli
    INTO TABLE it_spfli
    WHERE carrid    IN s_carrid
    AND   connid    IN s_connid
    AND   countryfr IN s_cntyfr
    AND   cityfrom  IN s_cityfr.

ENDFORM.
```
![실행결과](image/image_6_1.PNG)
![실행결과](image/image_6_2.PNG)
