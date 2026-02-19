```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_20260219_ZSCARR_2_TOP
*&---------------------------------------------------------------------*


TYPE-POOLS: OLE2.

TABLES: sscrfields.   " Selection Screen의 기능 코드(ucomm) 제어용
" 사용자가 버튼, 메뉴 등을 클릭. 엔터를 누를 때 발생하는 기능 코드(Function Code)를 SSCRFIELDS-UCOMM 필드에 저장된다.

*&---------------------------------------------------------------------*
* ALV 관련.
*
* ALV Grid와 같은 GUI 컨트롤을 ABAP 화면(Dynpro) 내에 배치할 수 있도록 영역(Area)을 정의하는 컨테이너 객체.
* CL_GUI_CUSTOM_CONTAINER 레이아웃 버튼 눌러서 커스텀 컨트롤로 그린 화면을 받아오는 클래스.
" 개발자가 화면 레이아웃에서 특정 영역을 미리 정의하고, 프로그램 실행 시
" 이 GO_CUSTOM 객체를 생성하여 그 영역을 ALV Grid를 담을 수 있는 '상자'로 만든다.

* CL_GUI_ALV_GRID
" 프로그램 실행 시 이 GO_GRID 객체를 생성하고, 이 객체가 위에서 생성된 GO_CUSTOM 컨테이너 안에 '쏙' 들어가서 데이터를 표시.
" 데이터를 표 형태로 표시하고, 정렬, 필터링, 합계 등 ALV의 모든 표준 기능을 제공
*&---------------------------------------------------------------------*
DATA: go_custom TYPE REF TO CL_GUI_CUSTOM_CONTAINER, " 이 줄이 반드시 있어야 함
      go_grid   TYPE REF TO CL_GUI_ALV_GRID.

*&---------------------------------------------------------------------*
* ALV Grid 객체(CL_GUI_ALV_GRID)를 사용하여 리포트를 출력할 때, 표의 형태와 레이아웃을 정의하는 데 사용되는 변수.
* LVC는 ALV Grid 컨트롤(CL_GUI_ALV_GRID)에서 사용되는 데이터 타입 집합.
*&---------------------------------------------------------------------*
DATA: gt_fcat   TYPE LVC_T_FCAT,  " ALV 리포트에 표시될 모든 열(Column)에 대한 정의 정보를 저장하는 메인 테이블
      " ALV 리포트에 표시될 모든 열(Column)에 대한 정의 정보를 저장하는 메인 테이블
      " 이 테이블의 각 행은 ALV의 한 열을 나타냅니다.
      " 여기에 열 이름, 데이터 타입, 필드 라벨(열 제목), 출력 길이, 정렬 가능 여부, 숨김 여부 등 모든 메타데이터를 담게 됩니다.

      gs_fcat   TYPE LVC_S_FCAT,  " gs_fcat의 행구조.

      gs_layout TYPE LVC_S_LAYO.  " 레이아웃 구조체. 단일 레이아웃 구조체. 레이아웃 행
      " ZEBRA: 행에 줄무늬(얼룩말 무늬)를 표시.
      " GRID_TITLE: ALV 표 상단에 표시될 제목.
      " NO_TOOLBAR: 툴바를 숨김
      " EDIT: ALV를 편집 모드로 설정


*&---------------------------------------------------------------------*
* 사용자가 현재 화면에서 수행한 액션(기능 코드)을 저장.
" 1. 사용자가 ABAP 프로그램 화면에서 '저장' 버튼을 클릭.
" 2. 이 '저장' 버튼에 할당된 기능 코드(예: 'SAVE')가 SY-UCOMM 필드에 자동으로 채워진다.
" 3. SAP 시스템은 PAI 이벤트를 발생시키며 프로그램의 PAI 로직으로 이동..
" 4. ABAP 프로그램은 PAI 로직 내에서 IF SY-UCOMM = 'SAVE'. 와 같은 코드를 사용하여 사용자가 어떤 액션을 취했는지 판단하고,
" 5. 맞는 저장 로직을 수행.
*&---------------------------------------------------------------------*
DATA: OK_CODE   TYPE SY-UCOMM.           " 버튼 값을 받을 변수
DATA: gv_title  TYPE sy-title.           " 화면 타이틀 바 텍스트



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





*&---------------------------------------------------------------------*
DATA: GT_ZSC    LIKE TABLE OF GS_ZSC,   " 오류검증한 테이블
      GT_EXCEL  LIKE TABLE OF GS_EXCEL. " 업로드할 테이블

DATA: GT_ZSCARR TYPE TABLE OF ZSCARR,   " ZSCARR 불러오는 테이블
      GS_ZSCARR TYPE          ZSCARR.   " LOOP돌릴때 필요한 행구조
*&---------------------------------------------------------------------*

*&---------------------------------------------------------------------*
FIELD-SYMBOLS : <gt_data>       TYPE STANDARD TABLE .
*&---------------------------------------------------------------------*

* 툴바 관련
" fcode 내부 테이블은 개발자가 표준 ALV 툴바에서 숨기거나 비활성화하고 싶은 기능 코드(버튼)들을 모아서 저장하는 데 사용된다.
" 툴바에서 '인쇄' 버튼을 숨기고 싶다면, '인쇄' 기능 코드(&PRINT 등)를 이 테이블에 추가.
DATA: fcode    TYPE TABLE OF sy-ucomm,    " 화면에서 제외할 기능 코드(Function Code) 리스트
      wa_fcode TYPE sy-ucomm.             " 기능 코드 추가를 위한 작업 영역(Work Area)
*&---------------------------------------------------------------------*
```