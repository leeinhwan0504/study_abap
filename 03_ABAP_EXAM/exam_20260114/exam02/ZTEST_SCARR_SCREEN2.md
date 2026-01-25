```abap
*&---------------------------------------------------------------------*
*& Report ZTEST_SCARR_SCREEN2
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ZTEST_SCARR_SCREEN2.

INCLUDE ZABAP_SCARR_TOP. " 데이터 선언
INCLUDE ZABAP_SCARR_SEL. " 조회창(라디오버튼 등)
INCLUDE ZABAP_SCARR_C01. " 클래스(더블클릭 이벤트)
INCLUDE ZABAP_SCARR_F01. " 실제 기능(Subroutine)
INCLUDE ZABAP_SCARR_O01. " PBO (화면 출력 전)
INCLUDE ZABAP_SCARR_I01. " PAI (화면 입력 후)



START-OF-SELECTION.
  PERFORM get_data.      " DB에서 데이터 읽기
  CALL SCREEN 100.       " 화면 100 호출

```
