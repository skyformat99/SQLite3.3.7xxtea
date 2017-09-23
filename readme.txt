用XXTEA加密算法为SQLite实现了加密功能

SQLite是一个很好用的嵌入式数据库。可惜美中不足的是SQLite的免费版本不具备加密功能。曾经在网上看到一个用WinCrypt实现加密功能的版本，可惜太依赖于Windows平台了。这几日有时间，自己就用XXTEA算法在SQLite3.3.7版本的基础上实现了加密功能。选择XXTEA主要是因为XXTEA算法速度很快，对性能造成的影响相对较小。
用SQLite的一般是单机版软件比较多，有加密需求的一定不少，所以现在放出源代码与大家分享。代码的工程文件是用VC2005，如果用其它编译器，编译的时候不要忘记加上SQLITE_HAS_CODEC宏。代码只是粗略测试过，不保证100％无BUG和逻辑错误。

////////////////////////////////////////////////////////////////////////////////////
实例代码：

#include "..\sqlite3xxtea\sqlite3.h"

#define OUTERR     if ( SQLITE_OK != sqlite3_errcode(db))\
    {\
    errmsg = (TCHAR *)sqlite3_errmsg16(db);\
    wprintf(errmsg);\
    }


void test(){

    sqlite3 *db;
    sqlite3_open16(L"a.zdb", &db);

    sqlite3_key(db,"123456",6); //以密码123456打开数据库，如果密码不对会报错。

    TCHAR  sql[4096];
    TCHAR * errmsg;

    sqlite3_stmt * stmt;
    void * pzTail;


    memset(sql, 0, sizeof(sql));
    lstrcpy(sql, L"CREATE TABLE test ([ID] INTEGER PRIMAY KEY DEFAULT 0, [Name] TEXT);");

    sqlite3_prepare16(db, sql, 4096, &stmt, (const void * *)& pzTail );
    sqlite3_step (stmt);
    sqlite3_finalize (stmt);
    OUTERR;


    sqlite3_rekey(db,"abcdefgh",8); //修改密码为abcdefgh

   memset(sql, 0, sizeof(sql));
   lstrcpy(sql, L"BEGIN;");

   sqlite3_prepare16(db, sql, 4096, &stmt, (const void * *)& pzTail );
   sqlite3_step (stmt);
   sqlite3_finalize (stmt);
   OUTERR;

   for (int i = 0; i < 128; i++)
   {

        memset(sql, 0, sizeof(sql));
        lstrcpy(sql, L"INSERT INTO test ([ID] , [Name] ) VALUES(?, ?);");

        sqlite3_prepare16(db, sql, 4096, &stmt, (const void * *)& pzTail );

        sqlite3_bind_int(stmt, 1, i);
        sqlite3_bind_text16(stmt, 2, sql, lstrlen(sql)*2,   SQLITE_STATIC);


        sqlite3_step (stmt);
        sqlite3_finalize (stmt);
        OUTERR; 


   }

   memset(sql, 0, sizeof(sql));
   lstrcpy(sql, L"COMMIT;");
   OUTERR;

   sqlite3_prepare16(db, sql, 4096, &stmt, (const void * *)& pzTail );
   sqlite3_step (stmt);
   sqlite3_finalize (stmt);
   OUTERR;



    sqlite3_rekey(db,"1q2w3e4r",8);//修改密码为1q2w3e4r

//    sqlite3_rekey(db,"",0); //清除数据库密码
    sqlite3_close (db);
}
