# SAP-ABAP-Project-Sales-Order-Report-using-ALV-Grid
This project involves creating an ABAP program that retrieves sales order details (like order number, customer, material, quantity, and price) from SAP tables (e.g., VBAK, VBAP, and MARA) and displays the data in an ALV grid.

REPORT zsales_order_report.

" Selection screen
PARAMETERS: p_vbeln TYPE vbak-vbeln OBLIGATORY,
            p_kunnr TYPE vbak-kunnr.

DATA: lt_vbak TYPE TABLE OF vbak,
      lt_vbap TYPE TABLE OF vbap,
      lt_mara TYPE TABLE OF mara,
      lt_final TYPE TABLE OF zfinal_data,
      gr_alv_table TYPE REF TO cl_salv_table.

" Fetch Sales Order Header (VBAK)
SELECT * FROM vbak INTO TABLE lt_vbak
  WHERE vbeln = p_vbeln AND kunnr = p_kunnr.

" Fetch Sales Order Items (VBAP)
SELECT * FROM vbap INTO TABLE lt_vbap
  FOR ALL ENTRIES IN lt_vbak
  WHERE vbeln = lt_vbak-vbeln.

" Fetch Material Data (MARA)
SELECT * FROM mara INTO TABLE lt_mara
  FOR ALL ENTRIES IN lt_vbap
  WHERE matnr = lt_vbap-matnr.

" Create final structure
TYPES: BEGIN OF zfinal_data,
         vbeln  TYPE vbak-vbeln,
         kunnr  TYPE vbak-kunnr,
         matnr  TYPE vbap-matnr,
         maktx  TYPE mara-maktx,
         kwmeng TYPE vbap-kwmeng,
         netpr  TYPE vbap-netpr,
       END OF zfinal_data.

" Loop through data and fill final internal table
LOOP AT lt_vbak INTO DATA(ls_vbak).
  LOOP AT lt_vbap INTO DATA(ls_vbap) WHERE vbeln = ls_vbak-vbeln.
    READ TABLE lt_mara INTO DATA(ls_mara) WITH KEY matnr = ls_vbap-matnr.
    IF sy-subrc = 0.
      DATA(ls_final) = VALUE zfinal_data(
                            vbeln  = ls_vbak-vbeln
                            kunnr  = ls_vbak-kunnr
                            matnr  = ls_vbap-matnr
                            maktx  = ls_mara-maktx
                            kwmeng = ls_vbap-kwmeng
                            netpr  = ls_vbap-netpr ).
      APPEND ls_final TO lt_final.
    ENDIF.
  ENDLOOP.
ENDLOOP.

" Display data using ALV Grid
cl_salv_table=>factory(
  IMPORTING r_salv_table = gr_alv_table
  CHANGING  t_table      = lt_final ).

gr_alv_table->display( ).
