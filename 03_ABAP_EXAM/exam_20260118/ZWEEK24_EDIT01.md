```abap
*&---------------------------------------------------------------------*
*& Report ZWEEK23_EDIT01
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZWEEK23_EDIT01.

TABLES : zscarr.                    " zcarr 테이블을 출력력할 것이다.

TYPES: BEGIN OF t_zscarr.
        INCLUDE TYPE zscarr.        " zscarr 구조를 가져온다.
        TYPES: box.                 " 선택했을때 x가 들어가게 해서 선택한 것만 삭제하기
TYPES: END OF t_zscarr.

DATA: it_zscarr TYPE TABLE OF t_zscarr WITH HEADER LINE,    " 출력할 데이터를 넣는 테이블. 사용자가 지금 화면에서 보고 수정하고 있는 데이터.
      wa_zscarr TYPE t_zscarr.                              " 한 행씩 불러온다.

DATA: it_zscarrcp TYPE TABLE OF t_zscarr WITH HEADER LINE,  " 수정을 시작하기 전, DB에서 get_data로 불러와서넣은 데이터를 저장하는 테이블.
      it_changes  TYPE TABLE OF t_zscarr WITH HEADER LINE,  " 변경한 것만 저장 테이블
      it_deletes  TYPE TABLE OF t_zscarr WITH HEADER LINE.  " 삭제한 것만 저장 테이블

DATA: togl TYPE c.                                          " 편집/조회 모드 왔다 갔다 하도록


*DATA: it_alv TYPE STANDARD TABLE OF t_zscarr INITIAL SIZE 0,
*      wa_alv TYPE t_zscarr.
*DATA: it_zscarr TYPE TABLE OF zscarr WITH HEADER LINE,
*      wa_zscarr TYPE zscarr.
*
*DATA: it_zscarrcp TYPE STANDARD TABLE OF zscarr,
*      it_changes  TYPE STANDARD TABLE OF zscarr.

DATA: wa_fieldcat TYPE slis_fieldcat_alv,                   " 필드가탈로그 행
      it_fieldcat TYPE slis_t_fieldcat_alv,                 " 필트카탈로그 테이블
      ls_layout   TYPE slis_layout_alv.


START-OF-SELECTION.
  PERFORM get_data.
  PERFORM build_fieldcat.
  PERFORM build_layout.
  PERFORM alv_grid.

" 툴바 만드는 함수
" ------------------------------------------------------------------------------------------
FORM pf_status_set USING extab TYPE slis_t_extab.           " extab (제외할 버튼 리스트)

  SET PF-STATUS 'STANDARD'.   " 툴바를 만드는.

ENDFORM.
" ------------------------------------------------------------------------------------------


FORM popup_confirm USING p_button CHANGING p_confirm.     " p_button 파라미터로 받는다. 리턴 p_confirm

  DATA: text_q  TYPE string,                              " 질문 내용 저장 변수
        text_b1 TYPE string.                              " 버튼 텍스트를 저장 변수

  IF p_button = 'S'.                                      " 파라미터 p_button의 값이 S이면
    text_q  = '변경된 데이터가 있습니다.저장하시겠습니까?'.               " 질문 저장
    text_b1 = '저장'.                                       " 버튼 텍스트 저장
  ELSEIF p_button = 'D'.                                  " 파라미터 p_button의 값이 D이면
    text_q  = '정말 삭제하시겠습니까?'.                            " 질문 저장
    text_b1 = '삭제'.                                       " 버튼 텍스트 저장
  ENDIF.

  CALL FUNCTION 'POPUP_TO_CONFIRM'
    EXPORTING
      titlebar              = 'POPUP'                   " 팝업창의 제목 (Title)
      text_question         = text_q                    " 표시될 질문
      text_button_1         = text_b1                   " 첫 번째 버튼의 이름 (예: 저장, 삭제)
      icon_button_1         = 'ICON_CHECKED'            " 버튼에 표시될 아이콘
      text_button_2         = '취소'                      " 두 번째 버튼의 이름
      icon_button_2         = 'ICON_INCOMPLETE'         " 버튼에 표시될 아이콘
      default_button        = '1'                       " 기본적으로 포커스가 가 있을 버튼
      display_cancel_button = space                     " 취소(Cancel) 버튼 별도 표시 여부 (space = No)
    IMPORTING
      answer                = p_confirm.                " 사용자가 누른 결과값 리턴 (1: 첫번째 버튼, 2: 두번째 버튼, A: 중단)
ENDFORM.

FORM f_save_data.
  DATA: wa_zscarrcp   TYPE t_zscarr,          " 복사한 행 비교
        wa_zscarr_tmp TYPE zscarr.            " DB 테이블과 똑같은 구조에 넣어서 적용해햐 하기 때문에



  " 화면 데이터(it_zscarr) 중에 빈 값이 하나라도 있는가? 를 찾는다.
  " it_zscarr: 출력할 데이터를 넣는 테이블. 사용자가 지금 화면에서 보고 수정하고 있는 데이터.
  READ TABLE it_zscarr WITH KEY carrname = ''.  " carrname을 읽어오면 빈 값이다.
  IF sy-subrc = '0'.
    MESSAGE '입력하지 않은 값이 있습니다.' TYPE 'I'.
    LEAVE SCREEN.
  ENDIF.

  READ TABLE it_zscarr WITH KEY currcode = ''.  " currcode을 읽어오면 빈 값이다.
  IF sy-subrc = '0'.
    MESSAGE '입력하지 않은 값이 있습니다.' TYPE 'I'.
    LEAVE SCREEN.
  ENDIF.

  READ TABLE it_zscarr WITH KEY url = ''.       " url을 읽어오면 빈 값이다.
  IF sy-subrc = '0'.
    MESSAGE '입력하지 않은 값이 있습니다.' TYPE 'I'.
    LEAVE SCREEN.
  ENDIF.
  " --------------------------------------------------------------------------------

  DATA : it_scurx TYPE TABLE OF scurx.          " 체크 테이블을 만들 테이블 구조.
  DATA : wa_scurx TYPE scurx.                   " 행으로 불러오는 구조

  SELECT * FROM scurx INTO TABLE it_scurx.      " 데이터를 it_scurx(체크 테이블)에 읽어온다.

  " it_zscarr: 출력할 데이터를 넣는 테이블. 사용자가 지금 화면에서 보고 수정하고 있는 데이터.
  LOOP AT it_zscarr INTO wa_zscarr.

    " it_zscarr에서 한 행을 읽어 wa_zscarr에 넣는다.
    " wa_zscarr(화면 데이터)의 통화 코드(wa_zscarr-currcode)와 마스터 테이블(it_scurx)의 키 값(currkey)이 같은지 확인
    READ TABLE it_scurx INTO wa_scurx
                        WITH KEY currkey = wa_zscarr-currcode.

    IF sy-subrc <> 0.                           " 못 불러왔다면
      MESSAGE 'CURRCODE에 들어있지 않은 값을 넣었습니다' TYPE 'I'.
      LEAVE SCREEN.                             " 화면을 빠져나간다
    ENDIF.
  ENDLOOP.

  " --------------------------------------------------------------------------------

  " 변경된 데이터를 넣을 테이블. [] 테이블 표시.
  REFRESH it_changes[].

  LOOP AT it_zscarr INTO wa_zscarr.  " it_zscarr(사용자가 지금 화면에서 보고 수정하고 있는 데이터)에서 한 행을 읽어서 wa_zscarr에 넣는다.

    " 현재 루프의 순번(sy-tabix)을 이용해서, get_data에서 불러와서 저장했던 원본데이터 it_zscarrcp 테이블의 한 행을 읽어서 wa_zscarrcp에 넣는다.
    READ TABLE it_zscarrcp INTO wa_zscarrcp INDEX sy-tabix.

    IF wa_zscarrcp NE wa_zscarr.      " 원본 데이터(wa_zscarrcp)와 현재 화면 데이터(wa_zscarr)가 다른지 비교

      MOVE-CORRESPONDING wa_zscarr TO wa_zscarr_tmp.  " 화면용 데이터에서 DB 테이블 구조와 이름이 일치하는 필드만 복사 (Field Mapping)

      MODIFY zscarr FROM wa_zscarr_tmp.   " 실제 DB 테이블(zscarr)에 반영 (Upsert: 존재하면 Update, 없으면 Insert)

      IF sy-subrc = 0.
        APPEND wa_zscarr TO it_changes.   " 변경된 행 데이터를 it_changes 테이블에
      ELSE.
        MESSAGE '테이블 저장 중 에러가 발생했습니다.' TYPE 'I'.
        LEAVE SCREEN.
      ENDIF.
    ENDIF.

    CLEAR wa_zscarrcp.
  ENDLOOP.
  " --------------------------------------------------------------------------------

  DESCRIBE TABLE it_changes LINES DATA(lines).
  MESSAGE lines && '건의 데이터가 저장되었습니다.' TYPE 'S'.

ENDFORM.

" 버튼이 클릭 될 때 실행되는 함수
FORM user_command USING p_ucomm TYPE sy-ucomm           " 사용자가 클릭한 버튼의 기능 코드(ID)가 저장. (예: 'EDIT', '&DATA_SAVE')
                        p_selfield TYPE slis_selfield.  " 사용자가 클릭한 셀의 정보가 들어있는 구조체.

  DATA : p_confirm  TYPE c.
  DATA : p_button   TYPE c.

  CASE p_ucomm.

    WHEN 'EDIT'.
      " it_fieldcat 라는 리스트를 fieldcat_edit이라는 메소드에 보낼 테니, 그 안에서 내용을 변경(CHANGING)해서 린터하라.
      PERFORM fieldcat_edit CHANGING it_fieldcat.

      IF togl = ''.           " togl 변수가 빈 값이면  (조회)
        togl = 'X'.           " wa_fieldcat-edit 에 x를 넣는다.
      ELSE.                   " togl 변수에 값이 있으면 (편집)
        CLEAR togl.           " wa_fieldcat-edit 에 빈 값을 넣는다.
      ENDIF.

      PERFORM alv_grid.       " 변경된 설정(it_fieldcat)과 현재 모드(togl)를 바탕으로 화면을 다시 그린다.

    WHEN '&BA'.               " 뒤로 가기 버튼. (BACK &BACK &F03은 내장되어 있어서 &BA로 했다)
      LEAVE PROGRAM.          " 프로그램 나가기


    WHEN '&DATA_SAVE'.
      IF togl = 'X'.                        " 현재 프로그램이 '편집 모드'일 때만 저장
        p_button = 'S'.

        IF it_zscarrcp[] NE it_zscarr[].    " 처음 로딩한 원본 데이터(it_zscarrcp)와 현재 화면의 데이터(it_zscarr)를 비교

          " popup_confirm 함수를 호출. p_button를 파라미터로 던지고 리턴 값을 p_confirm 에 받아온다.
          PERFORM popup_confirm USING p_button CHANGING p_confirm.

          IF p_confirm = '1'.               " 팝업 창에서 저장버튼을 클릭하면 p_confirm에 1을 넣어서 리턴한다.
            PERFORM f_save_data.            "
            PERFORM get_data.

            p_selfield-refresh = 'X'.       " 새로고침
            p_selfield-row_stable = 'X'.    " 새로고침 할때 커서 위치 행을 그대로 유지
            p_selfield-col_stable = 'X'.    " 새로고침 할때 커서 위치 열을 그대로 유지
          ELSE.                             " 팝업 창에서 취소버튼을 클릭하면 p_confirm에 2을 넣어서 리턴한다.
            LEAVE SCREEN.
          ENDIF.
        ELSE.
          MESSAGE '변경할 데이터가 없습니다.' TYPE 'I'.
        ENDIF.
      ELSE.
        MESSAGE '조회모드입니다.' TYPE 'I'.
      ENDIF.

    WHEN '&DEL'.

      READ TABLE it_zscarr WITH KEY box = 'X'.

      IF sy-subrc = '0'.
        p_button = 'D'.
*      READ TABLE it_zscarr INTO wa_zscarr INDEX p_selfield-tabindex.
        PERFORM popup_confirm USING p_button CHANGING p_confirm.
        IF p_confirm = '1'.
          PERFORM f_delete_data.
          PERFORM get_data.
          p_selfield-refresh = 'X'.
          p_selfield-row_stable = 'X'.
          p_selfield-col_stable = 'X'.
        ELSE.
          LEAVE SCREEN.
        ENDIF.
      ELSE.
        MESSAGE '삭제할 데이터가 없습니다.' TYPE 'I'.
      ENDIF.

  ENDCASE.

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
  SELECT * FROM zscarr
    INTO CORRESPONDING FIELDS OF TABLE it_zscarr.

  it_zscarrcp[] = it_zscarr[].
ENDFORM.
*&---------------------------------------------------------------------*
*& Form F_DELETE_DATA
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> WA_ZSCARR
*&---------------------------------------------------------------------*
FORM f_delete_data.
  LOOP AT it_zscarr INTO wa_zscarr WHERE box = 'X'.
    DELETE FROM zscarr
    WHERE carrid = wa_zscarr-carrid.

    IF sy-subrc = 0.
      APPEND wa_zscarr TO it_deletes.
    ELSE.
      MESSAGE '테이블 삭제 중 에러가 발생했습니다.' TYPE 'I'.
      LEAVE SCREEN.
    ENDIF.
  ENDLOOP.

  DESCRIBE TABLE it_deletes LINES DATA(lines).
  MESSAGE lines && '건의 데이터가 삭제되었습니다.' TYPE 'S'.


ENDFORM.
*&---------------------------------------------------------------------*
*& Form FIELDCAT_EDIT
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      <-- IT_FIELDCAT
*&---------------------------------------------------------------------*
FORM fieldcat_edit  CHANGING p_it_fieldcat TYPE slis_t_fieldcat_alv.

  " ------------------------------------------------------------------------------------------
  " 수정4
  LOOP AT p_it_fieldcat INTO wa_fieldcat.         " 받아온 p_it_fieldcat를 반복.
    CASE wa_fieldcat-fieldname.                   " wa_fieldcat-fieldname 값에 따라
      WHEN 'CARRID'.                              " 값이 키 값인 경우
                                                  " 아무것도 안한다.
      WHEN OTHERS.                                " 값이 키가 아닌 경우에 편집 가능
        IF togl = ''.                             " togl 변수가 빈 값이면  (조회)
          wa_fieldcat-edit  = 'X'.                " wa_fieldcat-edit 에 x를 넣는다.
        ELSE.                                     " togl 변수에 값이 있으면 (편집)
          wa_fieldcat-edit  = ''.                 " wa_fieldcat-edit 에 빈 값을 넣는다.
        ENDIF.
        MODIFY p_it_fieldcat FROM wa_fieldcat.    " CHANGING으로 넘어온 it_fieldcat에 wa_fieldcat 로 수정하라.
    ENDCASE.

  ENDLOOP.
  " ------------------------------------------------------------------------------------------

*  LOOP AT p_it_fieldcat INTO wa_fieldcat.     " 받아온 p_it_fieldcat를 반복.
*
*    CASE wa_fieldcat-fieldname.               " wa_fieldcat-fieldname 값에 따라
*      WHEN 'CARRNAME' OR 'CURRCODE' OR 'URL'. " 값이 키가 아닌 경우에 편집 가능
*        IF togl = ''.                         " togl 변수가 빈 값이면  (조회)
*          wa_fieldcat-edit  = 'X'.            " wa_fieldcat-edit 에 x를 넣는다.
*        ELSE.                                 " togl 변수에 값이 있으면 (편집)
*          wa_fieldcat-edit  = ''.             " wa_fieldcat-edit 에 빈 값을 넣는다.
*        ENDIF.
*      WHEN OTHERS.                            " 값이 키 값인 경우
*        wa_fieldcat-edit = ''.                " wa_fieldcat-edit 에 빈 값을 넣는다.
*    ENDCASE.
*
*    MODIFY p_it_fieldcat FROM wa_fieldcat.    " CHANGING으로 넘어온 it_fieldcat에 wa_fieldcat 로 수정하라.
*  ENDLOOP.
ENDFORM.
*&---------------------------------------------------------------------*
*& Form BUILD_FIELDCAT
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM build_fieldcat .
  DATA: lv_index TYPE int1.
  lv_index = 0.

  wa_fieldcat-fieldname = 'CARRID'.   " 필드이름
  wa_fieldcat-seltext_m = 'CARRID'.   " 컬럼제목
  wa_fieldcat-col_pos = lv_index.     " 0 (컬럼 위치 순서)
  wa_fieldcat-outputlen = 10.         " 넓이(넓이 최적화가 되면 알아서 맞춰지고, 편집모드에서는 입력받는 길이이다.)
  wa_fieldcat-key  = 'X'.             " 키
  wa_fieldcat-just = 'L'.             " 왼쪽 정렬

  APPEND wa_fieldcat TO it_fieldcat.  " 한 줄(wa_fieldcat) 테이블(it_fieldcat)에 넣기.
  CLEAR wa_fieldcat.
  " --------------------------------------------------------------------------------

  lv_index = lv_index + 1.            " 1증가

  wa_fieldcat-fieldname = 'CARRNAME'. " 필드이름
  wa_fieldcat-seltext_m = 'CARRNAME'. " 컬럼제목
  wa_fieldcat-col_pos = lv_index.     " 1 (컬럼 위치 순서)
  wa_fieldcat-outputlen = 20.         " 넓이
  wa_fieldcat-just = 'L'.
  wa_fieldcat-edit = 'X'.
  wa_fieldcat-lowercase = 'X'.        " 편집모드에서는 수정하고 저장하면 자동으로 대문자로 바꿔준다. 그래서 입력한 소문자 형태로 하려고

  APPEND wa_fieldcat TO it_fieldcat.
  CLEAR wa_fieldcat.
  " --------------------------------------------------------------------------------

  lv_index = lv_index + 1.
  wa_fieldcat-fieldname = 'CURRCODE'.
  wa_fieldcat-seltext_m = 'CURRCODE'.
  wa_fieldcat-col_pos = lv_index.
  wa_fieldcat-outputlen = 5.          " 수정1
*  wa_fieldcat-key = 'X'.
  wa_fieldcat-just = 'L'.
*wa_fieldcat-edit = 'X'.
  APPEND wa_fieldcat TO it_fieldcat.
  CLEAR wa_fieldcat.
  " --------------------------------------------------------------------------------

  lv_index = lv_index + 1.
  wa_fieldcat-fieldname = 'URL'.
  wa_fieldcat-seltext_m = 'URL'.
  wa_fieldcat-col_pos = lv_index.
  wa_fieldcat-outputlen = 255.       " 수정2
*  wa_fieldcat-key = 'X'.
  wa_fieldcat-just = 'L'.
*wa_fieldcat-edit = 'X'.
  wa_fieldcat-lowercase = 'X'.
  APPEND wa_fieldcat TO it_fieldcat.
  CLEAR wa_fieldcat.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form BUILD_LAYOUT
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM build_layout .
  ls_layout-zebra = 'X'.              " 줄무늬
  ls_layout-no_input = 'X'.           " 입력 못하게

  " 선언할 때 box 라는 필드를 만들었는데, box라는 이름을 구조안에 추가했을 때 행을 선택하면 자동으로 이 box 필드에 x가 들어간다.
  " 선택한 행에 대한 액션을 할 때. box1, box2 이렇게 선언하는 곳에서 고쳐서 사용가능.
  ls_layout-box_fieldname = 'box'.
  ls_layout-colwidth_optimize = 'X'. " 수정3
ENDFORM.
*&---------------------------------------------------------------------*
*& Form ALV_GRID
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*& -->  p1        text
*& <--  p2        text
*&---------------------------------------------------------------------*
FORM alv_grid .

  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
    EXPORTING
      i_callback_program       = sy-repid           " 현재 프로그램 ID
      i_callback_pf_status_set = 'PF_STATUS_SET'    " PF_STATUS_SET을 호출. 출력 전에 실행.
      i_callback_user_command  = 'USER_COMMAND'     " USER_COMMAND를 호출.  입력 후에 실행.
      is_layout                = ls_layout          " 표의 모양(색상, 줄무늬 등) 설정을 하는 객체
      it_fieldcat              = it_fieldcat        " 컬럼 정보(이름, 길이, 타입 등) 객체
    TABLES
      t_outtab                 = it_zscarr          " it_zscarr 인터널 테이블을 입력 파라미터로 넣는다.
* EXCEPTIONS
*     PROGRAM_ERROR            = 1
*     OTHERS                   = 2
    .
  IF sy-subrc <> 0.
* Implement suitable error handling here
  ENDIF.


ENDFORM.

```
