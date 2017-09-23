��XXTEA�����㷨ΪSQLiteʵ���˼��ܹ���

SQLite��һ���ܺ��õ�Ƕ��ʽ���ݿ⡣��ϧ���в������SQLite����Ѱ汾���߱����ܹ��ܡ����������Ͽ���һ����WinCryptʵ�ּ��ܹ��ܵİ汾����ϧ̫������Windowsƽ̨�ˡ��⼸����ʱ�䣬�Լ�����XXTEA�㷨��SQLite3.3.7�汾�Ļ�����ʵ���˼��ܹ��ܡ�ѡ��XXTEA��Ҫ����ΪXXTEA�㷨�ٶȺܿ죬��������ɵ�Ӱ����Խ�С��
��SQLite��һ���ǵ���������Ƚ϶࣬�м��������һ�����٣��������ڷų�Դ�������ҷ�������Ĺ����ļ�����VC2005������������������������ʱ��Ҫ���Ǽ���SQLITE_HAS_CODEC�ꡣ����ֻ�Ǵ��Բ��Թ�������֤100����BUG���߼�����

////////////////////////////////////////////////////////////////////////////////////
ʵ�����룺

#include "..\sqlite3xxtea\sqlite3.h"

#define OUTERR     if ( SQLITE_OK != sqlite3_errcode(db))\
    {\
    errmsg = (TCHAR *)sqlite3_errmsg16(db);\
    wprintf(errmsg);\
    }


void test(){

    sqlite3 *db;
    sqlite3_open16(L"a.zdb", &db);

    sqlite3_key(db,"123456",6); //������123456�����ݿ⣬������벻�Իᱨ��

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


    sqlite3_rekey(db,"abcdefgh",8); //�޸�����Ϊabcdefgh

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



    sqlite3_rekey(db,"1q2w3e4r",8);//�޸�����Ϊ1q2w3e4r

//    sqlite3_rekey(db,"",0); //������ݿ�����
    sqlite3_close (db);
}
