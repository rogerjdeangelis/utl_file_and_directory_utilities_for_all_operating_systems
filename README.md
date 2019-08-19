# utl_file_and_directory_utilities_for_all_operating_systems
Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    File and directory utilities for all operating systems

    see github
    https://goo.gl/gChmFX
    https://github.com/rogerjdeangelis/utl_file_and_directory_utilities_for_all_operating_systems

    Unix, Perl and Python have some amazing additional tools(not present here)

    Many of these have unique advantages, however the best windows tool to create
    SAS dataset from a directory is  #11. This tool was written by SAS.

    Many of these have unique advantages, however the best fro putting a windows
    directory into a SAS dataset is #11. It has a lot of bells and whistles.

    see
    http://support.sas.com/kb/24/820.html

     Directory and File Tools

       1.  Windows: directory tree (windows)
       2.  All Platforms - Number of file, file number and filename
       3.  Windows: Recursively list all directories, sub-directories and files into a SAS datasets (DATA _NULL_ SAS-L)
       4.  Window:  Directory -  creation, size, owner, filename  (with sub-directories)
       5.  Unix:    Simple directory information into a sas dataset.
       6.  Unix:    directory information into a sas dataset.
       7.  Windows: delete empty files in a directory
       8.  Windows: members of zip archive into SAS dataset
       9.  Windows: Roland Rashleigh-Berry Windows directory into macro variable(RIP)
      10.  Windows: Another Windows directory into macro variable;
      11.  Windows: Another Windows directory into macro variable;
      12.  Unix: Return the path up one level
      13.  All Platforms does a file exist
      14.  Given a full path extract the folder and filename with and without extension
      15. Delete empty directory
      16. Detailed-List-of-Directories-and-Files-in-Linux    
          https://tinyurl.com/y59u45t7                       

    *                _          _                   _
     _ __ ___   __ _| | _____  (_)_ __  _ __  _   _| |_
    | '_ ` _ \ / _` | |/ / _ \ | | '_ \| '_ \| | | | __|
    | | | | | | (_| |   <  __/ | | | | | |_) | |_| | |_
    |_| |_| |_|\__,_|_|\_\___| |_|_| |_| .__/ \__,_|\__|
                                       |_|
    ;

       data _null_;

         * create directory;
         if _n_=0 then do;
             %let rc=%sysfunc(dosubl('
                data _null_;
                    rc=dcreate("parent","c:/");
                    rc=dcreate("child_1","c:/parent");
                    rc=dcreate("child_2","c:/parent");
                run;quit;
            '));
         end;

         file "c:/parent/child_1/file1.txt"; put "file1";
         file "c:/parent/child_1/file2.txt"; put "file2";
         file "c:/parent/child_1/file3.txt"; put "file3";

         file "c:/parent/child_2/file1.txt"; put "file1";
         file "c:/parent/child_2/file2.txt"; put "file2";
         file "c:/parent/child_2/file3.txt";

       run;quit;


    INPUT
    =====

        C:\PARENT

           +---CHILD_1
                  |
                  +-- file1.txt
                  +-- file2.txt
                  +-- file3.txt

           \---CHILD_2
                  |
                  +-- file1.txt
                  +-- file2.txt
                  +-- file3.txt  (this one is empty)

    PROCESS and OUTPUT
    ==================

     1.  WINDOWS DIRECTORY TREE
     ==========================

        filename pipetree pipe 'tree "c:\parent" /F /A' lrecl=5000;
        data _null_;
           infile pipetree truncover;
           input dirlist $char1000.;
           if index(dirlist,'.') =0;
           put dirlist;
        run;quit;

        Folder PATH listing
        Volume serial number is 883A-2108

        C:\PARENT

           +---child-1
           |
           \---child_2


      2. ALL PLATFORMS - NUMBER OF FILES, FILE NUMBER AND FILENAME
         ========================================================

          %macro utl_dir(pth,out);
           data &out(drop=rc did i);
             retain filename number_of_members filename member_number;
             rc=filename("mydir","&pth");
             did=dopen("mydir");
             if did > 0 then do;
                number_of_members=dnum(did);
                do i=1 to number_of_members;
                  filename=dread(did,i);
                  member_number+1;
                  output;
               end;
            end;
          run;quit;

          %mend utl_dir;

          %utl_dir(c:/parent,child_1);

          proc print data=child_1;
          run;quit;

        OUTPUT

          WORK,CHILD_1 total obs=3
                             NUMBER_OF_    MEMBER_
          Obs    FILENAME      MEMBERS      NUMBER

           1     child_1          2           1
           2     child_2          2           2


      3. RECURSIVELY LIST ALL DIRECTORIES, SUB-DIRECTORIES AND FILES INTO A SAS DATASETS
         ================================================================================

          Here is what is going on

            1. Build a template to hold the root, path and dir(depth)
            2. Modifily that datsets as new objects are found
            3. Use SAS dir commands to traverse directories

          data dir;
             length root path $200 dir 8;
             call missing(path,dir);
             input root;
           cards;
           c:/parent
           ;run;

           data dir;
             modify dir;
             rc=filename('tmp',catx('/',root,path));
             dir=dopen('tmp');
             replace;
             if dir;
             path0=path;
             do _N_=1 to dnum(dir);
               path=catx('/',path0,dread(dir,_N_));
               output;
               end;
             rc=dclose(dir);
           run;quit;

         OUTPUT

           WORK.DIR total obs=9

              ROOT       PATH                 DIR

            c:/parent                          1

            c:/parent    child_1               1
            c:/parent    child_2               1

            c:/parent    child_1/file1.txt     0
            c:/parent    child_1/file2.txt     0
            c:/parent    child_1/file3.txt     0
            c:/parent    child_2/file1.txt     0
            c:/parent    child_2/file2.txt     0
            c:/parent    child_2/file3.txt     0


       4.  Window directory -  creation, size, owner, filename
       =======================================================

       filename pipes pipe 'DIR /TC /Q /S c:\parent\';
       data Dir(drop=command);
          length dir command $128;
          infile pipes truncover end=eof;
          do until(eof);
             input @;
             if index(_infile_,'Directory of') then input @ 'Directory of ' dir $128.;
             else do;
                input creation ?? mdyampm18. @;
                if not missing(creation) then do;
                   input @26 size:??comma16.  @40 owner:$32. @63 name$64.;
                   output;
                   end;
                else input;
                end;
              *put _infile_;
             end;
          _error_ = 0;
          stop;
          format creation datetime.;
          run;

       proc print;
       run;quit;

      OUTPUT

       40 obs from dir total obs=9

        DIR                      CREATION        SIZE    OWNER          NAME

        c:\parent            25JAN18:14:14:00      .     BEAST\beast
        c:\parent            25JAN18:14:14:00      .     NT             ..
        c:\parent            25JAN18:14:14:00      .     BEAST\beast    child_1
        c:\parent            25JAN18:14:14:00      .     BEAST\beast    child_2
        c:\parent\child_1    25JAN18:14:14:00      .     BEAST\beast
        c:\parent\child_1    25JAN18:14:14:00      .     BEAST\beast    ..
        c:\parent\child_1    25JAN18:14:14:00      7     BEAST\beast    file1.txt
        c:\parent\child_1    25JAN18:14:14:00      7     BEAST\beast    file2.txt
        c:\parent\child_1    25JAN18:14:14:00      7     BEAST\beast    file3.txt
        c:\parent\child_2    25JAN18:14:14:00      .     BEAST\beast
        c:\parent\child_2    25JAN18:14:14:00      .     BEAST\beast    ..
        c:\parent\child_2    25JAN18:14:14:00      7     BEAST\beast    file1.txt
        c:\parent\child_2    25JAN18:14:14:00      7     BEAST\beast    file2.txt

        c:\parent\child_2    25JAN18:14:14:00      0     BEAST\beast    file3.txt * note 0 size;


    5. SIMPLE UNIX DIRECTORY INFORMATION INTO A SAS DATASET.
    =======================================================

        %macro dirlst(pth=%str(.));
          filename oecmd pipe "ls -l &pth";
          data dirlst;
          length fyl $100 usr $16 siz $20;
            infile oecmd truncover lrecl=500 pad;
            input;
            fyl=left(reverse(scan(left(reverse(_infile_)),1,' ')));
            usr=scan(_infile_,3,' ');
            siz=scan(_infile_,4,' ');
            dte=substr(_infile_,index(_infile_,strip(siz))+length(strip(siz)));
            dte=substr(dte,1,index(dte,fyl)-1);
          run;quit;
        %mend dirlst;


      OUTPUT

        User   System            Siz_create           File

        roger  sas_compucraft    6 25Jan2018:15:09   file1.txt
        roger  sas_compucraft    6 25Jan2018:15:09   file2.txt
        roger  sas_compucraft    0 25Jan2018:15:09   file3.txt  * note empty file


      6.  UNIX DIRECTORY INFORMATION INTO A SAS DATASET.
      ==================================================

         %macro ls_l(dir)/des="directory list when SAS turns off the operating system suppot";

            %put %sysfunc(ifc(%sysevalf(%superq(dir)=,boolean),**** Please Provide a directory    ****,));

             %let res= %eval
             (
                 %sysfunc(ifc(%sysevalf(%superq(dir )=,boolean),1,0))
             );

              %if &res = 0 %then %do;

                 data work.__dirout(keep=lstname fyl1-fyl6);
                    length infoname infoval $60;
                    retain lstname;
                    length lstname $64;
                    rc=filename("_mydir","&dir");
                    did=dopen("_mydir");
                    if did > 0 then do;
                       memcount=dnum(did);
                       close=fclose(did);
                       do i=1 to memcount;
                          lstname=dread(did,i);
                          rc=filename("_myfyl",cats("&dir",'/',lstname));
                          fid=fopen("_myfyl");
                          if fid > 0 then do;
                             infonum=foptnum(fid);
                              array nfo[6] $255 fyl1-fyl6;
                              do j=1 to infonum;
                                 infoname=foptname(fid,j);
                                 infoval=finfo(fid,infoname);
                                 nfo[j]=finfo(fid,infoname);
                                 if j=6 then nfo[j]=put(input(nfo[j],16.),comma15.);
                              end;
                              if i=1 then do;
                                 put "Directory &dir" /;
                              end;
                              *put @1 fylnum comma16. @20 nfo[5] $20. nfo[6] $20. lstname $64.;
                              put nfo[2] $8. nfo[3] $8. @19 nfo[4] $10. @32 nfo[5] $15. @50  nfo[6] $15.-r @70  lstname ;
                              output;
                              close=fclose(fid);
                          end;
                          else do;
                             put "subdirectory or &dir/" lstname " no files in directory or file cannot be opened";

                          end;
                       end;
                   end;
                   else do;
                   put "&dir does not exist or cannot be opened";
                 end;
              %end;

            run;quit;
         %mend ls_l;

       OUTPUT

         WORK.__dirout

         User   System           Read/Write Hier   Creation Date    Size   File

         roger  sas_compucraft   -rw-rw----        25Jan2018:15:09   6     file1.txt
         roger  sas_compucraft   -rw-rw----        25Jan2018:15:09   6     file2.txt
         roger  sas_compucraft   -rw-rw----        25Jan2018:15:09   0     file3.txt  * note empty file


     7.  Windows delete empty files in a directory
     =============================================

       %macro utl_delmty(folder)/des="Delete empty folders";
          filename filelist "&folder";
          data _null_;
             dir_id = dopen('filelist');
             total_members = dnum(dir_id);
             do i = 1 to total_members;  /* walk through all the files in the folder */
                member_name = dread(dir_id,i);
                file_id = mopen(dir_id,member_name,'i',0);
                if file_id > 0 then do; /* if the file is readable */
                   freadrc = fread(file_id);
                   if freadrc = -1 then do; /* if the file is empty */
                      rc = fclose(file_id);
                      rc = filename('delete',member_name,,,'filelist');
                      rc = fdelete('delete');
                   end;
                end;
                rc = fclose(file_id);
             end;
             rc = dclose(dir_id);
          run;
       %mend utl_delmty;

       %utl_delmty(c:/parent/child_2);

       c:\parent\child_2\file3.txt  ( file was delete);


     8.  Windows members of zip archive into SAS dataset
     ===================================================


        filename inzip zip "d:/zip/trepac.zip";
        /* Read the "members" (files) from the ZIP file */
        data contents(keep=memname);
            length memname $200;
            fid=dopen("inzip");
            if fid=0 then
                stop;
            memcount=dnum(fid);
            do i=1 to memcount;
                memname=dread(fid,i);
                output;
            end;
            rc=dclose(fid);
        run;
        /* create a report of the ZIP contents */
        title "Files in the ZIP file";
        proc print data=contents noobs N;
        run;


    9.  Windows Roland Rashleigh-Berry Windows directory into macro variable;


         /*<pre><b>
         / Program   : qreadpipe.sas
         / Version   : 2.1
         / Author    : Roland Rashleigh-Berry
         / Date      : 23-Sep-2011
         / Purpose   : Function-style macro to read the output of a system command and
         /             return the result trimmed and MACRO QUOTED.
         / SubMacros : %qtrim
         / Notes     : Result will be MACRO QUOTED. Use %unquote to make the string
         /             output usable in ordinary sas code.
         / Usage     : %let mvar=%qreadpipe(echo $USER);
         /
         /===============================================================================
         / PARAMETERS:
         /-------name------- -------------------------description------------------------
         / command           (pos) System command. This should not be enclosed in quotes
         /                   but may be enclosed in %str(), %quote() etc..
         /===============================================================================
         / AMENDMENT HISTORY:
         / init --date-- mod-id ----------------------description------------------------
         / rrb  13Feb07         "macro called" message added
         / rrb  22Jul07         Header tidy
         / rrb  30Jul07         Header tidy
         / rrb  31Oct08         Major redesign for v2.0
         / rrb  12Oct09         Macro renamed from readpipe to qreadpipe (v2.1)
         / rrb  04May11         Code tidy
         / rrb  23Sep11         Header tidy
         /===============================================================================
         / This is public domain software. No guarantee as to suitability or accuracy is
         / given or implied. User uses this code entirely at their own risk.
         /=============================================================================*/

         %put MACRO CALLED: qreadpipe v2.1;

         %macro qreadpipe(command);
           %local fname fid str rc res err;
           %let err=ERR%str(OR);
           %let rc=%sysfunc(filename(fname,&command,pipe));
           %if &rc NE 0 %then %do;
             %put &err: (qreadpipe) Pipe file could not be assigned due to the following:;
             %put %sysfunc(sysmsg());
           %end;
           %else %do;
             %let fid=%sysfunc(fopen(&fname,s,80,b));
             %if &fid EQ 0 %then %do;
           %put &err: (qreadpipe) Pipe file could not be opened due to the following:;
           %put %sysfunc(sysmsg());
             %end;
             %else %do;
               %do %while(%sysfunc(fread(&fid)) EQ 0);
                 %let rc=%sysfunc(fget(&fid,str,80));
                 %let res=&res%superq(str);
               %end;
         %qtrim(&res)
               %let rc=%sysfunc(fclose(&fid));
               %if &rc NE 0 %then %do;
           %put &err: (qreadpipe) Pipe file could not be closed due to the following:;
           %put %sysfunc(sysmsg());
               %end;
               %let rc=%sysfunc(filename(fname));
               %if &rc NE 0 %then %do;
           %put &err: (qreadpipe) Pipe file could not be deassigned due to the following:;
           %put %sysfunc(sysmsg());
               %end;
             %end;
           %end;
         %mend qreadpipe;

         /*<pre><b>
         / Program   : dir.sas
         / Version   : 1.1
         / Author    : Roland Rashleigh-Berry
         / Date      : 26-Jun-2011
         / Purpose   : Function-style macro to return a list of members of a directory
         /             on a WINDOWS platform according to the file pattern you supply.
         /             If you supply just the directory name then all members are
         /             listed. This runs the MSDOS command in the form "dir /B mydir"
         / SubMacros : %qreadpipe
         / Notes     : Just the file names are returned unquoted. If you need the full
         /             path name in double quotes then use the %dirfpq macro instead
         /             which will correctly handle file names containing spaces.
         / Usage     : %let dirlist=%dir(C:\utilmacros);
         /             %let dirlist=%dir(C:\utilmacros\*.sas);
         /===============================================================================
         / PARAMETERS:
         /-------name------- -------------------------description------------------------
         / dir               (pos) Directory path name (no quotes)
         /===============================================================================
         / AMENDMENT HISTORY:
         / init --date-- mod-id ----------------------description------------------------
         / rrb  26Jun11         Remove quotes if supplied (v1.1)
         /===============================================================================
         / This is public domain software. No guarantee as to suitability or accuracy is
         / given or implied. User uses this code entirely at their own risk.
         /=============================================================================*/

         %put MACRO CALLED: dir v1.1;

         %macro dir(dir);
           %unquote(%qreadpipe(dir /B %sysfunc(dequote(&dir))))
         %mend dir;

       OUTPUT

         %let lst= %dir(c:\parent\child_2);
         %put &=lst;

         file1.txt  file2.txt

      10.  Windows: Another Windows directory into macro variable;



         /******************************************************************************
         ** PROGRAM:  CMN_MAC.ISDIR.SAS
         **
         ** DESCRIPTION: DETERMINES IF THE SPECIFIED PATH EXISTS OR NOT.
         **              RETURNS: 0 IF THE PATH DOES NOT EXIST OR COULD NOT BE OPENED.
         **                       1 IF THE PATH EXISTS AND CAN BE OPENED.
         **
         ** PARAMETERS: iPath: THE FULL PATH TO EXAMINE.  NOTE THAT / AND \ ARE TREATED
         **                    THE SAME SO &SASDIR/COMMON/MACROS IS THE SAME AS
         **                    &SASDIR\COMMON\MACROS.
         **
         *******************************************************************************
         ** VERSION:
         ** 1.0 ON: 13-JUL-07 BY: RP
         **     CREATED.
         ** 1.1 ON: 29-APR-10 BY: RP
         **     ADDED CLEANUP CODE SO FILES WOULD NOT REMAIN LOCKED.
         ** 1.2 ON: 15-APR-14 BY: JG
         **     ADDED MORE DEBUGGING INFO.
         ** 1.3 ON: 12-DEC-14 BY: RP
         **     CLEANED UP DEBUGGING INFO TO A SINGLE LINE
         ******************************************************************************/

         %macro isDir(iPath=,iQuiet=1);
           %local result dname did rc;

           %let result = 0;
           %let check_file_assign =  %sysfunc(filename(dname,&iPath));

           %put ASSIGNED FILEREF (0=yes, 1=no)? &check_file_assign &iPath;


           %if not &check_file_assign %then %do;

             %let did = %sysfunc(dopen(&dname));

             %if &did %then %do;
               %let result = 1;
             %end;
             %else %if not &iQuiet %then %do;
               %put &err: (ISDIR MACRO).;
               %put %sysfunc(sysmsg());
             %end;

             %let rc = %sysfunc(dclose(&did));

           %end;
           %else %if not &iQuiet %then %do;
             %put &err: (ISDIR MACRO).;
             %put %sysfunc(sysmsg());
           %end;

           &result

         %mend;
         %put %isDir(iPath=&sasdir\commonn\macros);


         %let isDir = %isDir(iPath=c:\parent\child_2);

         %put &=isDir;

       OUTPUT

         ISDIR=1

         /*%put %isDir(iPath=&sasdir/kxjfdkebnefe, iQuiet=0);*/
         /*%put %isDir(iPath=c:\temp);*/
         FileList Macro:

         /******************************************************************************
         ** PROGRAM:  MACRO.FILE_LIST.SAS
         **
         ** DESCRIPTION: RETURNS THE LIST OF FILES IN A DIRECTORY SEPERATED BY THE
         **              SPECIFIED DELIMITER. RETURNS AN EMPTY STRING IF THE THE
         **              DIRECTORY CAN'T BE READ OR DOES NOT EXIST.
         **
         ** PARAMETERS: iPath: THE FULL PATH TO EXAMINE.  NOTE THAT / AND \ ARE TREATED
         **                    THE SAME SO &SASDIR/COMMON/MACROS IS THE SAME AS
         **                    &SASDIR\COMMON\MACROS. WORKS WITH BOTH UNIX AND WINDOWS.
         **
         *******************************************************************************
         ** VERSION:
         ** 1.0 ON: 17-JUL-07 BY: RP
         **     CREATED.
         ** 1.1 ON: 29-APR-10 BY: RP
         **     ADDED CLEANUP CODE SO FILES WOULD NOT REMAIN LOCKED.
         **     FIXED ERROR OCCURRING WHEN IPATH DID NOT EXIST.
         ** 1.2 ON: 18-AUG-10 BY: RP
         **     CATERED FOR MACRO CHARS IN FILENAMES
         ** 1.3 ON: 14-APR-14 BY: RP&JG
         **     ADDED MORE DEBUGGING INFO TO CHECK PATH
         ** 1.4 ON: 13-NOV-14 BY: RP
         **     CHANGED PROGRAM FLOW TO MAKE IT BOTH EASIER TO READ AND TO
         **     REDUCE UNNECESSARY CHECKS / IMPROVE PERFORMANCE.
         ******************************************************************************/
         /*
         ** TODO. THERES ABOUT 100 WAYS THIS COULD BE IMPROVED.  DO IT SOMETIME.
         ** SIMPLY IF STATEMENTS FOR FILTERS.
         */
         %macro file_list(iPath=, iFilter=, iFiles_only=0, iDelimiter=|);
           %local result did dname cnt num_members filename rc check_dir_exist check_file_assign;

           %let result=;

           %let check_dir_exist = %isDir(iPath=&iPath);
           %let check_file_assign = %sysfunc(filename(dname,&iPath));

           %put The desired path:  &iPath;
           %if &check_dir_exist and not &check_file_assign %then %do;

             %let did = %sysfunc(dopen(&dname));
             %let num_members = %sysfunc(dnum(&did));

             %do cnt=1 %to &num_members;

               %let filename = %qsysfunc(dread(&did,&cnt));
               %if "&filename" ne "" %then %do;

                 %if "&iFilter" ne "" %then %do;
                   %if %index(%lowcase(&filename),%lowcase(&iFilter)) eq 0 %then %do;
                     %goto next;
                   %end;
                 %end;

                 %if &iFiles_only %then %do;
                   %if %isDir(iPath=%nrbquote(&iPath/&filename)) %then %do;
                     %goto next;
                   %end;
                 %end;

                 %let result = &result%str(&iDelimiter)&filename;

                 %next:

               %end;
               %else %do;
                 %put ERROR: (CMN_MAC.FILE_LIST) FILE CANNOT BE READ.;
                 %put %sysfunc(sysmsg());
               %end;
             %end;

             %let rc = %sysfunc(dclose(&did));

           %end;
           %else %do;

             %put ERROR: (CMN_MAC.FILE_LIST) PATH DOES NOT EXIST OR CANNOT BE OPENED.;
             %put %sysfunc(sysmsg());

             %put DIRECTORY EXISTS (1-yes, 0-no)?  &check_dir_exist;
             %put ASSIGN FILEREF SUCCESSFUL (0-yes, 1-no)?  &check_file_assign;

           %end;

           /*
           ** RETURN THE RESULT.  TRIM THE LEADING DELIMITER OFF THE FRONT OF THE RESULTS.
           */
           %if "&result" ne "" %then %do;
             %qsubstr(%nrbquote(&result),2)
           %end;

         %mend;

       OUTPUT

         %let dirlst= %file_list(iPath=c:\parent\child_2\);

         %put &=dirlst;

         DIRLST=file1.txt|file2.txt

         /*%put %file_list(iPath=e:\SASDev);*/
         /*%put %file_list(iPath=e:\SASDev\,iFiles_only=1);*/
         /*%put %file_list(iPath=e:\sasdev\,iFiles_only=1,iFilter=auto);*/


     11.  Windows: Another Windows directory into macro variable;
    =============================================================

         %macro DIRLISTWIN( PATH      /* Windows path of directory to examine                             */
                     , MAXDATE=  /* [optional] maximum date/time of file to report                   */
                     , MINDATE=  /* [optional] minimum date/time of file to report                   */
                     , MAXSIZE=  /* [optional] maximum size of file to report (bytes)                */
                     , MINSIZE=  /* [optional] minimum size of file to report (bytes)                */
                     , OUT=      /* [optional] name of output file containing results of %DIRLISTWIN */
                     , REPORT=Y  /* [optional] flag controlling report creation                      */
                     , REPORT1=N /* [optional] flag controlling 1-line report creation               */
                     , SUBDIR=Y  /* [optional] include subdirectories in directory processing        */
                     ) ;

       /* PURPOSE: create listing of files in specified directory to make evident
        *
        * NOTE:    %DIRLISTWIN is designed to be run on SAS installations using the Windows O/S
        *
        * NOTE:    &PATH must contain valid Windows path, e.g., 'c:' or 'c:\documents and settings'
        *
        * NOTE:    &MAXDATE and &MINDATE must be SAS date/time constants in one of the following formats:
        *             'ddMONyy:HH:MM'dt *** datetime constant ***
        *             'ddMONyy'd        *** date     constant ***
        *             'HH:MM't          *** time     constant ***
        *
        * NOTE:    if &SUBDIR = Y then all subdirectories of &PATH will be searched
        *          otherwise, only the path named in &PATH will be searched
        *
        * NOTE:    uses Windows pipe option on file reference
        *
        * NOTE:    if %DIRLISTWIN is used successively in the same job, then
        *             the report will contain the cumulative directory listing of all directories searched
        *             a separate &OUT dataset will be created for each %DIRLISTWIN invocation
        *
        * USAGE:
        *  %DIRLISTWIN( c:/data1 )
        *  %DIRLISTWIN( c:/data1, MINDATE='01JAN04:00:00:00'dt, MAXDATE='16MAR04:23:59:59'dt )
        *  %DIRLISTWIN( c:/data1, MINDATE='00:00:00't, MAXDATE='23:59:59't, MINSIZE=1000000 )
        *  %DIRLISTWIN( d:/data2, REPORT=Y )
        *  %DIRLISTWIN( d:, OUT=LIBNAME.DSNAME, REPORT=N )
        *  %DIRLISTWIN( d:/documents and settings/robett/my documents/my sas files/v8 )
        *
        * ALGORITHM:
        *  use Windows pipe with file reference to execute 'dir' command to obtain directory contents
        *  parse pipe output as if it were a file to extract file names, other info
        *  [optional] select files that are within the time interval [ &MINDATE, &MAXDATE ]
        *  [optional] select files that are at least as large as &MINSIZE bytes and no larger than &MAXSIZE
        *  sort records by owner, path, filename
        *  [optional] create report of files per owner/path if requested
        *  [optional] create 1-line report of files per owner/path if requested
        */

       %let DELIM   = ' ' ;
       %let REPORT  = %eval( %upcase( &REPORT ) = Y ) ;
       %let REPORT1 = %eval( %upcase( &REPORT1 ) = Y ) ;

       %if %upcase( &SUBDIR ) = Y %then %let SUBDIR = /s ; %else %let SUBDIR = ;

       /*============================================================================*/
       /* external storage references
       /*============================================================================*/

       /* run Windows "dir" DOS command as pipe to get contents of data directory */

       filename DIRLIST pipe "dir /-c /q &SUBDIR /t:c ""&PATH""" ;

       /*############################################################################*/
       /* begin executable code
       /*############################################################################*/

       /* use Windows pipe to recursively find all files in &PATH
        * parse out extraneous data, including unreadable directory paths
        * process files >= &MINSIZE in size
        *
        * directory list structure:
        *    "Directory of" record precedes listing of contents of directory:
        *
        *    Directory of <volume:> \ <dir1> [ \ <dir2>\... ]
        *    mm/dd/yy hh:mm:ss [AM|PM] ['<DIR>' | size ] filename.type
        *
        *    example:
        *
        *       Volume in drive C is WXP
        *       Volume Serial Number is 18C2-3BAA
        *
        *       Directory of C:\Documents and Settings\robett\My Documents\My SAS Files\V8\Test
        *
        *       05/21/03  10:58 AM    <DIR>          CARYNT\robett          .
        *       05/21/03  10:58 AM    <DIR>          CARYNT\robett          ..
        *       12/24/03  10:22 AM    <DIR>          CARYNT\robett          Codebook
        *       04/23/01  02:42 PM               387 CARYNT\robett          printCharMat.sas
        *       10/09/03  11:35 AM             20582 CARYNT\robett          test.log
        *       10/28/03  08:02 AM             58682 CARYNT\robett          test.lst
        *       10/09/03  11:35 AM              1575 CARYNT\robett          test.sas
        */

       data dirlist ;
          length path filename $255 line $1024 owner $17 temp $16 ;
          retain path ;

          infile DIRLIST length=reclen ;
          input line $varying1024. reclen ;

          if reclen = 0 then delete ;

          if scan( line, 1, &DELIM ) = 'Volume'  | /* beginning of listing */
             scan( line, 1, &DELIM ) = 'Total'   | /* antepenultimate line */
             scan( line, 2, &DELIM ) = 'File(s)' | /* penultimate line     */
             scan( line, 2, &DELIM ) = 'Dir(s)'    /* ultimate    line     */
          then delete ;

          dir_rec = upcase( scan( line, 1, &DELIM )) = 'DIRECTORY' ;

          /* parse directory     record for directory path
           * parse non-directory record for filename, associated information
           */

          if dir_rec
          then
             path = left( substr( line, length( "Directory of" ) + 2 )) ;
          else do ;
             date = input( scan( line, 1, &DELIM ), mmddyy8. ) ;

             time = input( scan( line, 2, &DELIM ), time5. ) ;

             post_meridian = ( scan( line, 3, &DELIM ) = 'PM' ) ;

             if post_meridian then time = time + '12:00:00'T ; /* add 12 hours to represent on 24-hour clock */

             temp = scan( line, 4, &DELIM ) ;

             if temp = '<DIR>' then size = 0 ; else size = input( temp, best. ) ;

             owner = scan( line, 5, &DELIM ) ;

             /* scan delimiters cause filename parsing to require special treatment */

             filename = scan( line, 6, &DELIM ) ;

             if filename in ( '.' '..' ) then delete ;

             ndx = index( line, scan( filename, 1 )) ;

             filename = substr( line, ndx ) ;
          end ;

          /* date/time filter */

          %if %eval( %length( &MAXDATE ) + %length( &MINDATE ) > 0 )
          %then %do ;
             if not dir_rec
             then do ;
                datetime = input( put( date, date7. ) || ':' || put( time, time5. ), datetime13. )  ;

                %if %length( &MAXDATE ) > 0 %then %str( if datetime <= &MAXDATE ; ) ;
                %if %length( &MINDATE ) > 0 %then %str( if datetime >= &MINDATE ; ) ;
             end ;
          %end ;

          /* size filter */

          %if %length( &MAXSIZE ) > 0 %then %str( if size <= &MAXSIZE ; ) ;
          %if %length( &MINSIZE ) > 0 %then %str( if size >= &MINSIZE ; ) ;

          drop dir_rec line ndx post_meridian temp ;
       run ;

       proc sort data=dirlist out=dirlist ; by owner path filename ; run ;

       /*============================================================================*/
       /* create output dataset if requested
       /*============================================================================*/

       %if %length( &OUT ) > 0 %then %str( data &OUT ; set dirlist ; run ; ) ;

       /*============================================================================*/
       /* add data for current directory path to cumulative report dataset
       /*============================================================================*/

       proc append base=report data=dirlist ; run ;

       /*============================================================================*/
       /* break association to previous path prior to next %DIRLISTWIN invocation
       /*============================================================================*/

       filename DIRLIST clear ;

       /*============================================================================*/
       /* create report of files by owner, if requested
       /*============================================================================*/

      %if &REPORT
      %then %do ;
          title "Directory Listing" ;
          title1 "Path: &PATH" ;
          proc report center data=report headskip nowindows spacing=1 split='\' ;
             column owner path size date time filename ;

             define owner    / order   width=17        'Owner' ;
             define path     / order   width=32 flow   'Path' ;
             define size     / display format=comma19. 'Size/(bytes)' ;
             define date     / display format=mmddyy8. 'Date' ;
             define time     / display format=time5.   'Time' ;
             define filename / display width=32 flow   'File Name' ;
          run ;
          title ;

       %end ;

      %if &REPORT1
      %then %do ;
          /* create 1-line report: truncate path to fit landscape layout */

          data report1( keep= owner path1 size ) ;
             length path1 $287 ; /* 255 chars from above + 32 chars max filename */
             set report ;

             path1 = catx( '\', path, filename ) ;

             path1 = left( reverse( substr( left( reverse( path1 )), 1, 80 ))) ;
          run ;

          title "Directory Listing" ;
          title1 "Path: &PATH" ;
          proc report nocenter data=report1 headskip nowindows spacing=1 split='/' ;
             column owner path1 size ;

             define owner / order width=17 'Owner' ;
             define path1 / order width=80 'Path' ;
             define size  / display format=comma19. 'Size/(bytes)' ;
          run ;
          title ;

       %end ;
       %mend DIRLISTWIN ;

       %DIRLISTWIN(c:\parent\child_2);

       proc print data=report;
         format DATE date9.    TIME hhmm8.;
       run;quit;

    OUTPUT

           PATH           FILENAME        OWNER            DATE        TIME    SIZE

     c:\parent\child_2                                        .           .      .
     c:\parent\child_2    file1.txt    BEAST\beast    25JAN2020       14:14      7
     c:\parent\child_2    file2.txt    BEAST\beast    25JAN2020       14:14      7



      12.  Unix: Return the path up one level

      =========================================

        /* T000760 MACRO TO RETURN PATH UP ONE LEVEL UNIX  */
           %macro lvlup1(fid);
              /* moves up a level in a path
                 if input fid=/home/rdeange/local/met
                 then the function returs /home/rdeange/local/
                 usage %let upo=%lvlup1(&fid);
              */
              %let dif=%qleft(%sysfunc(reverse(&fid)));
              %let up1=%qsubstr(&dif,%index(%str(&dif),%str(/)));
              %let rup=%qleft(%sysfunc(reverse(&up1)));
              &rup.
           %mend lvlup1;
           %put %lvlup1(/home/regusers/local/oto);


      13.  All Platforms does a file exist

      ====================================

        /* T002390 DOES A FOLDER EXIST MACRO AND DATASTEP */
           %let rc1 = %sysfunc(filename(fileref,/home/regusers/utl));
           %let xis = %sysfunc(fexist(&fileref));
           %put &xis;  /* 1 means it exists */
           data _null_;
               rc =filename('fileref','/home/regusers/utl');
               xis=fexist('fileref');
               put xis=;
           run;


      14.  Given a full path extract the folder and filename with and without extension
      =================================================================================

        Given a full path extract the folder and filename with and without extension

        Parse the full path

          %let full_path=d:/sd1/utl_class.sas7bdat;

          MACRO               RULE                              RESULT

          mGetFileNoExt      * get file without extention   ==> utl_class
          mGetFileWithExt    * get file with extention      ==> utl_class.sas7bdate
          mGetFolder         * get folder stem              ==> d:/sd1/

         Four Solutions

          1. macro variable 'fill_path';
          2. datastep  'full path';
          3. datastep  FCMP;
          4. datastep  DOSUBL;

          (some of these solutions predate scan enhancements, ie 'call scan, neg scanning'.

         *
         _ __   __ _ _ __ ___  ___   _ __ ___   __ _  ___ _ __ ___   __   ____ _ _ __
        | '_ \ / _` | '__/ __|/ _ \ | '_ ` _ \ / _` |/ __| '__/ _ \  \ \ / / _` | '__|
        | |_) | (_| | |  \__ \  __/ | | | | | | (_| | (__| | | (_) |  \ V / (_| | |
        | .__/ \__,_|_|  |___/\___| |_| |_| |_|\__,_|\___|_|  \___/    \_/ \__,_|_|
        |_|
        ;

          %let full_path=d:/sd1/utl_class.sas7bdat;

          mGetFileNoExt      * get file without extention   ==> utl_class
          mGetFileWithExt    * get file with extention      ==> utl_class.sas7bdate
          mGetFolder         * get folder stem              ==> d:/sd1/

           Test cases

            %let full_path=d:/sd1/utl_class.sas7bdat;

            %put %mGetFileNoExt  (&full_path);
            %put %mGetFileWithExt(&full_path);
            %put %mgetFolder     (&full_path);


            %macro mGetFileNoExt(pth)/des="macro variable get file name without extention";
               %qscan(&pth,-2,%str(./\))
             %mend mGetFileNoExt;

            %macro mGetFileWithExt(pth)/des="macro variable get file name without extention";
               %qscan(&pth,-1,%str(/\))
            %mend mGetFileWithExt;

            %macro mgetFolder(pth)/des="macro variable get folder name";
               %let revstr=%qleft(%qsysfunc(reverse(&pth)));
               %let cutstr=%qsubstr(&revstr,%qsysfunc(indexc(&revstr,%str(/\))));
               %let gotstm=%qleft(%qsysfunc(reverse(&cutstr)));
               %str(&gotstm)
            %mend mgetFolder;

        *                                _       _
         _ __   __ _ _ __ ___  ___    __| | __ _| |_ __ _  __   ____ _ _ __
        | '_ \ / _` | '__/ __|/ _ \  / _` |/ _` | __/ _` | \ \ / / _` | '__|
        | |_) | (_| | |  \__ \  __/ | (_| | (_| | || (_| |  \ V / (_| | |
        | .__/ \__,_|_|  |___/\___|  \__,_|\__,_|\__\__,_|   \_/ \__,_|_|
        |_|
        ;


          %getFileNoExt      * get file without extention   ==> utl_class
          %getFileWithExt    * get file with extention      ==> utl_class.sas7bdate
          %getFolder         * get folder stem              ==> d:/sd1/

          data _null_;

            full_path="d:/sd1/utl_class.sas7bdat";

            getFileNoExt   =  %getFileNoExt  (full_path);
            getFileWithExt =  %getFileWithExt(full_path);
            getFolder      =  %getFolder     (full_path);

            put (_all_) (= $ /);

          run;quit;

          FULL_PATH      = d:/sd1/utl_class.sas7bdat
          GETFILENOEXT   = utl_class
          GETFILEWITHEXT = utl_class.sas7bdat
          GETFOLDER      = utl_class.sas7bdat

            %macro getFileNoExt(pth)/des="datastep variable get file name without extention";
               scan(full_path,-2,"./\")
             %mend getFileNoExt;

            %macro getFileWithExt(pth)/des="datastep variable get file name without extention";
               scan(full_path,-1,"/\")
            %mend GetFileWithExt;


            %macro getFolder(pth)/des="macro variable get folder name";
             left(reverse(left(scan(reverse(full_path),1,"/\"))))
            %mend getFolder;

        * __
         / _| ___ _ __ ___  _ __
        | |_ / __| '_ ` _ \| '_ \
        |  _| (__| | | | | | |_) |
        |_|  \___|_| |_| |_| .__/
                           |_|
        ;

          getFileNoExt      * get file without extention   ==> utl_class
          getFileWithExt    * get file with extention      ==> utl_class.sas7bdate
          getFolder         * get folder stem              ==> d:/sd1/

          data _null_;

            full_path="d:/sd1/utl_class.sas7bdat";

            getFileNoExt   =  getFileNoExt  (full_path);
            getFileWithExt =  getFileWithExt(full_path);
            getFolder      =  getFolder     (full_path);

            put (_all_) (= $ /);

          run;quit;

          FULL_PATH      = d:/sd1/utl_class.sas7bdat
          GETFILENOEXT   = utl_class
          GETFILEWITHEXT = utl_class.sas7bdat
          GETFOLDER      = utl_class.sas7bdat

        options cmplib=work.funcs;

        proc fcmp outlib=work.funcs.parsepath;

          function getFolder(expr $) $200;
            length result $200;
            result=left(reverse(left(scan(reverse(expr),1,"/\"))));
            return(result);
          endsub;

          function getFileNoExt(expr $) $200;
            length result $200;
            result=scan(expr,-2,"./\");
            return(result);
          endsub;

          function getFileWithExt(expr $) $200;
            length result $200;
            result=left(reverse(left(scan(reverse(full_path),1,"/\"))));
            return(result);
          endsub;

        run;

        *    _                 _     _
          __| | ___  ___ _   _| |__ | |
         / _` |/ _ \/ __| | | | '_ \| |
        | (_| | (_) \__ \ |_| | |_) | |
         \__,_|\___/|___/\__,_|_.__/|_|

        ;

          getFileNoExt      * get file without extention   ==> utl_class
          getFileWithExt    * get file with extention      ==> utl_class.sas7bdate
          getFolder         * get folder stem              ==> d:/sd1/

          %symdel file /nowarn;
          data _null_;
            length file $32;
            full_path="d:/sd1/utl_class.sas7bdat";
            call symputx('full_path',full_path);

            * flexible - you can use fcmp, macro or datastep code;
            file=dosubl('
              %let file=%sysfunc(getFileNoExt  (&full_path)));
              %put &=full_path;
              /* more code datasteps or procedures */
              ');
            file=symget("file");
            put (_all_) (= $ /);

          run;quit;


          %symdel file /nowarn;
          data _null_;
            length file $32;
            full_path="d:/sd1/utl_class.sas7bdat";
            call symputx('full_path',full_path);

            * flexible - you can use fcmp, macro or datastep code;
            file=dosubl('
              data _null_;
                 file=getFileNoExt(symget("full_path"));
                 call symputx("file",file);
              run;quit;
              /* more code datasteps or procedures */
              ');
            file=symget("file");
            put (_all_) (= $ /);

          run;quit;


    15. Delete empth directory
    ===========================

    %macro utl_delmty(folder)/des="Delete empty folders";
      filename dir "&folder";
      data _null_;
         dir_id = dopen('dir');
         total_members = dnum(dir_id);
         if total_members <= 0 then do;
            rc = fdelete('dir');
            rc = fclose(dir_id);
         end;
       run;quit;
     %mend utl_delmty;

   %utl_delmty(c:/parent/child_3);
   
   
    16. Detailed-List-of-Directories-and-Files-in-Linux                                                                                 
        https://tinyurl.com/y59u45t7                                                                                                    
                                                                                                                                        
    SAS Doc                                                                                                                             
    https://tinyurl.com/y59u45t7                                                                                                        
    https://communities.sas.com/t5/SAS-Communities-Library/Detailed-List-of-Directories-and-Files-in-Linux/ta-p/582142                  
                                                                                                                                        
    TomRVincent                                                                                                                         
    https://communities.sas.com/t5/user/viewprofilepage/user-id/144199                                                                  
                                                                                                                                        
                                                                                                                                        
    %let directory=&_r/uml/;                                                                                                            
                                                                                                                                        
    filename tmp pipe "ls -mlR --full-time -1 &directory";                                                                              
                                                                                                                                        
    data want12;                                                                                                                        
          infile tmp dlm="¬";                                                                                                           
          length dir_line $2000;                                                                                                        
          input dir_line;                                                                                                               
    run;                                                                                                                                
                                                                                                                                        
    proc sql noprint;                                                                                                                   
          select max(countw(dir_line, ' ')) into :maxwords trimmed from want12;                                                         
    quit;                                                                                                                               
                                                                                                                                        
    data want;                                                                                                                          
          set want12;                                                                                                                   
          format dir $200.;                                                                                                             
          retain dir;                                                                                                                   
          words = countw(dir_line, ' ');                                                                                                
                                                                                                                                        
          if find(dir_line,"/") then                                                                                                    
                dir=dir_line;                                                                                                           
          array parsed_vars(*) $200 new_var1-new_var&maxwords;                                                                          
          i=1;                                                                                                                          
                                                                                                                                        
          do while(i<words or scan(dir_line, i, " ") ne "" or i=1);                                                                     
                if find(dir_line,"/")=0 and scan(dir_line, i, " ") ne ""  and i<>1 then                                                 
                      parsed_vars(i) =scan(dir_line, i, " ");                                                                           
                else if i=1 then parsed_vars(i)=dir_line;                                                                               
                i+1;                                                                                                                    
          end;                                                                                                                          
                                                                                                                                        
          if not find(dir_line,"/") and words>5 then                                                                                    
                do;                                                                                                                     
                      file_mode=new_var1;                                                                                               
                      number_of_links=new_var2;                                                                                         
                      owner_name=new_var3;                                                                                              
                      group_name=new_var4;                                                                                              
                      bytes=input(new_var5,comma12.);                                                                                   
                      date_last_modified=new_var6;                                                                                      
                      time_modified=new_var7;                                                                                           
                                                                                                                                        
                      if length(scan(dir_line, 8, " "))>0                                                                               
                            and substr(dir_line,1,1)='-'                                                                                
                                  then file_name=substr(dir_line,                                                                       
                                  find(dir_line,scan(dir_line, 8, " "))                                                                 
                                  +length(scan(dir_line, 8, " "))+1,                                                                    
                                  length(dir_line)-find(dir_line,scan(dir_line, 8, " ")));                                              
                end;                                                                                                                    
    run;                                                                                                                                
                                                                                                                                        
                                                                                                                                        
                                                





