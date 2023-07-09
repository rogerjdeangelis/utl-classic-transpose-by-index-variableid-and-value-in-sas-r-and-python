# utl-classic-transpose-by-index-variableid-and-value-in-sas-r-and-python
Classic transpose by index variableid and value in sas r and python    
    %let pgm=utl-classic-transpose-by-index-variableid-and-value-in-sas-r-and-python;

    github
    https://tinyurl.com/39bku4dx
    https://github.com/rogerjdeangelis/utl-classic-transpose-by-index-variableid-and-value-in-sas-r-and-python

    Classic transpose by index variableid and value in sas r and python

           Solutions

               1. SAS
               2, R
               3. Python
               4. SAS SQL (not as slow as you suspect. Also exaple of code generator.)
               5. WPS SQL
                  (note no change to macros %do_over and array - substitutes double quotes for singles)

    SOAPBOX ON

      Except for python SQL(very slow),it is difficult to leverage knowledge of the python syntax to other languages
      R syntax is fairly easy to map to other "matrix" languages like matlab and sas IML.

    SOAPBOX OFF

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */
    options validvarname=upcase;
    libname sd1 "d:/sd1";

    data sd1.have;
     input (time depth value) ($);
    cards4;
    t1 d1 v1
    t1 d2 v2
    t1 d3 v3
    t2 d1 v1
    t2 d2 v2
    t2 d3 v3
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* SD1.HAVE total obs=6 08MAY2023:10:44:34                                                                                */
    /*                                                                                                                        */
    /* Obs    TIME    DEPTH    VALUE                                                                                          */
    /*                                                                                                                        */
    /*  1      t1      d1       v1                                                                                            */
    /*  2      t1      d2       v2                                                                                            */
    /*  3      t1      d3       v3                                                                                            */
    /*  4      t2      d1       v1                                                                                            */
    /*  5      t2      d2       v2                                                                                            */
    /*  6      t2      d3       v3                                                                                            */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  WORK.WANT total obs=2 08MAY2023:10:46:00                                                                              */
    /*                                                                                                                        */
    /*  Obs    TIME    _NAME_    D1    D2    D3                                                                               */
    /*                                                                                                                        */
    /*   1      t1     VALUE     v1    v2    v3                                                                               */
    /*   2      t2     VALUE     v1    v2    v3                                                                               */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*
    / |  ___  __ _ ___
    | | / __|/ _` / __|
    | | \__ \ (_| \__ \
    |_| |___/\__,_|___/

    */
    libname sd1 "d:/sd1";

    proc datasets lib=sd1 nodetails nolist;
     delete want;
    run;quit;

    proc transpose data=sd1.have out=want;
      by time;
      id depth;
      var value;
    run;quit;

    /*___
    |___ \   _ __
      __) | | `__|
     / __/  | |
    |_____| |_|

    */
    libname sd1 "d:/sd1";

    proc datasets lib=sd1 nodetails nolist;
     delete want;
    run;quit;

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    library(tidyr);
    want <- pivot_wider(have,id_cols = TIME, names_from = DEPTH, values_from = VALUE);
    want;
    endsubmit;
    import data=sd1.want r=want;
    run;quit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  The WPS System (proc r)                                                                                               */
    /*                                                                                                                        */
    /*  # A tibble: 2 x 4                                                                                                     */
    /*    TIME  d1    d2    d3                                                                                                */
    /*    <chr> <chr> <chr> <chr>                                                                                             */
    /*                                                                                                                        */
    /*  1 t1    v1    v2    v3                                                                                                */
    /*  2 t2    v1    v2    v3                                                                                                */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____               _   _
    |___ /   _ __  _   _| |_| |__   ___  _ __
      |_ \  | `_ \| | | | __| `_ \ / _ \| `_ \
     ___) | | |_) | |_| | |_| | | | (_) | | | |
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_|
            |_|    |___/
    */

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc python;
    export data=sd1.have python=have;
    submit;
    want_py = have.pivot(index="TIME", columns="DEPTH", values="VALUE");
    print(want_py);
    endsubmit;
    run;quit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* The PYTHON Procedure                                                                                                   */
    /*                                                                                                                        */
    /* DEPTH     d1        d2        d3                                                                                       */
    /* TIME                                                                                                                   */
    /* t1        v1        v2        v3                                                                                       */
    /* t2        v1        v2        v3                                                                                       */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _                               _
    | || |    ___  __ _ ___   ___  __ _| |
    | || |_  / __|/ _` / __| / __|/ _` | |
    |__   _| \__ \ (_| \__ \ \__ \ (_| | |
       |_|   |___/\__,_|___/ |___/\__, |_|
                                     |_|
    */
    libname sd1 "d:/sd1";

    proc datasets lib=sd1 nodetails nolist;
     delete want_sql;
    run;quit;

    proc sql;

      select distinct depth into :_1- from sd1.have

    ;quit;

    %let _n=&sqlobs;

    proc sql;

      create
         table sd1.want_sql as
      select
        time
        %do_over(_,phrase=%str(
          ,max(case when depth="?"  then value else "" end) as ?))
      from
        sd1.have
      group
        by time
    ;quit;

    proc print data=sd1.want_sql;
    run;quit;

    /*---- Display macro variables                                           ----*/
    %put &_1;
    %put &_2;
    %put &_n;

    %arraydelete(_);

    /*----  Check macro vars  are deleted                                    ----*/
    %put &_1;
    %put &_2;
    %put &_n;

    /*                                _
      __ _  ___ _ __     ___ ___   __| | ___
     / _` |/ _ \ `_ \   / __/ _ \ / _` |/ _ \
    | (_| |  __/ | | | | (_| (_) | (_| |  __/
     \__, |\___|_| |_|  \___\___/ \__,_|\___|
     |___/
    */

    data _null_;
      put
       %do_over(_,phrase=%str(",max(case when depth='?'  then value else '' end) as ?" / ))
     ;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* ,max(case when depth='d1'  then value else '' end) as d1                                                               */
    /* ,max(case when depth='d2'  then value else '' end) as d2                                                               */
    /* ,max(case when depth='d3'  then value else '' end) as d3                                                               */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                                   _
    | ___|  __      ___ __  ___   ___  __ _| |
    |___ \  \ \ /\ / / `_ \/ __| / __|/ _` | |
     ___) |  \ V  V /| |_) \__ \ \__ \ (_| | |
    |____/    \_/\_/ | .__/|___/ |___/\__, |_|
                     |_|                 |_|
    */

    proc datasets lib=sd1 nodetails nolist;
     delete want_wps;
    run;quit;

    %utl_submit_wps64('

    libname sd1 "d:/sd1";

    options validvarname=any;

    proc sql;

      select        distinct depth  into :_1- from sd1.have;
      select  count(distinct depth) into :_n  from sd1.have;

    ;quit;

    proc sql;

      create
         table sd1.want_wps as
      select
        time
        %do_over(_,phrase=%str(
         ,max(case when depth="?"  then value else "" end) as ?))
      from
        sd1.have
      group
        by time
    ;quit;

    proc print data=sd1.want_wps;
    run;quit;

    ');

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System  (SAME IN SAS)                                                                                          */
    /*                                                                                                                        */
    /* Obs    TIME    d1    d2    d3                                                                                          */
    /*                                                                                                                        */
    /*  1      t1     v1    v2    v3                                                                                          */
    /*  2      t2     v1    v2    v3                                                                                          */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
