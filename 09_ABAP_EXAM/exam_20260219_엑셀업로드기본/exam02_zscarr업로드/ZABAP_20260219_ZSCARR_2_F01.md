```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_20260219_ZSCARR_2_F01
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Form CREATE_OBJECT_INSTANCE
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM create_object_instance .
  CREATE OBJECT go_custom       " CL_GUI_CUSTOM_CONTAINER 클래스의 객체를 생성하여 go_custom 변수에 넣는다.
    EXPORTING
      container_name = 'CON1'.  " 컨테이너이름을 인자로 넣는다.

  CREATE OBJECT go_grid
      EXPORTING
        i_parent = go_custom.
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

  _FCAT: 'ZSTATUS' '상태' 'X' '' '3' '',
         'MANDT' '클라이언트' 'X' '' '5' '',
         'CARRID' '아이디' 'X' '' '5' '',
         'CARRNAME' '이름' '' '' '20' '',
         'CURRCODE' '통화' '' '' '5' '',
         'URL' '사이트' '' '' '30' '',
         'ZRESULT' '비고' '' '' '50' ''.
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
  GS_LAYOUT-ZEBRA   = 'X'.
  GS_LAYOUT-CWIDTH_OPT  = 'A'.
  GS_LAYOUT-SEL_MODE  = 'D'.
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

  CALL METHOD GO_GRID->SET_TABLE_FOR_FIRST_DISPLAY
    EXPORTING
      IS_LAYOUT                     = GS_LAYOUT     " 레이아웃 구조체. 단일 레이아웃 구조체. 레이아웃 행
    CHANGING
      IT_OUTTAB                     = GT_ZSC        " 오류검증한 테이블
      IT_FIELDCATALOG               = GT_FCAT       " 필드 카탈로그 구조체의 표준 내부 테이블 타입. 필드 카탈로그 테이블.
        .










*   이거 안써주면 더블클릭 안먹힘!!!
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
  go_grid->refresh_table_display( ).
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
  DATA : LT_FILE TYPE FILETABLE,    " 다이얼로그에서 선택한 파일 경로와 정보가 담기는 내부 테이블.  List<FileDTO> list (리스트)
         LS_FILE TYPE FILE_TABLE,   " 파일 경로 정보 (Structure). 하나의 파일 정보를 넣는다.
         LV_RC   TYPE I.            " 리턴 코드(Return Code) 파일 다이얼로그가 성공적으로 닫혔는지 판단.



  " CL_GUI_FRONTEND_SERVICES : 클라이언트 PC의 기능을 제어하는 SAP 표준 클래스
  " FILE_OPEN_DIALOG() : 스태틱 메소드이다. 파일 열기 다이얼로그를 출력.
  CALL METHOD CL_GUI_FRONTEND_SERVICES=>FILE_OPEN_DIALOG
    CHANGING                    " 메소드가 실행된 후 값이 변해서 돌아오는 변수를 지정.
      FILE_TABLE = LT_FILE      " 사용자가 파일 탐색기에서 선택한 파일들의 경로가 LT_FILE에 저장.
      RC         = LV_RC.       " 열기(Open)를 눌렀으면 LV_RC에 1이상의 값이, 취소(Cancel) 0 저장.

  " LT_FILE(List)에서 첫 번째(INDEX 1) 데이터를 꺼내서 LS_FILE(작업 영역/객체)에 넣는다.
  READ TABLE LT_FILE INTO LS_FILE INDEX 1.

  IF SY-SUBRC = 0.      " 파일을 성공적으로 읽어왔다면(파일을 선택하고 '열기'를 눌렀다면)
    P_FILE = LS_FILE.   " LS_FILE 구조체에 담긴 파일 경로를 P_FILE에 넣는다.
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
    " 현재 이벤트 블록(START-OF-SELECTION)의 실행을 즉시 중단하고, 프로그램 제어를 선택 화면(Selection Screen)로 이동.
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
  DATA : lv_filename      TYPE string,      " 파일 경로 문자열
         lt_records       TYPE solix_tab,   " 2진 데이터를 쪼개서 담을 테이블 (Binary Table)
         lv_headerxstring TYPE xstring,     " 쪼개진 데이터를 하나로 (Hex String)
         lv_filelength    TYPE i.           " 읽어온 파일의 실제 크기(바이트). 자바의 byte[]

  lv_filename = p_file.

  " PC에 있던 엑셀 파일을 SAP 서버 메모리(RAM)로 복사.
  CALL FUNCTION 'GUI_UPLOAD'
    EXPORTING
      filename                = lv_filename         " 자바의 FileInputStream(path) 처럼 읽어올 파일 경로를 전달
      filetype                = 'BIN'
    IMPORTING
      filelength              = lv_filelength       " 읽어들인 파일의 총 바이트(Byte) 크기
      header                  = lv_headerxstring    " 파일의 헤더 정보
    TABLES
      data_tab                = lt_records          " 읽어온 이진 데이터를 SAP의 '표(Internal Table)' 에 넣는다. 자바의 List<byte[]>
    EXCEPTIONS                                      " 예외 처리. 각 번호는 sy-subrc 시스템 변수에 할당
      file_open_error         = 1                   " 파일을 열 수 없음 (파일이 없거나 이미 열려 있음)
      file_read_error         = 2                   " 파일 읽기 실패
      no_batch                = 3                   " 백그라운드 모드에서는 GUI_UPLOAD 사용 불가
      gui_refuse_filetransfer = 4                   " GUI에서 전송을 거부함
      invalid_type            = 5                   " 잘못된 파일 타입
      no_authority            = 6                   " 권한 없음
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
  " &------------------------------------------------------------------------------------------*


  " 바이너리 데이터를 xstring(가로로 긴 16진수)으로 변환
  " 만약 OData 환경에서 cl_fdt_xl_spreadsheet를 사용 중이라면 이 단계는 건너뜁니다.
  " 왜냐하면 엑셀 파일이 이미 xstring 형태(바이너리 통째)로 들어와 있기 때문입니다.

  CALL FUNCTION 'SCMS_BINARY_TO_XSTRING'
    EXPORTING
      input_length = lv_filelength        " 입력 파라미터. 합칠 데이터의 전체 크기(바이트)
    IMPORTING
      buffer       = lv_headerxstring     " 출력 파라미터. 조각들을 본드로 붙여서 완성한 하나의 커다란 데이터
    TABLES
      binary_tab   = lt_records           " 입력 파라미터. 이전 단계(GUI_UPLOAD)에서 조각조각 담아온 데이터 리스트
    EXCEPTIONS
      failed       = 1
      OTHERS       = 2.
  " &------------------------------------------------------------------------------------------*


  IF sy-subrc <> 0.
    "Implement suitable error handling here
  ENDIF.


  " 엑셀 파일을 핸들링할 수 있는 기능을 가진 클래스(cl_fdt_xl_spreadsheet)의 참조 변수를 선언..
  DATA : lo_excel_ref TYPE REF TO cl_fdt_xl_spreadsheet .

  " 객체 생성 및 생성자 호출.
  " cl_fdt_xl_spreadsheet 클래스의 생성자를 호출하여 lo_excel_ref 객체를 생성
  " document_name: 객체의 메타데이터 정보로 사용될 파일명을 바인딩
  " xdocument    : Raw Data(xstring)를 넘겨주면, 클래스 내부의 파서(Parser)가 데이터를 분석(Parsing)
  TRY .
      lo_excel_ref = NEW cl_fdt_xl_spreadsheet(
                              document_name = lv_filename
                              xdocument     = lv_headerxstring ) .
    CATCH cx_fdt_excel_core.
      "Implement suitable error handling here
  ENDTRY .
  " &------------------------------------------------------------------------------------------*


  " 인스턴스화된 객체로부터 실제 데이터를 추출하기 위한 첫 번째 Method Call 단계.
  " 참조변수->인터페이스명~메서드명
  " 인터페이스 이름을 거쳐서(Casting)
  " 호출하려는 get_worksheet_names() 메소드가 해당 클래스 고유의 것이 아니라, 인터페이스로부터 물려받아 구현한 것이기 때문에 인터페이스 이름을 거쳐서(Casting)
  "
  " 자바와 비교하면 cl_fdt_xl_spreadsheet 클래스의 객체 lo_excel_ref를 생성하고.
  " cl_fdt_xl_spreadsheet lo_excel_ref = new cl_fdt_xl_spreadsheet(xdocument);
  " lo_excel_ref 이 객체의 get_worksheet_names() 메소드를 호출하는데..
  " if_fdt_doc_spreadsheet 이 인터페이스 타입으로 캐스팅해서 저장하겠다.
  " ((if_fdt_doc_spreadsheet) lo_excel_ref).get_worksheet_names();
  "
  " 실행결과 : 엑셀 파일 내에 존재하는 모든 워크시트의 이름이 담긴 Internal Table이 생성
  "         예시) Index 1: "Sheet1"

  lo_excel_ref->if_fdt_doc_spreadsheet~get_worksheet_names(
    IMPORTING                                   " 출력 파라미터
      worksheet_names = DATA(lt_worksheets) ).  " 동적으로 바로 만든 변수 lt_worksheets에 값이 리턴된다.


  " 엑셀 객체 내부의 데이터를 SAP가 핸들링할 수 있는 메모리 주소로 연결(Mapping)
  IF NOT lt_worksheets IS INITIAL.
    " 이전 단계에서 만들어서 받은 lt_worksheets에서 시트 이름 추출해서 lv_woksheetname 변수를 만들어서 넣는다.
    " String sheetName = sheetNames.get(0);
    READ TABLE lt_worksheets INTO DATA(lv_woksheetname) INDEX 1.

    " 데이터 추출. lo_excel_ref 객체의 get_itab_from_worksheet() 메소드 호출. 파라미터로 lv_woksheetname를 넣는다.
    " get_itab_from_worksheet() 메소드를 실행하면 해당 시트의 내용을 통째로 읽어 Dynamic Internal Table을 생성하고,
    " 메모리 주소를 lo_data_ref 변수에 넣는다.
    DATA(lo_data_ref) = lo_excel_ref->if_fdt_doc_spreadsheet~get_itab_from_worksheet(
                                             lv_woksheetname ).
    " 위에서 만든 lo_data_ref 포인터 변수에 있는 주소를 <gt_data> 에 바인딩.
    ASSIGN lo_data_ref->* TO <gt_data>.
  ENDIF.
  " &------------------------------------------------------------------------------------------*

  " 2차원 동적 배열을 순회하며 인덱스 기반으로 값을 추출하여 DTO(Data Transfer Object)에 매핑하는 과정

  DATA : lv_numberofcolumns   TYPE i,         " 컬럼
         lv_date_string       TYPE string,    "
         lv_target_date_field TYPE datum.


  FIELD-SYMBOLS : <ls_data>  TYPE any,        " 반복에서 현재 읽고 있는 행(Row) 전체를 가리키는 포인터
                  <lv_field> TYPE any.        " 한 행 안에서 특정 열(Column/Cell)의 값을 가리키는 포인터

  lv_numberofcolumns = 5 .                    " 매핑할 컬럼 개수


  LOOP AT <gt_data> ASSIGNING <ls_data> FROM 2 .  " 엑셀 데이터가 담긴 테이블을 2번째 줄부터 반복.
  " <gt_data>에서 한 행씩 읽어서 <ls_data>에 넣는다.

    DO lv_numberofcolumns TIMES.                  " 컬럼 개수 만큼 반복(5번)

      " <ls_data> 구조체(행)에서 sy-index번째 필드(열)의 메모리 주소를 찾아 <lv_field>에 연결
      " sy-index: 현재 DO 루프가 몇 번째 돌고 있는지 나타내는 시스템 변수.
      ASSIGN COMPONENT sy-index OF STRUCTURE <ls_data> TO <lv_field> .

      IF sy-subrc = 0 .
        CASE sy-index .
          when 1 .                            " sy-index가 1이면
            gs_EXCEL-MANDT = <lv_field>.      " 첫 번째 열의 값을 MANDT
          when 2 .                            " sy-index가 2이면
            gs_EXCEL-CARRID = <lv_field>.     " 두 번재 열의 값을 CARRID
          when 3 .                            " sy-index가 2이면
            gs_EXCEL-CARRNAME = <lv_field>.   " 세 번재 열의 값을 CARRNAME
          when 4 .                            " sy-index가 2이면
            gs_EXCEL-CURRCODE = <lv_field>.   " 네 번재 열의 값을 CURRCODE
          when 5 .                            " sy-index가 2이면
            gs_EXCEL-URL = <lv_field>.        " 다섯 번재 열의 값을 URL
          WHEN OTHERS.
*            WRITE : <lv_field> .
        ENDCASE .
      ENDIF.

    ENDDO .

    " 한 줄의 매핑이 끝나면 완성된 gs_EXCEL 구조체를 테이블gt_EXCEL에 추가
    APPEND gs_EXCEL TO gt_EXCEL.

    CLEAR gs_EXCEL.
*    NEW-LINE .
  ENDLOOP .


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
* 필요 데이터 취합. 중복체크할 데이터를 불러온다.
  PERFORM GET_NEEDED_DATA.  " 엑셀 데이터가 DB의 기존 데이터와 중복되는지 확인하기 위해 기준 데이터를 가져온다.
* 업로드 조건에 따라 ALV 출력 데이터 취합. 아이콘과 비고란 채워서 오류검증
  PERFORM GET_ZSC_DATA.
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
    FROM ZSCARR
    INTO CORRESPONDING FIELDS OF TABLE GT_ZSCARR.
    SORT GT_ZSCARR BY CARRID.
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
  " SAP 표준 통화키(Currency Key) 체크를 위한 마스터 테이블
  DATA: GT_SCURX TYPE TABLE OF SCURX,
        GS_SCURX TYPE          SCURX.

  SELECT * FROM SCURX INTO TABLE GT_SCURX.    " 시스템에 정의된 모든 통화키를 메모리로 로드


  LOOP AT GT_EXCEL INTO GS_EXCEL.             " 엑셀 데이터를 담은 gt_excel 반복

    " 같은 필드명을 가진 데이터를 TOP에서 만든 오류 검증하는 구조 GS_ZSC에 복사.
    MOVE-CORRESPONDING GS_EXCEL TO GS_ZSC.

    " 1. CARRID (Primary Key) 키 검증
    IF GS_ZSC-CARRID IS INITIAL.              " 필수 키값이 누락된 경우
          GS_ZSC-ZSTATUS = ICON_LED_RED.
          GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[CARRID 키값이 없습니다.]'.
    ELSE.                                     " 중복 체크 (GT_ZSCARR와 비교)
          SORT GT_ZSCARR BY CARRID.

          " GT_ZSCARR 테이블에서 데이터를 한 줄 읽어서 GS_ZSCARR라는 구조체(Work Area)에 넣기.
          " 검색 조건에 맞는것을 넣는다. CARRID 필드 값이 현재 엑셀 줄의 CARRID와 같은 것.
          " BINARY SEARCH 를 사용한다. 그래서 위에서 미리 정렬을 해야한다.
          READ TABLE GT_ZSCARR INTO GS_ZSCARR
                               WITH KEY CARRID = GS_ZSC-CARRID
                               BINARY SEARCH.


          IF SY-SUBRC = 0.                      " 이미 DB에 존재하는 키값이면 에러 처리.
             GS_ZSC-ZSTATUS = ICON_LED_RED.
             GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[CARRID 키값이 이미 들어있습니다.]'.
          ENDIF.

    ENDIF.

    " 2. CURRCODE (Foreign Key) 검증
    IF GS_ZSC-CURRCODE IS INITIAL.              " 필수 키값이 누락된 경우
          GS_ZSC-ZSTATUS = ICON_LED_RED.
          GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[CURRCODE 값이 없습니다.]'.
    ELSE.                                       " 마스터 테이블(GT_SCURX)에 존재하는 통화키인지 확인

          " GT_SCURX 테이블에서 데이터를 한 줄 읽어서 GS_SCURX라는 구조체(Work Area)에 넣기.
          " 검색 조건에 맞는것을 넣는다. CURRKEY 필드 값이 현재 엑셀 줄의 CURRCODE와 같은 것.
          READ TABLE GT_SCURX INTO GS_SCURX
                              WITH KEY CURRKEY = GS_ZSC-CURRCODE.
          IF SY-SUBRC <> 0.                     " SAP 표준에 정의되지 않은 통화키인 경우 에러.
             GS_ZSC-ZSTATUS = ICON_LED_RED.
             GS_ZSC-ZRESULT = GS_ZSC-ZRESULT && '[CURRCODE에 없는 통화키를 입력했습니다.]'.
          ENDIF.
    ENDIF.


    IF GS_ZSC-ZRESULT IS INITIAL.           " 에러 메시지(ZRESULT)가 비어있다면 검증을 통과
      GS_ZSC-ZSTATUS = ICON_LED_YELLOW.     " 준비완료 아이콘 표시
    ENDIF.

    APPEND GS_ZSC TO GT_ZSC.    " 검증 결과가 포함된 행을 ALV 출력용 테이블에 추가
    CLEAR: GS_EXCEL, GS_ZSC.

  ENDLOOP.
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
    PERFORM SAVE_ZSC_FINAL CHANGING <FS_ZSC>.
  ENDLOOP.

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
    MOVE-CORRESPONDING GS_ZSC TO GS_ZSCARR.
    GS_ZSCARR-MANDT = SY-MANDT.

    INSERT INTO ZSCARR VALUES GS_ZSCARR.

    IF SY-SUBRC = 0.
        GS_ZSC-ZSTATUS = ICON_LED_GREEN.
        GS_ZSC-ZRESULT = '저장 성공'.
      COMMIT WORK AND WAIT.
    ELSEIF SY-SUBRC <> 0.
        GS_ZSC-ZSTATUS = ICON_LED_RED.
        GS_ZSC-ZRESULT = '저장 실패'.
    ENDIF.
  ENDIF.
ENDFORM.
```