[sqlcipher windows下编译](https://blog.csdn.net/harborian/article/details/89924346)


# sqlcipher自己编译
## sqlcipher是sqlite的加密版本，提供源代码，但是在编译时，尤其是在编译windows版本时，需要一些技巧。
- sqlite和sqlcipher的下载
  - sqlite源码下载
    - sqlite可以从https://www.sqlite.org上下载最新版本。amalgamation版即为整合版，把所有的c源码都集中到了sqlite3.c文件中去，工程源代码只包含sqlite3.c, sqlite3.h,     shell.c,sqlite3ext.h。而完整的需要使用tcl编译后，才能生成这4个文件。sqlcipher是在完整的基础上开发的。
    - 如果需要下载之前的某个版本，使用fossil更加简便。sqlite网站上提供了fossil的windows，mac和linux版，根据自己使用的平台下载即可。

    - fossil clone http://www.sqlite.org/cgi/src sqlite.fossil
      - 使用上述命令，将sqlite源代码保存到sqlite.fossil文件。

    - fossil open sqlite.fossil
      - 将当前最新版本展开到当前目录下。

    - fossil update version-3.27.2
      - 上面命令行，获取3.27.2版本到当前目录。

  - sqlcipher下载
    - 下载网址，https://www.zetetic.net/sqlcipher。当然，如果您的后面有位多金的老板，也可以直接购买编译好的版本。源码保存在github上，地址如下：
    - git clone https://github.com/sqlcipher/sqlcipher.git
   
- 编译
  - 无论是sqlite还是sqlcipher，在linux上编译都非常简单。编译sqlite，只需按照readme中的方法编译即可。编译sqlcipher，在linux（本人Ubuntu）上测试，只需配置configure时，增加  SQLITE_HAS_CODEC和SQLITE_TEMP_STORE=2，就可以正常编译出有加密功能的sqlite3可执行文件。

  - 在windows上编译sqlite
    - 假设完成的sqlite源代码保存在./sqlite文件夹中，那么，在当前路径下，创建bld文件夹，cd到该文件夹中，执行如下命令。
      ```
        mkdir bld
        cd bld
        nmake /f ..\sqlite\Makefile.msc TOP=..\sqlite
        nmake /f ..\sqlite\Makefile.msc sqlite3.c TOP=..\sqlite
        nmake /f ..\sqlite\Makefile.msc sqlite3.dll TOP=..\sqlite
        nmake /f ..\sqlite\Makefile.msc sqlite3.exe TOP=..\sqlite
        nmake /f ..\sqlite\Makefile.msc test TOP=..\sqlite
      ```
        
  - 在windows上编译sqlcipher
    - sqlcipher使用了openssl的加密算法，因此，需要下载编译openssl库文件。对openssl编译后，生成libeay32.lib和ssleay32.lib。
    - 修改Makefile.msc文件。

      - 修改TCC和RCC，增加加密配置项
      ```
      TCC = $(TCC) -DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2
      RCC = $(RCC) -DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2
      ```
      - 修改TCC和RCC，增加openssl包含路径
      ```
      TCC = $(TCC) -DSQLITE_OS_WIN=1 -I. -I$(TOP) -I$(TOP)\src -I"path to openssl include" -fp:precise
      RCC = $(RC) -DSQLITE_OS_WIN=1 -I. -I$(TOP) -I$(TOP)\src -I"path to openssl include" $(RCOPTS) $(RCCOPTS)
      ```
      - 修改LTLIBPATHS项，增加openssl库文件所在路径
        - LTLIBPATHS = $(LTLIBPATHS) /LIBPATH:'path to openssl lib'

      - 修改( S Q L I T E 3 D L L ) 和 (SQLITE3DLL)和(SQLITE3DLL)和(SQLITE3EXE)，在后面追加如下库文件
        - libeay32.lib ssleay32.lib Advapi32.lib User32.lib Gdi32.lib
        ```
        $(SQLITE3DLL):	$(LIBOBJ) $(LIBRESOBJS) $(CORE_LINK_DEP)
        $(LD) $(LDFLAGS) $(LTLINKOPTS) $(LTLIBPATHS) /DLL $(CORE_LINK_OPTS) /OUT:$@ $(LIBOBJ) $(LIBRESOBJS) $(LTLIBS) $(TLIBS) libeay32.lib ssleay32.lib Advapi32.lib User32.lib Gdi32.lib
        
        $(SQLITE3EXE):	shell.c $(SHELL_CORE_DEP) $(LIBRESOBJS) $(SHELL_CORE_SRC) $(SQLITE3H) 
        $(LTLINK) $(SHELL_COMPILE_OPTS) $(READLINE_FLAGS) shell.c $(SHELL_CORE_SRC) link $(SQLITE3EXEPDB) $(LDFLAGS) $(LTLINKOPTS) $(SHELL_LINK_OPTS) $(LTLIBPATHS) $(LIBRESOBJS) $(LIBREADLINE) $(LTLIBS) $(TLIBS) libeay32.lib ssleay32.lib Advapi32.lib User32.lib Gdi32.lib
        ```

      - 通过如上修改Makefile.msc后，使用与sqlite相同的nmake指令，可以编译出sqlite3.exe文件

- 使用
  - 在命令行输入sqlite3.exe，进入sqlite>命令行。输入.help，显示可以属于的命令，都以.开头。
  ```
  c:\temp\sqlcipher-master\build>sqlite3.exe
  SQLCipher version 3.27.2 2019-02-25 16:06:06
  Enter ".help" for usage hints.
  Connected to a transient in-memory database.
  Use ".open FILENAME" to reopen on a persistent database.
  sqlite> .open test.db
  sqlite> .tables
  sqlite> PRAGMA key='hello' ;
  sqlite> create table encrypted (id integer, name text);
  sqlite> .tables
  encrypted
  sqlite> .schema
  CREATE TABLE encrypted (id integer, name text);
  sqlite> select * from encrypted;
  sqlite> .q
  ```
