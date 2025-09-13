

```abap
*&---------------------------------------------------------------------*
*& Report ZWEEK7
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZWEEK7.


TABLES:     spfli.

TYPE-POOLS: slis.


* 데이터 정의
*-----------------------------------------------------------------------*
TYPES: BEGIN OF t_spfli,
  carrid    TYPE spfli-carrid,    " 항공사 코드(AA: American Airlines)
  connid    TYPE spfli-connid,    " 항공편 연결 번호
  deptime   TYPE spfli-deptime,   " 출발시간
  arrtime   TYPE spfli-arrtime,   " 도착시간
  distance  TYPE spfli-distance,  " 비행거리(출발지에서 도착지 까지 거리)
  distid    TYPE spfli-distid,    " 거리 단위(KM-킬로미터, MI-마일)
  fltype    TYPE spfli-fltype,    " 항공편 타입
  period    TYPE spfli-period,    " 도착 예정일 (출발일 기준 며칠 뒤)
END OF t_spfli.


DATA: it_spfli TYPE STANDARD TABLE OF t_spfli INITIAL SIZE 0.

*ALV data declarations
DATA: fieldcatalog TYPE slis_t_fieldcat_alv WITH HEADER LINE,
      gd_tab_group TYPE slis_t_sp_group_alv,
      gd_layout    TYPE slis_layout_alv,
      gd_repid     LIKE sy-repid.


SELECTION-SCREEN BEGIN OF BLOCK part1 WITH FRAME TITLE text-001.
  SELECT-OPTIONS s_carrid FOR spfli-carrid.
  SELECT-OPTIONS s_connid FOR spfli-connid.
  SELECT-OPTIONS s_period FOR spfli-period.
  SELECTION-SCREEN SKIP.
  PARAMETERS NUM TYPE I DEFAULT 10.
SELECTION-SCREEN END OF BLOCK part1.



INITIALIZATION.

AT SELECTION-SCREEN ON VALUE-REQUEST FOR s_period-low.
* s_cname-low = 'American Airlines'.

  TYPES : BEGIN OF abc,
    period type spfli-period,
  END of abc.

  DATA: IT_F4HELP3 TYPE TABLE OF abc.   " IT_F4HELP3으로 spfli-period 필드를 여러 개 넣을수 있는 테이블.

  SELECT PERIOD FROM SPFLI
  INTO TABLE IT_F4HELP3.                " IT_F4HELP3 테이블에 SPFLI 데이터 넣기

    DATA: IT_RETURN_TAB TYPE ddshretval OCCURS 0 WITH HEADER LINE .

    CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
    EXPORTING
      RETFIELD         = 'PERIOD'       " 도움말에서 2개의 필드가 있다면 더블클릭했을때 선택된 필드 값. 이 값이 검색화면에 넣어진다
      VALUE_ORG        = 'S'            " C Cell by Cell 하나를 선택해서 하나를 넣는 형태 S Structured 정해진 구조를 넣는 형태
      DYNPROFIELD      = 's_period-low' " RETFIELD 검색화면에 들어간 값이 이 변수에 들어간다.
      DYNPPROG         = SY-REPID
      DYNPNR           = SY-DYNNR
      CALLBACK_FORM    = 'CALL_BACK2'
      CALLBACK_PROGRAM = SY-REPID
    TABLES
      VALUE_TAB        = IT_F4HELP3       " 데이터 있어서 EXPORT
      RETURN_TAB       = IT_RETURN_TAB    " 데이터 없고 선언되어서 IMPORTING
    EXCEPTIONS
      PARAMETER_ERROR  = 1
      NO_VALUES_FOUND  = 2
      OTHERS           = 3.

*Start-of-selection. 실행했을 때
START-OF-SELECTION.

  PERFORM data_retrieval.
  PERFORM build_fieldcatalog.
  PERFORM build_layout.
  PERFORM display_alv_report.



AT SELECTION-SCREEN ON VALUE-REQUEST FOR s_period-high.
* s_cname-low = 'American Airlines'.

  TYPES : BEGIN OF abc,
    period type spfli-period,
  END of abc.

  DATA: IT_F4HELP3 TYPE TABLE OF abc.   " IT_F4HELP3으로 spfli-period 필드를 여러 개 넣을수 있는 테이블.

  SELECT PERIOD FROM SPFLI
  INTO TABLE IT_F4HELP3.                " IT_F4HELP3 테이블에 SPFLI 데이터 넣기

    DATA: IT_RETURN_TAB TYPE ddshretval OCCURS 0 WITH HEADER LINE .

    CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
    EXPORTING
      RETFIELD         = 'PERIOD'        " 도움말에서 2개의 필드가 있다면 더블클릭했을때 선택된 필드 값. 이 값이 검색화면에 넣어진다
      VALUE_ORG        = 'S'             " C Cell by Cell 하나를 선택해서 하나를 넣는 형태 S Structured 정해진 구조를 넣는 형태
      DYNPROFIELD      = 's_period-high' " RETFIELD 검색화면에 들어간 값이 이 변수에 들어간다.
      DYNPPROG         = SY-REPID
      DYNPNR           = SY-DYNNR
      CALLBACK_FORM    = 'CALL_BACK2'
      CALLBACK_PROGRAM = SY-REPID
    TABLES
      VALUE_TAB        = IT_F4HELP3       " 데이터 있어서 EXPORT
      RETURN_TAB       = IT_RETURN_TAB    " 데이터 없고 선언되어서 IMPORTING
    EXCEPTIONS
      PARAMETER_ERROR  = 1
      NO_VALUES_FOUND  = 2
      OTHERS           = 3.

*&---------------------------------------------------------------------*
*&      Form  call_back2
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->RECORD_TAB   text
*      -->SHLP_TOP     text
*      -->CALLCONTROL  text
*----------------------------------------------------------------------*
FORM CALL_BACK2 TABLES RECORD_TAB STRUCTURE SEAHLPRES
               CHANGING SHLP_TOP TYPE SHLP_DESCR
                     CALLCONTROL LIKE DDSHF4CTRL.

  SHLP_TOP-INTDESCR-DIALOGTYPE = 'A'.   "A 100개 이상이면 다이알로그 조회, D 즉시조회,  C 다이알로그 조회

ENDFORM.                    "call_back


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

  fieldcatalog-fieldname   = 'DEPTIME'.
  fieldcatalog-seltext_m   = 'Departure time'.
  fieldcatalog-col_pos     = 2.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'ARRTIME'.
  fieldcatalog-seltext_m   = 'Arrival time'.
  fieldcatalog-col_pos     = 3.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'DISTANCE'.
  fieldcatalog-seltext_m   = 'Distance'.
  fieldcatalog-col_pos     = 4.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'DISTID'.
  fieldcatalog-seltext_m   = 'Mass unit of distance (kms, miles)'.
  fieldcatalog-col_pos     = 5.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'FLTYPE'.
  fieldcatalog-seltext_m   = 'Flight type'.
  fieldcatalog-col_pos     = 6.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'PERIOD'.
  fieldcatalog-seltext_m   = 'Arrival n day(s) later'.
  fieldcatalog-col_pos     = 7.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

ENDFORM.


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

  SELECT carrid connid deptime arrtime distance distid fltype period
    UP TO NUM ROWS
    FROM spfli
    INTO TABLE it_spfli
    WHERE carrid IN s_carrid
      AND connid IN s_connid
      AND period IN s_period.

ENDFORM.
```
