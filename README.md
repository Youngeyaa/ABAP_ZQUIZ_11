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

```


  CL_DEMO_OUTPUT=>DISPLAY( GT_AVGR ).
