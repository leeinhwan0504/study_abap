```abap
DATA: lt_emp TYPE TABLE OF zemp.

lt_emp = VALUE #(
  ( mandt = '001' empno = '2022070301' ename = '김민지' )
  ( mandt = '001' empno = '2022070302' ename = '이승우' )
  ( mandt = '001' empno = '2022070303' ename = '박지영' )
  ( mandt = '001' empno = '2022070304' ename = '송재호' )
  ( mandt = '001' empno = '2022070305' ename = '이하늘' )
  ( mandt = '001' empno = '2022010101' ename = '김순자' )
  ( mandt = '001' empno = '2022121501' ename = '엄상준' )
  ( mandt = '001' empno = '2022121502' ename = '김소연' )
  ( mandt = '001' empno = '2023020301' ename = '박준희' )
  ( mandt = '001' empno = '2023020302' ename = '최현우' )
  ( mandt = '001' empno = '2023070101' ename = '박지호' )
  ( mandt = '001' empno = '2023070102' ename = '안서현' )
  ( mandt = '001' empno = '2023070103' ename = '한은진' )
  ( mandt = '001' empno = '2023102301' ename = '여신입' )
).

" 3. 데이터베이스 테이블에 입력
" MODIFY를 사용하면 데이터가 없을 땐 INSERT, 있을 땐 UPDATE(덮어쓰기)가 수행되어 더 안전합니다.
MODIFY zemp FROM TABLE lt_emp.

" 4. 처리 결과 출력
IF sy-subrc = 0.
  WRITE: / '총', lines( lt_emp ), '건의 사원 정보가 성공적으로 반영되었습니다.'.
ELSE.
  WRITE: / '데이터 입력 중 오류가 발생했습니다.'.
ENDIF.
```






```abap
DATA: lt_project TYPE TABLE OF zproject.

lt_project = VALUE #(
  ( mandt = '001' project_id = 'PRJ0000001' project_name = '한국전자통신 SAP구축' )
  ( mandt = '001' project_id = 'PRJ0000002' project_name = '대양무역서비스 SAP구축' )
  ( mandt = '001' project_id = 'PRJ0000003' project_name = '한양화학사업 SAP구축' )
  ( mandt = '001' project_id = 'PRJ0000004' project_name = '한국건설기술 SAP유지보수' )
  ( mandt = '001' project_id = 'PRJ0000005' project_name = '국가자동차그룹 SAP유지보수' )
  ( mandt = '001' project_id = 'PRJ0000006' project_name = '코리아식품산업 SAP모듈구축' )
  ( mandt = '001' project_id = 'PRJ0000007' project_name = '대한에너지솔루션 SAP모듈구축' )
  ( mandt = '001' project_id = 'PRJ0000008' project_name = '큰백화점그룹 SAP구축' )
  ( mandt = '001' project_id = 'PRJ0000009' project_name = '사랑헬스케어 SAP구축' )
  ( mandt = '001' project_id = 'PRJ0000010' project_name = '국민전기산업 SAP구축1차' )
  ( mandt = '001' project_id = 'PRJ0000011' project_name = '국민전기산업 SAP구축2차' )
).

" 3. 데이터베이스 테이블에 입력 (기존 데이터가 있으면 Update, 없으면 Insert)
MODIFY zproject FROM TABLE lt_project.

" 4. 처리 결과 확인
IF sy-subrc = 0.
  COMMIT WORK. " 확정
  WRITE: / '성공: ', lines( lt_project ), '건의 프로젝트 정보가 저장되었습니다.'.
ELSE.
  ROLLBACK WORK. " 취소
  WRITE: / '오류: 데이터 저장 중 문제가 발생했습니다.'.
ENDIF.
```
























```abap
DATA: lt_module TYPE TABLE OF zmodule.

lt_module = VALUE #(
  ( mandt = '001' module_name = 'CO' )
  ( mandt = '001' module_name = 'FI' )
  ( mandt = '001' module_name = 'HR' )
  ( mandt = '001' module_name = 'MM' )
  ( mandt = '001' module_name = 'SD' )
).

" 3. 데이터베이스 테이블에 입력 (기존 데이터가 있으면 갱신, 없으면 삽입)
MODIFY zmodule FROM TABLE lt_module.

" 4. 처리 결과 출력
IF sy-subrc = 0.
  COMMIT WORK.
  WRITE: / '성공: ', lines( lt_module ), '건의 모듈 정보가 성공적으로 반영되었습니다.'.
ELSE.
  ROLLBACK WORK.
  WRITE: / '오류: 데이터 입력 중 문제가 발생했습니다.'.
ENDIF.
```






