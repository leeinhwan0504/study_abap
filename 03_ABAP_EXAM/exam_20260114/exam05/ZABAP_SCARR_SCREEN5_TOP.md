```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_SCARR_SCREEN5_TOP
*&---------------------------------------------------------------------*

TYPE-POOLS: OLE2. " TYPES:   OLE2_OBJECT LIKE   OBJ_RECORD.  =>> 이 구조를 매크로 돌리기 위해서 사용했다.



TABLES: sscrfields.
" 사용자가 화면에서 버튼을 클릭하거나, 메뉴를 선택하거나, 엔터를 누를 때 발생하는 기능 코드(Function Code)가
" SSCRFIELDS-UCOMM 필드에 자동으로 저장.
" 특정 이벤트 블록(예: AT SELECTION-SCREEN 또는 AT USER-COMMAND) 내에서 이 구조체의 값을 읽어 사용자가 어떤 동작을 했는지 파악할 수 있다.
" 검색화면을 구성하는 구조  =>> 검색화면에서 클릭했을때 그 버튼 코드를 자동으로 받아올 수 있는 필드/ 버튼을 넣는 필드도 있다.

" ALV 필수 변수
DATA: go_custom  TYPE REF TO cl_gui_custom_container,
      go_grid    TYPE REF TO cl_gui_alv_grid.
*&---------------------------------------------------------------------*


DATA: gt_fcat   TYPE lvc_t_fcat,      " 필드 카탈로그 구조체의 표준 내부 테이블 타입. 필드 카탈로그 테이블.
      gs_fcat   TYPE lvc_s_fcat,      " 필드 카탈로그 작업 영역. 단일 필드 카탈로그 구조체. 필드카탈로그 행.
      gs_layout TYPE lvc_s_layo.      " 레이아웃 구조체. 단일 레이아웃 구조체. 레이아웃 행
*&---------------------------------------------------------------------*


DATA: ok_code   TYPE sy-ucomm.
DATA: gv_title  TYPE sy-title.
*&---------------------------------------------------------------------*


*DATA: OBJFILE TYPE REF TO CL_GUI_FRONTEND_SERVICES. " 클래스를 참조해서 선언했다. =>> OBJFILE이 대신 받아서 클래스 메소드를 실행할 수 있게
*
* OLE2_OBJECT 타입: ABAP이 클라이언트 PC의 Excel 애플리케이션과 통신할 때, Excel 내부의 특정 구성 요소(객체)를 가리키는 역할
*DATA: GO_APPLICATION  TYPE OLE2_OBJECT,
*       엑셀 매크로 기능을 받는 구조. MS Excel 자체를 나타내는 최상위 객체. OLE 통신의 시작점이며, Excel을 실행하고 종료하는 데 사용.
*
*       GO_APPLICATION에 속한 모든 통합 문서(Workbook)들의 집합. 새 통합 문서를 열거나 추가하는 데 사용.
*      GO_BOOKS        TYPE OLE2_OBJECT,  " Workbooks Collection.
*
*       개별적인 통합 문서 파일 하나 (파일 열기/저장/닫기)
*      GO_WBOOK        TYPE OLE2_OBJECT,  " Workbook. 엑셀 매크로 워크북 통합문서 기능을 받는다.
*      GO_BOOK         TYPE OLE2_OBJECT,  " Workbook. 엑셀 매크로 워크북 통합문서 기능을 받는다.
*
*       특정 GO_BOOK에 속한 모든 워크시트(Worksheet)들의 집합
*      GO_SHEETS       TYPE OLE2_OBJECT,  " Sheets Collection
*
*       개별적인 워크시트(시트 탭 하나) (시트 선택/활성화)
*      GO_SHEET        TYPE OLE2_OBJECT,  " Worksheet. 엑셀 매크로 워크시트 기능을 받는다.
*
*       워크시트 내의 개별 셀 또는 전체 셀 집합. 데이터 입력/읽기에 사용
*      GO_CELLS        TYPE OLE2_OBJECT,  " Cells / Cell
*      GO_CELL         TYPE OLE2_OBJECT,
*
*       워크시트 내의 셀 범위(예: A1:C10). 서식 지정이나 블록 데이터 처리에 사용.
*      GO_RANGE        TYPE OLE2_OBJECT,  " Range
*
*       셀 또는 범위의 글꼴 속성. (글꼴 크기, 굵기 등 서식 제어)
*      GO_FONT         TYPE OLE2_OBJECT,  " Font
*
*       워크시트 내의 행
*      GO_ROW          TYPE OLE2_OBJECT,  " Row
*
*      GV_PATH         TYPE STRING,       " 다운로드할 Excel 파일의 전체 파일 경로
*      GV_NUM          TYPE I.
*&---------------------------------------------------------------------*

DATA: BEGIN OF GS_EXCEL,  " 업로드하는 구조
        MANDT       TYPE ZSCARR-MANDT,
        CARRID      TYPE ZSCARR-CARRID,
        CARRNAME    TYPE ZSCARR-CARRNAME,
        CURRCODE    TYPE ZSCARR-CURRCODE,
        URL         TYPE ZSCARR-URL,
      END OF GS_EXCEL.
*&---------------------------------------------------------------------*

DATA: BEGIN OF GS_ZSC,  " 오류검증하는 구조
        ZSTATUS     TYPE ICON-ID,         " 아이콘
        MANDT       TYPE ZSCARR-MANDT,
        CARRID      TYPE ZSCARR-CARRID,
        CARRNAME    TYPE ZSCARR-CARRNAME,
        CURRCODE    TYPE ZSCARR-CURRCODE,
        URL         TYPE ZSCARR-URL,
        ZRESULT     TYPE CHAR200,         " 비고란
      END OF GS_ZSC.
*&---------------------------------------------------------------------*
DATA: GT_ZSC    LIKE TABLE OF GS_ZSC,   " 오류검증한 테이블
      GT_EXCEL  LIKE TABLE OF GS_EXCEL. " 업로드할 테이블
*&---------------------------------------------------------------------*

* 엑셀 업로드

" Excel 업로드 함수를 호출할 때, 업로드된 데이터를 담을 실제 내부 테이블 (예: GT_EXCEL)을 <gt_data>에 할당한다.
" 업로드할 데이터를 모르기때문에 모르는 거를 넣을 수 있는 구조를 사용 =>> 필드심볼
FIELD-SYMBOLS : <gt_data>       TYPE STANDARD TABLE .
*&---------------------------------------------------------------------*

* DB 데이터
DATA: GT_ZSCARR TYPE TABLE OF ZSCARR,   " ZSCARR 불러오는 테이블
      GS_ZSCARR TYPE          ZSCARR.   " LOOP돌릴때 필요한 행구조
*&---------------------------------------------------------------------*

*툴바 관련
" ALV(ABAP List Viewer) 툴바의 특정 버튼이나 기능 코드를 제어(비활성화/숨김)하기 위해 사용
" ALV Grid 컨트롤은 기본적으로 많은 표준 기능(저장, 정렬, 합계 등)을 툴바에 제공.
" fcode 내부 테이블은 개발자가 표준 ALV 툴바에서 숨기거나 비활성화하고 싶은 기능 코드(버튼)들을 모아서 저장하는 데 사용된다.
" 툴바에서 '인쇄' 버튼을 숨기고 싶다면, '인쇄' 기능 코드(&PRINT 등)를 이 테이블에 추가.
DATA: fcode    TYPE TABLE OF sy-ucomm,    " 업로드 모드일때 버튼 코드 안나오게 하려고
      wa_fcode TYPE sy-ucomm.
*&---------------------------------------------------------------------*
```