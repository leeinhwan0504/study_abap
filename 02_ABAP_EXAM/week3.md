```abap
*&---------------------------------------------------------------------*
*& Report ZTEST09
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZTEST09.

TABLES:     sflight.  " 데이터베이스에 있는 sflight 테이블을 사용하겠다.

TYPE-POOLS: slis.     " slis 구조를 가져다 쓰겠다.

*Data Declaration
*--------------------
TYPES: BEGIN OF t_sflight,
  carrid      TYPE sflight-carrid,      " 항공사 코드 (CHAR 3 - AA, AZ)
  connid      TYPE sflight-connid,      " 항공편 노선 (NUMC 4 - 0018)
  fldate      TYPE sflight-fldate,      " 운항날짜   (DATS 8 - 2017.12.19)
  price       TYPE sflight-price,       " 항공편 금액 (CURR 15 - 185.00)
  currency    TYPE sflight-currency,    " 통화단위   (CUKY 5 - USD)
  seatsmax    TYPE sflight-seatsmax,    " 이코노미석의 최대 좌석 수  (INT4 10 - 385)
  END OF t_sflight.

* it_sflight : TABLE OF를 사용해서 STANDARD 테이블을 선언한다.
* wa_sflight : t_sflight 구조를 가지고 행을 선언한다 (구조를 복사해서 쓴다)
DATA: it_sflight  TYPE STANDARD TABLE OF t_sflight INITIAL SIZE 0,
      wa_sflight  TYPE t_sflight.


*ALV data declarations
DATA: fieldcatalog TYPE slis_t_fieldcat_alv WITH HEADER LINE,
      gd_tab_group TYPE slis_t_sp_group_alv,
      gd_layout    TYPE slis_layout_alv,
      gd_repid     LIKE sy-repid.


DATA : t TYPE slis_t_sp_group_alv.

*Start-of-selection.
START-OF-SELECTION.             " 실행버튼을 눌렀을 때 이벤트
  PERFORM data_retrieval.       " sflight 테이블에서 항공편 관련 데이터를 읽어와서 it_sflight 테이블에 저장
  PERFORM build_fieldcatalog.   " fieldcatalog 테이블에 ALV 리포트 컬럼을 정의
  PERFORM build_layout.         " ALV 리포트의 레이아웃을 정의
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
  fieldcatalog-key         = 'X'.
*  fieldcatalog-emphasize   = 'C100'.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR  fieldcatalog.

  fieldcatalog-fieldname   = 'CONNID'.
  fieldcatalog-seltext_m   = 'Flight Connection Number'.
  fieldcatalog-col_pos     = 1.
  fieldcatalog-lzero       = 'X'.
  fieldcatalog-emphasize   = 'C110'.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR fieldcatalog.

  fieldcatalog-fieldname   = 'FLDATE'.
  fieldcatalog-seltext_m   = 'Flight date'.
  fieldcatalog-col_pos     = 2.
  fieldcatalog-emphasize   = 'C110'.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR fieldcatalog.

  fieldcatalog-fieldname   = 'PRICE'.
  fieldcatalog-seltext_m   = 'Airfare'.
  fieldcatalog-col_pos     = 3.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR fieldcatalog.

  fieldcatalog-fieldname   = 'CURRENCY'.
  fieldcatalog-seltext_m   = 'Local currency of airline'.
  fieldcatalog-col_pos     = 4.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR fieldcatalog.


  fieldcatalog-fieldname   = 'SEATSMAX'.
  fieldcatalog-seltext_m   = 'Maximum capacity in economy class'.
  fieldcatalog-col_pos     = 5.
  APPEND fieldcatalog TO fieldcatalog.
  CLEAR fieldcatalog.

ENDFORM.                    " BUILD_FIELDCATALOG


*&---------------------------------------------------------------------*
*&      Form  BUILD_LAYOUT
*&---------------------------------------------------------------------*
*       Build layout for ALV grid report
*----------------------------------------------------------------------*
FORM build_layout.
  gd_layout-no_input  = 'X'.
  gd_layout-colwidth_optimize = 'X'.
  gd_layout-zebra = 'X'.
ENDFORM.


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
      t_outtab           = it_sflight
    EXCEPTIONS
      program_error      = 1
      OTHERS             = 2.
  IF sy-subrc <> 0.
* MESSAGE ID SY-MSGID TYPE SY-MSGTY NUMBER SY-MSGNO
*         WITH SY-MSGV1 SY-MSGV2 SY-MSGV3 SY-MSGV4.
  ENDIF.


ENDFORM.                    " DISPLAY_ALV_REPORT


*&---------------------------------------------------------------------*
*&      Form  DATA_RETRIEVAL
*&---------------------------------------------------------------------*
*       Retrieve data form EKPO table and populate itab it_ekko
*----------------------------------------------------------------------*
FORM data_retrieval.
  DATA: ld_color(1) TYPE c.   " 로컬 변수. PERFORM 구문 안에서만 쓸 수 있다.

  SELECT carrid connid fldate price currency seatsmax
*   UP TO 10 ROWS  "10줄 출력
    FROM sflight
    INTO TABLE it_sflight.

ENDFORM.                    " DATA_RETRIEVAL
``