```abap
DATA: lt_license TYPE TABLE OF zlicense.

lt_license = VALUE #(
  ( mandt = '001' license_id = 'C001' license_name = 'SAP CERTIFIED ABAP DEVELOPER' )
  ( mandt = '001' license_id = 'C002' license_name = 'SAP CERTIFIED HANA TECHNOLOGY' )
  ( mandt = '001' license_id = 'C003' license_name = 'SAP CERTIFIED FOR DEVELOPER' )
  ( mandt = '001' license_id = 'C004' license_name = 'SAP CERTIFIED US DEVELOPER' )
  ( mandt = '001' license_id = 'C005' license_name = 'SAP S/4HANA DEVELOPMENT' )
  ( mandt = '001' license_id = 'C006' license_name = 'SAP SOLUTION ARCHITECT' )
  ( mandt = '001' license_id = 'C007' license_name = 'SAP S/S INTEGRATION PRO' )
  ( mandt = '001' license_id = 'C008' license_name = 'SAP BUSINESS ANALYTICS PRO' )
  ( mandt = '001' license_id = 'C009' license_name = 'SAP DATA MIGRATION EXPERT' )
  ( mandt = '001' license_id = 'C010' license_name = 'SAP CLOUD MIGRATION EXPERT' )
).

" 3. 데이터베이스 테이블에 일괄 반영 (Insert or Update)
MODIFY zlicense FROM TABLE lt_license.

" 4. 처리 결과 출력
IF sy-subrc = 0.
  COMMIT WORK.
  WRITE: / '성공: ', lines( lt_license ), '건의 자격증 마스터 정보가 저장되었습니다.'.
ELSE.
  ROLLBACK WORK.
  WRITE: / '오류: 자격증 데이터 저장 중 문제가 발생했습니다.'.
ENDIF.
```



```abap
DATA: lt_lic_reg TYPE TABLE OF zlicense_reg.

" ACQ_DATE의 경우 SAP 표준 DATS 타입(YYYYMMDD) 형식을 따릅니다.
lt_lic_reg = VALUE #(
  ( mandt = '001' empno = '2022070301' license_id = 'C007' acq_date = '20231215' )
  ( mandt = '001' empno = '2022070302' license_id = 'C008' acq_date = '20231215' )
  ( mandt = '001' empno = '2022070303' license_id = 'C009' acq_date = '20231215' )
  ( mandt = '001' empno = '2022070304' license_id = 'C010' acq_date = '20231215' )
  ( mandt = '001' empno = '2023020301' license_id = 'C001' acq_date = '20231215' )
  ( mandt = '001' empno = '2023020302' license_id = 'C003' acq_date = '20231215' )
  ( mandt = '001' empno = '2023102301' license_id = 'C006' acq_date = '20231215' )
).

" 3. 데이터베이스 테이블에 입력 (기존 데이터가 있으면 갱신, 없으면 삽입)
MODIFY zlicense_reg FROM TABLE lt_lic_reg.

" 4. 처리 결과 확인 및 단순 출력
IF sy-subrc = 0.
  COMMIT WORK.
  " 입력된 데이터를 다시 읽어와서 출력창에 표시
  cl_demo_output=>display( lt_lic_reg ).
ELSE.
  ROLLBACK WORK.
  MESSAGE '데이터 저장 중 오류가 발생했습니다.' TYPE 'E'.
ENDIF.
```

```abap
DATA: lt_history TYPE TABLE OF zhistory.

" 2. 데이터 구성 (New ABAP Syntax 활용)
" START_DATE와 END_DATE는 요청하신 대로 YYYYMM 6자리 형식으로 입력합니다.
lt_history = VALUE #(
  ( mandt = '001' empno = '2022010101' project_id = 'PRJ0000001' module_name = 'HR' start_date = '202312' end_date = '202412' )
  ( mandt = '001' empno = '2022010101' project_id = 'PRJ0000005' module_name = 'SD' start_date = '202312' end_date = '202412' )
  ( mandt = '001' empno = '2023070102' project_id = 'PRJ0000006' module_name = 'HR' start_date = '202312' end_date = '202412' )
  ( mandt = '001' empno = '2023102301' project_id = 'PRJ0000008' module_name = 'MM' start_date = '202312' end_date = '202412' )
).

" 3. 데이터베이스 테이블에 반영 (Insert or Update)
MODIFY zhistory FROM TABLE lt_history.

" 4. 처리 결과 확인 및 출력
IF sy-subrc = 0.
  COMMIT WORK.
  " cl_demo_output을 사용하여 결과를 팝업창으로 표시
  cl_demo_output=>display( lt_history ).
ELSE.
  ROLLBACK WORK.
  MESSAGE '데이터 저장 중 오류가 발생했습니다.' TYPE 'E'.
ENDIF.
```