```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_SCARR_SCREEN5_F01
*&---------------------------------------------------------------------*
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

```
