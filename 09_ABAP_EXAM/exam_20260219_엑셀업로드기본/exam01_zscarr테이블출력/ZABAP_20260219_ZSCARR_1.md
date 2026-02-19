```abap
*&---------------------------------------------------------------------*
*& Report ZABAP_20260219_ZSCARR_1
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

REPORT ZABAP_20260219_ZSCARR_1.

INCLUDE ZABAP_20260219_ZSCARR_1_TOP. " 데이터 선언
INCLUDE ZABAP_20260219_ZSCARR_1_SEL. " 조회창(라디오버튼 등)
INCLUDE ZABAP_20260219_ZSCARR_1_C01. " 클래스(더블클릭 이벤트)
INCLUDE ZABAP_20260219_ZSCARR_1_F01. " 실제 기능(Subroutine)
INCLUDE ZABAP_20260219_ZSCARR_1_O01. " PBO (화면 출력 전)
INCLUDE ZABAP_20260219_ZSCARR_1_I01. " PAI (화면 입력 후)


START-OF-SELECTION.
  PERFORM get_data.
  CALL SCREEN 100.
```
