# utl-simple-classic-transpose-pivot-wider-in-native-and-sql-wps-r-python
    %let pgm=utl-simple-classic-transpose-pivot-wider-in-native-and-sql-wps-r-python;

    Simple classic transpose pivot wider in native and sql wps r python

      SIX SOLUTIONS

           1 wps sql
           2 wps r sql
           3 wps python sql
           4 wps no sql
           5 wps r no sql
           6 wps python no sql (it seems that python needs a column with the names for columns)
           7 loosely related repos

    stackoverflow R
    http://tinyurl.com/2s4d4w4s
    https://stackoverflow.com/questions/77806960/how-do-i-condense-a-data-frame-to-be-wider-while-maintaining-row-inputs-as-row-i

    /**************************************************************************************************************************/
    /*                       |                                                                                                */
    /*     INPUT             |                  PROCESSES                                  |           OUTPUT                 */
    /*                       |                                                             |                                  */
    /* SD1.HAVE total obs=8  | WPS SQL (use SQL arrays to si[lify?)                        |   NAME     ID1    ID2    ID3     */
    /*                       | ======                                                      |                                  */
    /* Obs     NAME     ID   | select                                                      |  Cecili     24     12      .     */
    /*                       |   name                                                      |  Mayana     13     24      .     */
    /*  1     Mayana    13   |  ,max(case when partition=1 then id else . end) as id1      |  Sierra     56      .      .     */
    /*  2     Mayana    24   |  ,max(case when partition=2 then id else . end) as id2      |  Sophia     54     12     15     */
    /*  3     Sierra    56   |  ,max(case when partition=3 then id else . end) as id3      |                                  */
    /*  4     Sophia    54   | from                                                        |                                  */
    /*  5     Sophia    12   |   %sqlpartition(sd1.have,by=name)                           |                                  */
    /*  6     Sophia    15   | group                                                       |                                  */
    /*  7     Cecili    24   |   by name                                                   |                                  */
    /*  8     Cecili    12   |                                                             |                                  */
    /*                       | WPS R and PYTHON SQL (use SQL arrays to si[lify?)           |                                  */
    /*                       | ====================                                        |                                  */
    /*                       | select                                                      |                                  */
    /*                       |  name                                                       |                                  */
    /*                       |  ,max(case when partition=1 then id else NULL end) as id1   |                                  */
    /*                       |  ,max(case when partition=2 then id else NULL end) as id2   |                                  */
    /*                       |  ,max(case when partition=3 then id else NULL end) as id3   |                                  */
    /*                       | from                                                        |                                  */
    /*                       |  (select name, id, row_number() OVER                        |                                  */
    /*                       |    (PARTITION BY name) as partition from have )             |                                  */
    /*                       | group                                                       |                                  */
    /*                       |  by name                                                    |                                  */
    /*                       |                                                             |                                  */
    /*                       | WPS NO SQL                                                  |                                  */
    /*                       | ====================                                        |                                  */
    /*                       | proc transpose data=sd1.have out=sd1.want prefix=ID;        |                                  */
    /*                       | by name notsorted;                                          |                                  */
    /*                       | run;quit;                                                   |                                  */
    /*                       |                                                             |                                  */
    /*                       | WPS R NO SQL                                                |                                  */
    /*                       | ============                                                |                                  */
    /*                       | have |>                                                     |                                  */
    /*                       |   mutate(idno = row_number(), .by = NAME) |>                |                                  */
    /*                       |   pivot_wider(                                              |                                  */
    /*                       |     values_from = ID                                        |                                  */
    /*                       |    ,names_from = idno                                       |                                  */
    /*                       |    ,values_fill = NaN                                       |                                  */
    /*                       |    ,names_prefix = 'ID'                                     |                                  */
    /*                       |   );                                                        |                                  */
    /*                       |                                                             |                                  */
    /*                       | WPS PYTHON NO SQL (need to add column name variable?)       |                                  */
    /*                       | =================                                           |                                  */
    /*                       |  df = pdsql('''                                             |                                  */
    /*                       |    select                                                   |                                  */
    /*                       |       name                                                  |                                  */
    /*                       |      ,id                                                    |                                  */
    /*                       |      ,partition                                             |                                  */
    /*                       |    from                                                     |                                  */
    /*                       |       (select name, id, row_number() OVER (PARTITION        |                                  */
    /*                       |         BY name) as partition from have )                   |                                  */
    /*                       |  ''');                                                      |                                  */
    /*                       |  want = df.pivot(index='name', columns='partition',         |                                  */
    /*                       |   values='id');                                             |                                  */
    /*                       |   );                                                        |                                  */
    /**************************************************************************************************************************/

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
      input Name $ id;
    cards4;
    Mayana 13
    Mayana 24
    Sierra 56
    Sophia 54
    Sophia 12
    Sophia 15
    Cecili 24
    Cecili 12
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* SD1.HAVE total obs=8                                                                                                   */
    /*                                                                                                                        */
    /* Obs     NAME     ID                                                                                                    */
    /*                                                                                                                        */
    /*  1     Mayana    13                                                                                                    */
    /*  2     Mayana    24                                                                                                    */
    /*  3     Sierra    56                                                                                                    */
    /*  4     Sophia    54                                                                                                    */
    /*  5     Sophia    12                                                                                                    */
    /*  6     Sophia    15                                                                                                    */
    /*  7     Cecili    24                                                                                                    */
    /*  8     Cecili    12                                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*                                  _
    / | __      ___ __  ___   ___  __ _| |
    | | \ \ /\ / / `_ \/ __| / __|/ _` | |
    | |  \ V  V /| |_) \__ \ \__ \ (_| | |
    |_|   \_/\_/ | .__/|___/ |___/\__, |_|
                 |_|                 |_|
    */

    /*----                                                                   ----*/
    /*----   you can use sql arrays for the repetitive code                  ----*/
    /*----   turns the repetive code to a one liner                          ----*/
    /*----

    proc datasets lib=sd1 nolist mt=data mt=view nodetails;delete want; run;quit;

    %utl_submit_wps64x("
    libname sd1 'd:/sd1';
    options validvarname=any;
    proc sql;
      create
        table sd1.want as
      select
        name
       ,max(case when partition=1 then id else . end) as id1
       ,max(case when partition=2 then id else . end) as id2
       ,max(case when partition=3 then id else . end) as id3
      from
        %sqlpartition(sd1.have,by=name)
      group
        by name
    ;quit;
    ");

    proc print data=sd1.want width=min;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  SD1.WANT total obs=4                                                                                                  */
    /*                                                                                                                        */
    /*  Obs     NAME     ID1    ID2    ID3                                                                                    */
    /*                                                                                                                        */
    /*   1     Cecili     24     12      .                                                                                    */
    /*   2     Mayana     13     24      .                                                                                    */
    /*   3     Sierra     56      .      .                                                                                    */
    /*   4     Sophia     54     12     15                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                                          _
    |___ \  __      ___ __  ___   _ __   ___  __ _| |
      __) | \ \ /\ / / `_ \/ __| | `__| / __|/ _` | |
     / __/   \ V  V /| |_) \__ \ | |    \__ \ (_| | |
    |_____|   \_/\_/ | .__/|___/ |_|    |___/\__, |_|
                     |_|                        |_|
    */
    proc datasets lib=sd1 nolist mt=data mt=view nodetails;delete want; run;quit;

    %utl_submit_wps64x("
    options validvarname=any;
    libname sd1 'd:/sd1';
    proc r;
    export data=sd1.have r=have;
    submit;
    library(sqldf);
    want<-sqldf('
       select
          name
          ,max(case when partition=1 then id else NULL end) as id1
          ,max(case when partition=2 then id else NULL end) as id2
          ,max(case when partition=3 then id else NULL end) as id3
       from
          (select name, id, row_number() OVER (PARTITION BY name) as partition from have )
       group
         by name
    ');
    want;
    endsubmit;
    import data=sd1.want r=want;
    run;quit;
    ");

    proc print data=sd1.want width=min;
    run;quit;

    /**************************************************************************************************************************/
    /*                       |                                                                                                */
    /* The WPS System        |  WPS System                                                                                    */
    /*                       |                                                                                                */
    /*     name id1 id2 id3  |  Obs     NAME     ID1    ID2    ID3                                                            */
    /*                       |                                                                                                */
    /* 1 Cecili  24  12  NA  |   1     Cecili     24     12      .                                                            */
    /* 2 Mayana  13  24  NA  |   2     Mayana     13     24      .                                                            */
    /* 3 Sierra  56  NA  NA  |   3     Sierra     56      .      .                                                            */
    /* 4 Sophia  54  12  15  |   4     Sophia     54     12     15                                                            */
    /*                       |                                                                                                */
    /**************************************************************************************************************************/

    /*____                                    _   _                             _
    |___ /  __      ___ __  ___   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
      |_ \  \ \ /\ / / `_ \/ __| | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) |  \ V  V /| |_) \__ \ | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____/    \_/\_/ | .__/|___/ | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
                     |_|         |_|    |___/                                |_|
    */

    %utlfkil(d:/xpt/want.xpt);

    /*----                                                                   ----*/
    /*----  elimiate issues with datastep and view with the same name        ----*/
    /*----                                                                   ----*/

    proc datasets lib=sd1 nolist mt=data mt=view nodetails;delete want; run;quit;
    proc datasets lib=work nolist mt=data mt=view nodetails;delete want; run;quit;

    %utl_submit_wps64x("
    options validvarname=any lrecl=32756;
    libname sd1 'd:/sd1';
    proc python;
    export data=sd1.have  python=have;
    export data=sd1.havpost python=havpost;
    submit;
    import pyreadstat as ps;
    from os import path;
    import pandas as pd;
    import numpy as np;
    from pandasql import sqldf;
    mysql = lambda q: sqldf(q, globals());
    from pandasql import PandaSQL;
    pdsql = PandaSQL(persist=True);
    sqlite3conn = next(pdsql.conn.gen).connection.connection;
    sqlite3conn.enable_load_extension(True);
    sqlite3conn.load_extension('c:/temp/libsqlitefunctions.dll');
    mysql = lambda q: sqldf(q, globals());
    want = pdsql('''
       select
          name
          ,max(case when partition=1 then id else NULL end) as id1
          ,max(case when partition=2 then id else NULL end) as id2
          ,max(case when partition=3 then id else NULL end) as id3
       from
          (select name, id, row_number() OVER (PARTITION BY name) as partition from have )
       group
         by name
    ''');
    print(want);
    ps.write_xport(want,'d:\\xpt\\want.xpt',table_name='want',file_format_version=5
    ,column_labels=['NAME','ID1','ID2','ID3']);
    endsubmit;
    ");

    /*--- handles long variable names by using the label to rename the variables  ----*/

    libname xpt xport "d:/xpt/want.xpt";
    proc contents data=xpt._all_;
    run;quit;

    data want_py_long_names;
      %utl_rens(xpt.want) ;
      set want;
    run;quit;
    libname xpt clear;

    proc print data=want_py_long_names;
    run;quit;

    /**************************************************************************************************************************/
    /*                                  |                                                                                     */
    /*  The WPS PYTHON Procedure        |   WPS System                                                                        */
    /*                                  |                                                                                     */
    /*         name   id1   id2   id3   |   Obs     NAME     ID1    ID2    ID3                                                */
    /*                                  |                                                                                     */
    /*  0  Cecili    24.0  12.0   NaN   |    1     Cecili     24     12      .                                                */
    /*  1  Mayana    13.0  24.0   NaN   |    2     Mayana     13     24      .                                                */
    /*  2  Sierra    56.0   NaN   NaN   |    3     Sierra     56      .      .                                                */
    /*  3  Sophia    54.0  12.0  15.0   |    4     Sophia     54     12     15                                                */
    /*                                  |                                                                                     */
    /**************************************************************************************************************************/

    /*  _                                                  _
    | || |   __      ___ __  ___   _ __   ___    ___  __ _| |
    | || |_  \ \ /\ / / `_ \/ __| | `_ \ / _ \  / __|/ _` | |
    |__   _|  \ V  V /| |_) \__ \ | | | | (_) | \__ \ (_| | |
       |_|     \_/\_/ | .__/|___/ |_| |_|\___/  |___/\__, |_|
                      |_|                               |_|
    */

    %utl_submit_wps64x("
    libname sd1 'd:/sd1';
    proc transpose data=sd1.have out=sd1.want prefix=ID;
    by name notsorted;
    run;quit;

    proc print data=sd1.want width=min;
    run;quit;
    ");

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Obs     NAME     _NAME_    ID1    ID2    ID3                                                                           */
    /*                                                                                                                        */
    /*  1     Mayana      ID       13     24      .                                                                           */
    /*  2     Sierra      ID       56      .      .                                                                           */
    /*  3     Sophia      ID       54     12     15                                                                           */
    /*  4     Cecili      ID       24     12      .                                                                           */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                                                        _
    | ___|  __      ___ __  ___   _ __   _ __   ___    ___  __ _| |
    |___ \  \ \ /\ / / `_ \/ __| | `__| | `_ \ / _ \  / __|/ _` | |
     ___) |  \ V  V /| |_) \__ \ | |    | | | | (_) | \__ \ (_| | |
    |____/    \_/\_/ | .__/|___/ |_|    |_| |_|\___/  |___/\__, |_|
                     |_|                                      |_|
    */

    %utl_submit_wps64x("
    options validvarname=any;
    libname sd1 'd:/sd1';
    proc r;
    export data=sd1.have r=have;
    submit;
    library(tidyverse);
    have |>
      mutate(idno = row_number(), .by = NAME) |>
      pivot_wider(
        values_from = ID
       ,names_from = idno
       ,values_fill = NaN
       ,names_prefix = 'ID'
      );
    endsubmit;
    import data=sd1.want r=want;
    run;quit;

    proc print data=sd1.want width=min;
    run;quit;
    ");

    /**************************************************************************************************************************/
    /*                               |                                                                                        */
    /*  The R WPS System             |                                                                                        */
    /*                               |                                                                                        */
    /*    NAME     ID1   ID2   ID3   |    NAME     _NAME_    ID1    ID2    ID3                                                */
    /*                               |                                                                                        */
    /*  1 Mayana    13    24   NaN   |   Mayana      ID       13     24      .                                                */
    /*  2 Sierra    56   NaN   NaN   |   Sierra      ID       56      .      .                                                */
    /*  3 Sophia    54    12    15   |   Sophia      ID       54     12     15                                                */
    /*  4 Cecili    24    12   NaN   |   Cecili      ID       24     12      .                                                */
    /*                               |                                                                                        */
    /**************************************************************************************************************************/


    /*----                                                                   ----*/
    /*----  elimiate issues with datastep and view with the same name        ----*/
    /*----                                                                   ----*/

    proc datasets lib=sd1 nolist mt=data mt=view nodetails;delete want; run;quit;
    proc datasets lib=work nolist mt=data mt=view nodetails;delete want; run;quit;

    %utl_submit_wps64x("
    options validvarname=any lrecl=32756;
    libname sd1 'd:/sd1';
    proc python;
    export data=sd1.have  python=have;
    submit;
    import pyreadstat as ps;
    from os import path;
    import pandas as pd;
    import numpy as np;
    from pandasql import sqldf;
    mysql = lambda q: sqldf(q, globals());
    from pandasql import PandaSQL;
    pdsql = PandaSQL(persist=True);
    sqlite3conn = next(pdsql.conn.gen).connection.connection;
    sqlite3conn.enable_load_extension(True);
    sqlite3conn.load_extension('c:/temp/libsqlitefunctions.dll');
    mysql = lambda q: sqldf(q, globals());
    df = pdsql('''
       select
          name
         ,id
         ,partition
       from
          (select name, id, row_number() OVER (PARTITION BY name) as partition from have )
    ''');
    want = df.pivot(index='name', columns='partition', values='id');
    print(want);
    endsubmit;
    ");

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  The WPS PYTHON Procedure                                                                                              */
    /*                                                                                                                        */
    /*  partition     1     2     3                                                                                           */
    /*  name                                                                                                                  */
    /*  Cecili     24.0  12.0   NaN                                                                                           */
    /*  Mayana     13.0  24.0   NaN                                                                                           */
    /*  Sierra     56.0   NaN   NaN                                                                                           */
    /*  Sophia     54.0  12.0  15.0                                                                                           */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____            _       _           _
    |___  |  _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
       / /  | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
      / /   | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
     /_/    |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                      |_|
    */

    https://github.com/rogerjdeangelis/utl-find-first-n-observations-per-category-using-proc-sql-partitioning
    https://github.com/rogerjdeangelis/utl-macro-to-enable-sql-partitioning-by-groups-montonic-first-and-last-dot
    https://github.com/rogerjdeangelis/utl-partitioning-your-table-for-a-big-parallel-systask-sort
    https://github.com/rogerjdeangelis/utl-pivot-transpose-by-id-using-wps-r-python-sql-using-partitioning
    https://github.com/rogerjdeangelis/utl-top-four-seasonal-precipitation-totals--european-cities-sql-partitions-in-wps-r-python
    https://github.com/rogerjdeangelis/utl-transpose-pivot-wide-using-sql-partitioning-in-wps-r-python
    https://github.com/rogerjdeangelis/utl-transposing-rows-to-columns-using-proc-sql-partitioning
    https://github.com/rogerjdeangelis/utl-transposing-words-into-sentences-using-sql-partitioning-in-r-and-python
    https://github.com/rogerjdeangelis/utl-using-sql-in-wps-r-python-select-the-four-youngest-male-and-female-students-partitioning
    https://github.com/rogerjdeangelis/utl_partition_a_list_of_numbers_into_3_groups_that_have_the_similar_sums_python
    https://github.com/rogerjdeangelis/utl_partition_a_list_of_numbers_into_k_groups_that_have_the_similar_sums
    https://github.com/rogerjdeangelis/utl_scalable_partitioned_data_to_find_statistics_on_a_column_by_a_grouping_variable


     https://github.com/rogerjdeangelis/utl_an_unsusual_transpose_based_on__groups_of_variable_names
     https://github.com/rogerjdeangelis/utl_classic_transpose_in_r_and_sas
     https://github.com/rogerjdeangelis/utl_diagonal_transpose_while_keeping_all_original_rows
     https://github.com/rogerjdeangelis/utl_excel_Import_and_transpose_range_A9-Y97_using_only_one_procedure
     https://github.com/rogerjdeangelis/utl_flexible_complex_multi-dimensional_transpose_using_one_proc_report
     https://github.com/rogerjdeangelis/utl_simple_three_dimensional_transpose_in_r_and_sas
     https://github.com/rogerjdeangelis/utl_sophisticated_transpose_with_proc_summary_idgroup
     https://github.com/rogerjdeangelis/utl_sort_summarize_and_transpose_multiple_variable_and_create_output_dataset
     https://github.com/rogerjdeangelis/utl_sort_summarize_transpose_and_format_in_1_datastep
     https://github.com/rogerjdeangelis/utl_sort_transpose_and_summarize_a_dataset_using_just_one_proc_report
     https://github.com/rogerjdeangelis/utl_sort_transpose_and_summarize_in_one_proc_v2
     https://github.com/rogerjdeangelis/utl_sort_transpose_summarize
     https://github.com/rogerjdeangelis/utl_sql_version_of_proc_transpose_with_major_advantage_of_summarization
     https://github.com/rogerjdeangelis/utl_techniques_to_transpose_and_stack_multiple_variables
     https://github.com/rogerjdeangelis/utl_transpose_and_concatenate_observations_by_id_in_one_datastep
     https://github.com/rogerjdeangelis/utl_transpose_long_to_wide_with_sequential_matching_pairs
     https://github.com/rogerjdeangelis/utl_transpose_multiple_variables_and_split_variables_into_multiple_variables
     https://github.com/rogerjdeangelis/utl_transpose_rows_to_column_identifying_type_of_data
     https://github.com/rogerjdeangelis/utl_transpose_table_by_two_variables_not_supported_by_proc_transpose
     https://github.com/rogerjdeangelis/utl_transpose_with_multiple_id_values_per_group
     https://github.com/rogerjdeangelis/utl_transpose_with_proc_report
     https://github.com/rogerjdeangelis/utl_transpose_with_proc_sql
     https://github.com/rogerjdeangelis/utl_transposing_multiple_variables_with_different_ids_a_single_transpose_cannot_do_this
     https://github.com/rogerjdeangelis/utl_two_families_itinerary_through_italy_transpose
     https://github.com/rogerjdeangelis/utl_using_a_hash_to_transpose_and_reorder_a_table


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
Simple classic transpose pivot wider in native and sql wps r python  
