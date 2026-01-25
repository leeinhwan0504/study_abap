```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_BOOK_SCREEN_TOP
*&---------------------------------------------------------------------*

* 1. 시스템 선언 및 인프라 (Type Pools & Tables)
TYPE-POOLS: OLE2.     " TYPES:   OLE2_OBJECT LIKE   OBJ_RECORD.  =>> 이 구조를 매크로 돌리기 위해서 사용했다.

TABLES: sscrfields.   " Selection Screen의 기능 코드(ucomm) 제어용
" 사용자가 화면에서 버튼을 클릭하거나, 메뉴를 선택하거나, 엔터를 누를 때 발생하는 기능 코드(Function Code)가
" SSCRFIELDS-UCOMM 필드에 자동으로 저장.
*&---------------------------------------------------------------------*
* 2. 화면 제어 및 상태 변수 (Global UI Variables)
DATA: ok_code   TYPE sy-ucomm.    " Screen PAI 기능 코드
DATA: gv_title  TYPE sy-title.    " 화면 타이틀 바 텍스트
*&---------------------------------------------------------------------*

* 3. ALV 컨트롤 객체 (ALV 필수 변수)
" 화면에 ALV를 그리기 위한 도구들입니다.
DATA: go_custom  TYPE REF TO cl_gui_custom_container,     " ALV가 담길 그릇
      go_grid    TYPE REF TO cl_gui_alv_grid.             " 실제 ALV 그리드
*&---------------------------------------------------------------------*
* 4. ALV 구성 정보 (ALV 설정값)
DATA: gt_fcat   TYPE lvc_t_fcat,      " 컬럼의 이름, 길이 등을 설정하는 테이블
      gs_fcat   TYPE lvc_s_fcat,      " 한 줄씩 설정할 때 쓰는 작업 영역. 필드 카탈로그 작업 영역. 필드카탈로그 행.
      gs_layout TYPE lvc_s_layo.      " 레이아웃 구조체. ALV 전체 디자인 (줄무늬, 열 너비 등)
*&---------------------------------------------------------------------*
* 5. 비즈니스 데이터 구조 (Internal Tables & Structures)
" A. 엑셀 업로드 원본 구조
DATA: BEGIN OF GS_EXCEL,
        MANDT         TYPE ZEMP-MANDT,            " 클라이언트
        EMPNO         TYPE ZEMP-EMPNO,            " 사원번호
        ENAME         TYPE ZEMP-ENAME,            " 사원이름
        PROJECT_ID    TYPE ZPROJECT-PROJECT_ID,   " 프로젝트 ID
        PROJECT_NAME  TYPE ZPROJECT-PROJECT_NAME, " 프로젝트 이름
        MODULE_NAME   TYPE ZMODULE-MODULE_NAME,   " 모듈이름
        START_DATE    TYPE ZHISTORY-START_DATE,   " 시작 날짜
        END_DATE      TYPE ZHISTORY-END_DATE,     " 종료 날짜
      END OF GS_EXCEL.
*&---------------------------------------------------------------------*
*" B. 검증 결과(아이콘 등)를 포함하여 ALV에 보여줄 구조
DATA: BEGIN OF GS_ZSC,  " 오류검증하는 구조
        ZSTATUS       TYPE ICON-ID,               " 아이콘
        EMPNO         TYPE ZEMP-EMPNO,            " 사원번호
        ENAME         TYPE ZEMP-ENAME,            " 사원이름
        PROJECT_ID    TYPE ZPROJECT-PROJECT_ID,   " 프로젝트 ID
        PROJECT_NAME  TYPE ZPROJECT-PROJECT_NAME, " 프로젝트 이름
        MODULE_NAME   TYPE ZMODULE-MODULE_NAME,   " 모듈이름
        START_DATE    TYPE ZHISTORY-START_DATE,   " 시작 날짜
        END_DATE      TYPE ZHISTORY-END_DATE,     " 종료 날짜
        ZRESULT       TYPE CHAR200,               " 비고란
      END OF GS_ZSC.
**&---------------------------------------------------------------------*
DATA: GT_ZSC    LIKE TABLE OF GS_ZSC,             " 오류검증한 테이블
      GT_EXCEL  LIKE TABLE OF GS_EXCEL.           " 업로드할 테이블
**&---------------------------------------------------------------------*
* 6. 데이터베이스 검증용 버퍼
" DB에 이미 데이터가 있는지 확인(중복 체크)하기 위해 기존 데이터를 불러와 보관하는 용도입니다.
DATA: GT_ZHISTORY TYPE TABLE OF ZHISTORY, " 기존 이력 데이터를 담을 테이블
      GS_ZHISTORY TYPE          ZHISTORY. " LOOP로 데이터를 하나씩 꺼내올 때 쓰는 구조체
**&---------------------------------------------------------------------*
* 7. 동적 처리 및 특수 제어 (Dynamic & UI Control)
* 엑셀 업로드
" Excel 업로드 함수를 호출할 때, 업로드된 데이터를 담을 실제 내부 테이블 (예: GT_EXCEL)을 <gt_data>에 할당한다.
" 업로드할 데이터를 모르기때문에 모르는 거를 넣을 수 있는 구조를 사용 =>> 필드심볼
FIELD-SYMBOLS : <gt_data>       TYPE STANDARD TABLE .
*&---------------------------------------------------------------------*
* 툴바 관련
DATA: fcode    TYPE TABLE OF sy-ucomm,    " 화면에서 제외할 기능 코드(Function Code) 리스트
      wa_fcode TYPE sy-ucomm.             " 기능 코드 추가를 위한 작업 영역(Work Area)
*&---------------------------------------------------------------------*
```