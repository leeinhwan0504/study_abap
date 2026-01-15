
```abap
TYPES: BEGIN OF ty_result,
         empno        TYPE zemp-empno,
         ename        TYPE zemp-ename,
         project_id   TYPE zproject-project_id,
         project_name TYPE zproject-project_name,
         module_name  TYPE zmodule-module_name,
         start_date   TYPE zhistory-start_date,
         end_date     TYPE zhistory-end_date,
         remark       TYPE string,
       END OF ty_result.

DATA: lt_result TYPE TABLE OF ty_result,
      ls_result TYPE ty_result.

SELECT h~empno,
       e~ename,
       h~project_id,
       p~project_name,
       h~module_name,
       h~start_date,
       h~end_date
  FROM zhistory AS h
  LEFT OUTER JOIN zemp     AS e ON e~empno       = h~empno
  LEFT OUTER JOIN zproject AS p ON p~project_id  = h~project_id
  INTO TABLE @DATA(lt_raw).

" 3. 비고(Remark) 로직 처리 및 결과 테이블 구성
LOOP AT lt_raw INTO DATA(ls_raw).
  MOVE-CORRESPONDING ls_raw TO ls_result.

  " 사원/프로젝트 존재 여부에 따른 단순 비고 처리
  IF ls_result-ename IS INITIAL OR ls_result-project_name IS INITIAL.
    ls_result-remark = '데이터 오류 확인 필요'.
  ELSE.
    ls_result-remark = '저장 가능'.
  ENDIF.

  APPEND ls_result TO lt_result.
ENDLOOP.

" 4. cl_demo_output을 이용한 단순 출력
cl_demo_output=>display( lt_result ).
```
