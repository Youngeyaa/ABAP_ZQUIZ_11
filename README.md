# ABAP_ZQUIZ_11 (still editing)

https://community.sap.com/t5/application-development-and-automation-discussions/average-calculation-for-different-rows-in-one-internal-table/m-p/737089

```ABAP
REPORT ZQUIZ_11_G20.
*
*"예시 결과
*"CARRID | CARRNAME | CONNID | DAYS_AHEAD
*
*"Selection Screen : Airline / Flight Date ( Text Elements 수정)
*"항공사, 항공편에 대한 출발 몇일전에 예약 했는지에대한 평균일.
*"항공사 이름은 scarr table 에서 취득
*"모든 데이터를 인터널 테이블에 관리하여, 필요한 곳에서 Read table 을 사ㅏ용하여 항공사 이름 취득.
*"평균 예약일 : SBOOK 테이블에서 출발일(FLDATE), 예약일 (ORDER_DATE0 에서 비행기 출발 몇일전에 예약했는지 계산해서 항공사 항공편 별로
*"평균 예약일 계산. 예약 취소건은 제외. (Cancelled = x)
*
TABLES: SCARR, SBOOK.

TYPES: BEGIN OF TS_RESV,
         CARRID     TYPE SBOOK-CARRID,
         CARRNAME   TYPE SCARR-CARRNAME,
         CONNID     TYPE SBOOK-CONNID,
*         DAYS_AHEAD TYPE F,
         FLDATE     TYPE SBOOK-FLDATE,
         ORDER_DATE TYPE SBOOK-ORDER_DATE,
         DAYS_AHEAD TYPE P LENGTH 6 DECIMALS 1,
       END OF TS_RESV.

"-----------평균 계산용 structure 추가-------------------------------------------------
TYPES: BEGIN OF TS_AVG,
         CARRID     TYPE SBOOK-CARRID,
         CARRNAME   TYPE SCARR-CARRNAME,
         CONNID     TYPE SBOOK-CONNID,
         DAYS_AHEAD TYPE P LENGTH 6 DECIMALS 1,
       END OF TS_AVG.
"------------------------------------------------------------

DATA: GT_AVGR  TYPE TABLE OF TS_RESV,
      GS_AVGR  LIKE LINE OF GT_AVGR,
      GT_AVGGR TYPE TABLE OF TS_AVG,
      GS_AVGGR LIKE LINE OF GT_AVGGR.

SELECT-OPTIONS: SO_AIR FOR SCARR-CARRID,
                SO_FLD FOR SBOOK-FLDATE.

START-OF-SELECTION.

  SELECT B~CARRID, A~CARRNAME, B~CONNID, B~FLDATE, B~ORDER_DATE
    FROM SCARR AS A INNER JOIN SBOOK AS B
    ON A~CARRID = B~CARRID

  WHERE B~CANCELLED <> 'X'
    AND B~CARRID IN @SO_AIR
    AND B~FLDATE IN @SO_FLD

  GROUP BY
     B~CARRID, A~CARRNAME, B~CONNID, B~FLDATE, B~ORDER_DATE
   INTO CORRESPONDING FIELDS OF TABLE @GT_AVGR.

  LOOP AT GT_AVGR INTO GS_AVGR.
    GS_AVGR-DAYS_AHEAD = GS_AVGR-FLDATE - GS_AVGR-ORDER_DATE.
    MODIFY GT_AVGR FROM GS_AVGR.
  ENDLOOP.

DATA GT_AVGR TYPE 


  CL_DEMO_OUTPUT=>DISPLAY( GT_AVGR ).

```


```ABAP

*&---------------------------------------------------------------------*
*& Report ZQUIZ_12_G21
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT zquiz_12_g21.

DATA : gv_year TYPE numc4.

TYPES : BEGIN OF ty_data,
          carrid   TYPE sflight-carrid,
          carrname TYPE scarr-carrname,
          currcode TYPE scarr-currcode,
          Jan      TYPE sflight-paymentsum,
          Feb      TYPE sflight-paymentsum,
          Mar      TYPE sflight-paymentsum,
          Apr      TYPE sflight-paymentsum,
          May      TYPE sflight-paymentsum,
          Jun      TYPE sflight-paymentsum,
          Jul      TYPE sflight-paymentsum,
          Aug      TYPE sflight-paymentsum,
          Sep      TYPE sflight-paymentsum,
          Oct      TYPE sflight-paymentsum,
          Nov      TYPE sflight-paymentsum,
          Dec      TYPE sflight-paymentsum,
        END OF ty_data.

TYPES : BEGIN OF ty_list,
          carrid     TYPE sflight-carrid,
          carrname   TYPE scarr-carrname,
          currcode   TYPE scarr-currcode,
          paymentsum TYPE sflight-paymentsum,
          fldate     TYPE sflight-fldate,
        END OF ty_list.

DATA : gs_list TYPE ty_list,
       gt_list TYPE TABLE OF ty_list.

DATA : gs_data TYPE ty_data,
       gt_data TYPE TABLE OF ty_data.

SELECT-OPTIONS : so_air FOR gs_list-carrid,
                 so_year FOR gv_year DEFAULT 2025.

*---------------------------------
*---------------------------------
*---------------------------------

AT SELECTION-SCREEN.

START-OF-SELECTION.
  IF so_year-low IS NOT INITIAL AND so_year-high IS INITIAL.
    so_year-high = so_year-low && '1231'.
    so_year-low = so_year-low && '0101'.
  ENDIF.

  SELECT sf~carrid sc~carrname sc~currcode sf~paymentsum sf~fldate
    FROM sflight AS sf INNER JOIN scarr AS sc ON sf~carrid = sc~carrid
    INTO CORRESPONDING FIELDS OF gs_list
    WHERE sf~carrid IN so_air
      AND sf~fldate BETWEEN so_year-low AND so_year-high.

    CLEAR gs_data.
    MOVE-CORRESPONDING gs_list TO gs_data.

    IF gs_list-fldate+4(2) = '01'.
      gs_data-jan = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '02'.
      gs_data-feb = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '03'.
      gs_data-mar = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '04'.
      gs_data-apr = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '05'.
      gs_data-may = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '06'.
      gs_data-jun = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '07'.
      gs_data-jul = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '08'.
      gs_data-aug = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '09'.
      gs_data-sep = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '10'.
      gs_data-oct = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '11'.
      gs_data-nov = gs_list-paymentsum.
    ELSEIF gs_list-fldate+4(2) = '12'.
      gs_data-dec = gs_list-paymentsum.
    ENDIF.

    COLLECT gs_data INTO gt_data.

  ENDSELECT.

  cl_demo_output=>display( gt_data ).

```
