#utl-identify-the-most-recent-file-in-the-directory-and-copy-the-file-to-another-folder

    Identify most recent file in directory amnd copy file to aother folder

    github
    https://tinyurl.com/tzludfw
    https://github.com/rogerjdeangelis/utl-identify-the-most-recent-file-in-the-directory-and-copy-the-file-to-another-folder

    related repo
    https://goo.gl/gChmFX
    https://github.com/rogerjdeangelis/utl_file_and_directory_utilities_for_all_operating_systems

    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    SAS Forum
    https://tinyurl.com/yx3fcdzx
    https://communities.sas.com/t5/SAS-Programming/How-to-find-the-latest-file-and-how-to-move-it-in-a-different/m-p/639687

    * you nrrd this macro;
    %macro dosubl(arg);
      %let rc=%qsysfunc(dosubl(&arg));
    %mend dosubl;

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    * source and target directories;

    %let dirSrc=d:\fro;   * source directory  might be better to use backslahes ;
    %let dirTrg=d:\too;   * target directory;


       d:\fro              Last Modifieed date

          +-- file1.txt    14Apr2020:12:06:20
          +-- file2.txt    14Apr2020:12:07:30

       d:/too (currently empty)


    * CREATE SAMPE DIRECTORIES AND FILES;

    data _null_;

         * create directory;
         if _n_=0 then do; %dosubl('
                data _null_;
                    length cc $8;
                    cc=dcreate("fro","d:/");
                    cc=dcreate("too","d:/");
                run;quit;
            ');
         end;

         * you need to use dosubl here because cration date would be the same without dosubl;

         rc=dosubl('data _null_; file "d:/fro/file1.txt"; put "file1";run;quit;');
         * sleep gpt 1 min 10 seconds;
         call sleep(70,1); * sleep for 1 min 10 seconds;

         rc=dosubl('data _null_; file "d:/fro/file2.txt"; put "file1";run;quit;');

         stop;

     run;quit;

    * macro on end an in github;
    * directory listing to SAS table;


    %utl_dirContents( dir=d:\fro,dsout=work.have);


    Up to 40 obs WORK.HAVE total obs=2
                                                                          FILE_
                                                                          SIZE__
    Obs    BASEFILE         PATHNAME        FILE_SEQ    RECFM    LRECL    BYTES_      LAST_MODIFIED          CREATE_TIME

     1     file1.txt    d:\fro\file1.txt        1         V       384       7       14Apr2020:12:06:20    14Apr2020:11:44:23
     2     file2.txt    d:\fro\file2.txt        2         V       384       7       14Apr2020:12:07:30    14Apr2020:11:44:23


    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    This is a copy of the most recent file in d:/fro


       d:/too             LAST_MODIFIED

         +-- file2.txt    14Apr2020:12:07:30

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    * for testing only;
    %symdel dirSrc dirTrg fylsrc last_modified / nowarn;

    filename src clear;
    filename trg clear;

    %utlfkil(d:/too/file2.txt);

    * SOLUTION;

    %let dirSrc=d:\fro;   * from directory  might be better to use backslahes ;
    %let dirTrg=d:\too;   * target directory;

    /* and you need work,have */

    data _null_;

      if _n_=0 then do; %dosubl('
          * get latest file;
          proc sql;
            select
               basefile
              ,last_modified
            into
               :fylSrc         trimmed
              ,:last_modified  trimmed
            from
               have
            having
               max(input(last_modified ,anydtdtm.))=input(last_modified ,anydtdtm.)
          ;quit;

          /* filename not supported inside dosubl? */
          run;quit;
          ');
      end;
      * should work for binary files like SAS&BDATs;
      rc = filename('src', 'd:/fro/file2.txt', , 'recfm = n' ) ;
      rc = filename('trg', 'd:/too/file2.txt', , 'recfm = n' ) ;
      rc = fcopy('src','trg');

    run;quit;

    filename src clear;
    filename trg clear;

    *
     _ __ ___   __ _  ___ _ __ ___  ___
    | '_ ` _ \ / _` |/ __| '__/ _ \/ __|
    | | | | | | (_| | (__| | | (_) \__ \
    |_| |_| |_|\__,_|\___|_|  \___/|___/

    ;

    %macro utl_dirContents(
     dir= /* directory name to process */
    , ext= /* optional extension to filter on */
    , dsout=work.dir_contents /* dataset name to hold the file names */
    , attribs=Y /* get file attributes? (Y/N ) */
    );
    /*
    michael.goulding@experis.com
    https://www.pharmasug.org/proceedings/2015/QT/PharmaSUG-2015-QT23.pdf
    */
    %global _dir_fileN;
    %local _syspathdlim _exitmsg _attrib_vars;
    %* verify the required parameter has been provided. ;
    %if %length(&dir) = 0 %then %do;
     %let _exitmsg = %str(E)RROR: No directory name specified - macro will exit.;
     %goto finish;
    %end;
    %* verify existence of the requested directory name. ;
    %if %sysfunc(fileexist(&dir)) = 0 %then %do;
     %let _exitmsg = %str(E)RROR: Specified input location, &dir., does not exist - macro
    will exit.;
     %goto finish;
    %end;
    %* set the separator character needed for the full file path: ;
    %* (backslash for Windows, forward slash for UNIX systems) ;
    %if &sysscp = WIN %then %do;
     %let _syspathdlim = \;
    %end;
    %else %do;
     %let _syspathdlim = /;
    %end;
    /*--- begin data step to capture names of all file names found in the specified
    directory. ---*/
    data &dsout(keep=file_seq basefile pathname);
     length basefile $ 40 pathname $ 1000 _msg $ 1000;
     /* Allocate directory */
     rc=FILENAME('xdir', "&dir");

     if rc ne 0 then do;
     _msg = "E" || 'RROR: Unable to assign fileref to specified directory. ' ||
    sysmsg();
     go to finish_datastep;
     end;
     /* Open directory */
     dirid=DOPEN('xdir');
     if dirid eq 0 then do;
     _msg = "E" || 'RROR: Unable to open specified directory. ' || sysmsg();
     go to finish_datastep;
     end;
     /* Get number of information items */
     nfiles=DNUM(dirid);
     do j = 1 to nfiles;
     basefile = dread(dirid, j);
     pathname=strip("&dir") || "&_syspathdlim." || strip(basefile);

     %if %length(&ext) %then %do;
    /* scan the final "word" of the full file name, delimited by dot character. */

     ext = scan(basefile,-1,'.');
     if ext="&ext." then do;
     file_seq + 1;
     output;
     end;
     %end;
     %else %do;
     file_seq + 1;
     output;
     %end;
     end;
     /* Close the directory */
     rc=DCLOSE(dirid);
     /* Deallocate the directory */
     rc=FILENAME('xdir');

     call symputx('_dir_fileN', file_seq);
     finish_datastep:
     if _msg ne ' ' then do;
     call symput('_exitmsg', _msg);
     end;
    run;
    %if %upcase(&attribs)=Y and &_dir_fileN > 0 %then %do;
     data _file_attr(keep=file_seq basefile infoname infoval);
     length infoname infoval $ 500;
     set &dsout.;
    /* open each file to get the additional attributes available. */
     rc=filename("afile", pathname);
     fid=fopen("afile");
    /* return the number of system-dependent information items available for the
    external file. */
     infonum=foptnum(fid);
    /* loop to get the name and value of each information item. */
     do i=1 to infonum;
     infoname=foptname(fid,i);
     infoval=finfo(fid,infoname);
     if upcase(infoname) ne 'FILENAME' then output;
     end;
     close=fclose(fid);
     run;
     /* transpose each information item into its own variable */
     proc transpose data=_file_attr out=trans_attr(drop=_:) ;
     by file_seq basefile ;
     var infoval;
     id infoname;
     run;
     proc sql noprint;
     select distinct name into : _attrib_vars separated by ', '
     from dictionary.columns
     where memname='TRANS_ATTR' and upcase(name) not in('BASEFILE', 'FILE_SEQ')
     order by varnum;
     quit;
     /* merge back the additional attributes to the related file name. */
     data &dsout.;
     merge &dsout. trans_attr;
     by file_seq basefile;
     run;
     proc datasets nolist memtype=data lib=work;
     delete _file_attr trans_attr;
     run;
     quit;
    %end;
    %if %length(&_exitmsg) = 0 %then
    %let _exitmsg = NOTE: &dsout created with &_dir_fileN. file names ;
    %if %length(&ext) %then
    %let _exitmsg = &_exitmsg where extension is equal to &ext.;
    %let _exitmsg = &_exitmsg from &dir..;
    %finish:
    %put &_exitmsg;
    %if %length(&_attrib_vars) ne 0 %then %do;
     %put;
     %put NOTE: File attributes were requested and have been added to &dsout.. Variable
    names are &_attrib_vars.;
    %end;
    %mend utl_dirContents;


    %utl_dirContents(dir=c:\parent\child_1,dsout=work.want);


    /*
    Up to 40 obs from WORK.WANT total obs=3
                                                                                     FILE_
                                                                                     SIZE__
    Obs    BASEFILE              PATHNAME              FILE_SEQ    RECFM    LRECL    BYTES_      LAST_MODIFIED          CREATE_TIME

     1     file1.txt    c:\parent\child_1\file1.txt        1         V       384       7       14Apr2020:16:43:22    25Mar2020:16:34:23
     2     file2.txt    c:\parent\child_1\file2.txt        2         V       384       7       14Apr2020:16:43:22    25Mar2020:16:34:23
     3     file3.txt    c:\parent\child_1\file3.txt        3         V       384       7       14Apr2020:16:43:22    25Mar2020:16:34:23
