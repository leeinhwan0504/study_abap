```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_20260219_ZSCARR_2_SEL
*&---------------------------------------------------------------------*

SELECTION-SCREEN BEGIN OF BLOCK bl01 WITH FRAME TITLE TEXT-001.
  PARAMETERS: p_file TYPE rlgrap-filename DEFAULT 'C:\' OBLIGATORY.
SELECTION-SCREEN END OF BLOCK bl01.
```