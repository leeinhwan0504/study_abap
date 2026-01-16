```abap
*&---------------------------------------------------------------------*
*& Include          ZABAP_SCARR_SCREEN5_SEL
*&---------------------------------------------------------------------*




SELECTION-SCREEN BEGIN OF BLOCK bl01 WITH FRAME TITLE TEXT-001.
  PARAMETERS: p_file TYPE rlgrap-filename DEFAULT 'C:\' OBLIGATORY.
SELECTION-SCREEN END OF BLOCK bl01.
```