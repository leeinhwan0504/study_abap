
***조회***
```abap
TABLES: ZLICENSE_REG.


TYPES: BEGIN OF ts_test,
        mandt TYPE zlicense_reg-mandt,
        empno TYPE zlicense_reg-empno,
        license_id TYPE zlicense_reg-license_id,
        acq_date TYPE zlicense_reg-acq_date,
       END OF ts_test.

DATA: gs_test TYPE TABLE OF ts_test.

SELECT  mandt, empno, license_id, acq_date
  FROM  zlicense_reg
  INTO  TABLE @gs_test.

cl_demo_output=>display( gs_test ).
```

***삭제***
```abap
TABLES: ZLICENSE_REG.

DELETE FROM zlicense_reg.

IF sy-subrc = 0.
  COMMIT WORK. " 변경 사항을 DB에 확정 반영합니다.
  WRITE: '모든 데이터가 성공적으로 삭제되었습니다.'.
ELSE.
  WRITE: '삭제할 데이터가 없거나 오류가 발생했습니다.'.
ENDIF.
```

***삭제***
```abap
TABLES: ZLICENSE_REG.

DELETE FROM zlicense_reg.

IF sy-subrc = 0.
  COMMIT WORK. " 변경 사항을 DB에 확정 반영합니다.
  WRITE: '모든 데이터가 성공적으로 삭제되었습니다.'.
ELSE.
  WRITE: '삭제할 데이터가 없거나 오류가 발생했습니다.'.
ENDIF.
```

***삭제***
```abap
TABLES: ZHISTORY.

DELETE FROM ZHISTORY.

IF sy-subrc = 0.
  COMMIT WORK. " 변경 사항을 DB에 확정 반영합니다.
  WRITE: '모든 데이터가 성공적으로 삭제되었습니다.'.
ELSE.
  WRITE: '삭제할 데이터가 없거나 오류가 발생했습니다.'.
ENDIF.
```

***삭제***
```abap
TABLES: ZLICENSE.

DELETE FROM ZLICENSE.

IF sy-subrc = 0.
  COMMIT WORK. " 변경 사항을 DB에 확정 반영합니다.
  WRITE: '모든 데이터가 성공적으로 삭제되었습니다.'.
ELSE.
  WRITE: '삭제할 데이터가 없거나 오류가 발생했습니다.'.
ENDIF.
```

***삭제***
```abap
TABLES: ZMODULE.

DELETE FROM ZMODULE.

IF sy-subrc = 0.
  COMMIT WORK. " 변경 사항을 DB에 확정 반영합니다.
  WRITE: '모든 데이터가 성공적으로 삭제되었습니다.'.
ELSE.
  WRITE: '삭제할 데이터가 없거나 오류가 발생했습니다.'.
ENDIF.
```

***삭제***
```abap
TABLES: ZPROJECT.

DELETE FROM ZPROJECT.

IF sy-subrc = 0.
  COMMIT WORK. " 변경 사항을 DB에 확정 반영합니다.
  WRITE: '모든 데이터가 성공적으로 삭제되었습니다.'.
ELSE.
  WRITE: '삭제할 데이터가 없거나 오류가 발생했습니다.'.
ENDIF.
```


***삭제***
```abap
TABLES: ZEMP.

DELETE FROM ZEMP.

IF sy-subrc = 0.
  COMMIT WORK. " 변경 사항을 DB에 확정 반영합니다.
  WRITE: '모든 데이터가 성공적으로 삭제되었습니다.'.
ELSE.
  WRITE: '삭제할 데이터가 없거나 오류가 발생했습니다.'.
ENDIF.
```







