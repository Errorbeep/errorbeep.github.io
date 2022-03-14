

### 当前连接数

```sql
select count(*) from v$process;
--修改最大连接数（把processes改成500就好了）
alter system set processes = 500 scope = spfile;
--select value from v$parameter where name = 'processes';
--COMMIT;
--允许最大连接数 （默认是100）
select value from v$parameter where name = 'processes';
```

### 查询session信息
```sql
SHOW PARAMETER SESSION;
```

### 查询processes信息


```sql
SHOW PARAMETER PROCESS;
```
### 创建表空间


```sql
CREATE TABLESPACE TempSpace DATAFILE 'E:\Sorcery\Necromancy\DataBase\data\TempSpace_DATA.DBF' SIZE 50M;
```
### 创建用户


```sql
CREATE USER user1 IDENTIFIED BY user1 DEFAULT TABLESPACE TempSpace;
```
### 授予用户DBA权限


```sql
GRANT DBA TO user1;
```
### 创建表

```sql
CREATE TABLE Temp
(
  sno CHAR(10) PRIMARY KEY,
  sname VARCHAR(13) UNIQUE,
  gender CHAR(3) CHECK(gender in ('男', '女')),
  dept VARCHAR(14) NOT NULL,
  remarks VARCHAR(30) DEFAULT('三好学生')
);
SELECT * FROM Temp;
CREATE TABLE ScoreTable
(
  sno CHAR(10) PRIMARY KEY, --REFERENCES Temp(sno),
  cno CHAR(4) DEFAULT('0001'),
  score INT DEFAULT(0)
  --CONSTRAINT cno_ScoreTable FOREIGN KEY(cno)REFERENCES Subject(nationkey)
  
  --PRIMARY KEY(sno,cno),/*定义主码*/
  -- FOREIGN KEY(sno)REFERENCES Temp(sno),
  -- FOREIGN KEY(cno)REFERENCES Subject(cno)
);
INSERT INTO ScoreTable VALUES(189074010, 0001, 88);
SELECT * FROM ScoreTable ;
UPDATE ScoreTable SET cno = '0001';
CREATE TABLE Subject
(
  cno CHAR(4) PRIMARY KEY,
  cname VARCHAR2(10) UNIQUE
);
ALTER TABLE Subject MODIFY cname VARCHAR2(30);
INSERT INTO Subject VALUES('0008', '离散数学');
SELECT * FROM Subject;
```
### 参照完整性


```sql
ALTER TABLE ScoreTable ADD CONSTRAINT sno_const FOREIGN KEY(sno) REFERENCES Temp(sno); 
ALTER TABLE ScoreTable ADD CONSTRAINT cno_const FOREIGN KEY(cno) REFERENCES Subject(cno);
-- ALTER TABLE ScoreTable DROP CONSTRAINT sno_const;
```
### 修改基本表


```sql
ALTER TABLE Temp ADD CONSTRAINT temp_dept_uni UNIQUE(dept);-- 增加约束条件
ALTER TABLE Temp DROP CONSTRAINT temp_dept_uni;-- 删除约束条件
ALTER TABLE Subject ADD ctime INT DEFAULT(40);-- 增加属性
ALTER TABLE Temp ADD resume VARCHAR(30);-- 增加属性
ALTER TABLE Temp DROP COLUMN resume;-- 删除属性
ALTER TABLE ScoreTable ADD no INT;
ALTER TABLE ScoreTable DROP COLUMN no;
-- SELECT * FROM ScoreTable;
ALTER TABLE ScoreTable ADD ID INT ;
ALTER TABLE ScoreTable DROP PRIMARY KEY;
ALTER TABLE ScoreTable ADD CONSTRAINT st_prk PRIMARY KEY(ID);
ALTER TABLE Temp MODIFY sname VARCHAR2(14）; -- 这样不会去除sname的唯一约束条件
-- 如下写法是错误的：
-- ALTER TABLE Temp MODIFY sname VARCHAR2(14）UNIQUE;
```
### 删除基本表


```sql
-- DROP TABLE Temp;
```
### 向表中插入数据


```sql
INSERT INTO Temp VALUES('189074004', 'Jim', '男', '机械', '三坏学生');-- 单行插入
INSERT INTO Temp(sno, sname, dept) VALUES('189074000', 'Jenny', '机械');
INSERT INTO Temp SELECT * FROM Stu;-- 多行插入（表间拷贝）
CREATE TABLE XS AS SELECT * FROM TEMP;-- 创建表XS并插入表Temp中的所有数据
```
### 更改记录


```sql
UPDATE Temp SET remarks = '三好学生';-- 修改所有记录
UPDATE Temp SET remarks = '三坏学生' WHERE sno = '189074005'; 
UPDATE Temp SET remarks = '三坏学生', dept = '软件工程' WHERE sno = '189074006'; 
UPDATE ScoreTable SET ID = to_number(sno)  - 189074000;
SELECT * FROM ScoreTable;
```
### 删除记录


```sql
DELETE FROM Temp WHERE dept = '机械';
```
### 单表查询


```sql
SELECT 3 + 5 FROM DUAL;-- 表达式求值
SELECT * FROM Temp;
SELECT sno, sname FROM Temp;
SELECT sno AS 学号, sname AS 姓名 FROM Temp;
SELECT dept FROM Temp;
SELECT DISTINCT dept FROM Temp;-- 投影-- DISTINCT 必须放在开头
SELECT DISTINCT sno, score FROM ScoreTable;-- 根据字段sno和score去重
SELECT sno, credit FROM Temp WHERE credit > 50;
SELECT sno, credit FROM Temp WHERE credit BETWEEN 55 AND 57;
SELECT sno, credit FROM Temp WHERE credit NOT BETWEEN 55 AND 57;
SELECT sno, dept FROM Temp WHERE dept IN('计算机', '电气');
SELECT sno, dept FROM Temp WHERE dept NOT IN('计算机', '电气');
```
#### 模糊查询


```sql
SELECT sno, sname FROM Temp WHERE sname LIKE '%m%';
SELECT sno, sname FROM Temp WHERE sname LIKE '%m_';
SELECT sno, sname FROM Temp WHERE sname LIKE 'ABC\_DEF' ESCAPE '\';
--  在mysql中不用加“ESCAPE '\'”
 
SELECT sno, credit FROM Temp WHERE credit IS NULL;
SELECT sno, credit FROM Temp WHERE credit IS NOT NULL;
```
#### 组函数


```sql
SELECT COUNT(*) AS 总人数 FROM Temp;
SELECT COUNT(DISTINCT dept) AS 院系数 FROM Temp;
SELECT AVG(credit) AS 电气专业平均学分 FROM Temp WHERE dept = '电气';
SELECT MAX(credit) AS 最高学分 FROM Temp;
SELECT MIN(credit) AS 最低学分 FROM Temp;-- 不包括NULL
SELECT SUM(credit) AS 学分之和 FROM Temp;
```
#### 分组查询


```sql
SELECT dept AS 系别, COUNT(*) AS 人数 FROM Temp GROUP BY dept;
SELECT dept AS 系别, AVG(credit) AS 均分 FROM Temp GROUP BY dept;
```
#### HAVING子句


```sql
-- WHERE作用于基本表或视图，从中选择满足条件的元组；
-- HAVING短语作用于组，从中选择满足条件的组。 
SELECT dept, COUNT(*) FROM Temp GROUP BY dept HAVING COUNT(*) > 2;
```
#### 排序


```sql
SELECT * FROM Temp ORDER BY sno ASC, class DESC;
```
 

### 多表查询

#### 无条件连接


```sql
SELECT * FROM Temp, ScoreTable;
SELECT ctime FROM Subject a,ScoreTable b WHERE a.cno = b.cno AND b.sno = '189074004';
```
#### 笛卡尔积


```sql
SELECT * FROM Temp CROSS JOIN ScoreTable;
```
#### theta连接


```sql
SELECT Temp.*, ScoreTable.* FROM Temp join ScoreTable ON Temp.sno = ScoreTable.sno;
SELECT * FROM Temp JOIN ScoreTable ON Temp.sno = ScoreTable.sno;
SELECT * FROM Temp JOIN ScoreTable ON Temp.sno = ScoreTable.sno WHERE dept = '计算机' ORDER BY Temp.sno ASC;
SELECT * FROM Temp a JOIN ScoreTable b ON a.sno = b.sno JOIN Subject c ON b.cno = c.cno WHERE b.score >= 80;
```
#### 自连接


```sql
SELECT * FROM ScoreTable a JOIN ScoreTable b ON a.sno <> b.sno WHERE b.sno = '189074001' AND a.score > b.score;
SELECT * FROM Temp a JOIN Temp b ON a.sno <> b.sno WHERE a.sname = 'Tom' AND a.credit < b.credit;
-- 与以下语句结果相同：SELECT * FROM Temp a, Temp b WHERE a.sname = 'Tom' AND a.credit < b.credit;
```
#### 右连接

右(外)连接，右表的记录将会全部表示出来，而左表只会显示符合搜索条件的记录。左表记录不足的地方均为NULL。左(外)连接与之相反。

```sql
SELECT * FROM ScoreTable a RIGHT JOIN Temp b ON a.sno = b.sno WHERE a.sno is NULL;
```
#### 自然连接


```sql
SELECT * FROM Temp JOIN ScoreTable USING(sno);
SELECT * FROM Temp NATURAL JOIN ScoreTable;
SELECT * FROM Temp NATURAL JOIN ScoreTable NATURAL JOIN Subject;
-- NATURAL 联接中使用的列不能有限定词
-- 误：SELECT * FROM Temp a NATURAL JOIN ScoreTable b NATURAL JOIN Subject c WHERE a.sno = '189074001';
-- 误：SELECT * FROM Temp a NATURAL JOIN ScoreTable b WHERE a.sno = '189074001';
-- 正：SELECT * FROM Temp a JOIN ScoreTable b ON a.sno = b.sno JOIN Subject c ON b.cno = c.cno WHERE a.sno = '189074001';
```
### 子查询

#### 子查询放在WHERE子句中：


```sql
SELECT ctime FROM Subject WHERE Subject.cno = (SELECT cno FROM ScoreTable WHERE ScoreTable.sno = '189074004');-- 逻辑清晰但速度慢
-- 同：SELECT ctime FROM Subject a, ScoreTable b WHERE a.cno = b.cno AND b.sno = '189074004';
SELECT * FROM Temp WHERE sno IN (SELECT sno FROM ScoreTable WHERE cno  = (SELECT cno FROM Subject WHERE cname = '大学英语'));
-- 同：SELECT * FROM Temp a, ScoreTable b, Subject c WHERE a.sno = b.sno AND b.cno = c.cno AND c.cname = '大学英语'; 
-- 同：SELECT * FROM Temp a JOIN ScoreTable b ON a.sno = b.sno JOIN Subject c ON b.cno = c.cno WHERE cname = '大学英语';
-- 同：SELECT * FROM Temp WHERE sno IN (SELECT sno FROM ScoreTable WHERE cno IN (SELECT cno FROM Subject WHERE cname = '大学英语'));
SELECT * FROM ScoreTable WHERE score > (SELECT MAX(score) FROM ScoreTable WHERE cno = '0002');
SELECT * FROM ScoreTable WHERE score > ALL (SELECT score FROM ScoreTable WHERE cno = '0002');
SELECT * FROM ScoreTable WHERE score < (SELECT MIN(score) FROM ScoreTable WHERE cno = '0001');
SELECT * FROM ScoreTable WHERE score < ALL (SELECT score FROM ScoreTable WHERE cno = '0001');
SELECT * FROM ScoreTable WHERE score > ANY (SELECT score FROM ScoreTable WHERE cno = '0002');
--  == 
SELECT * FROM ScoreTable WHERE score > SOME (SELECT score FROM ScoreTable WHERE cno = '0002');
```
#### 将子查询放在SELECT中：


```sql
SELECT sno, score, (SELECT MAX(score) FROM ScoreTable)MaxSocre FROM ScoreTable;
```
#### 将子查询放在FROM中：


Oracle:
```sql
SELECT * FROM (SELECT * FROM ScoreTable ORDER BY score DESC) WHERE ROWNUM <= 5;-- 求成绩的前5名
-- 误：SELECT * FROM ScoreTable WHERE ROWNUM <= 5 ORDER BY score DESC;
```
mysql:
```sql
--  mysql  Ver 8.0.21-0ubuntu0.20.04.4 for Linux on x86_64 ((Ubuntu))
--  当前版本mysql中没有ROWNUM, 可以使用如下方法实现ROWNUM:
SELECT @ROWNUM := @ROWNUM + 1 AS ROWNUM, sno, score FROM (SELECT @ROWNUM := 0)r, ScoreTable;
--  以@符号开头的变量是会话变量。直到会话结束前它可用和可访问。
-- 因此该例的实现为：
SELECT * FROM (SELECT @ROWNUM := @ROWNUM + 1 AS ROWNUM, sno, score FROM (SELECT @ROWNUM := 0)r, ScoreTable ORDER BY score DESC)sc WHERE ROWNUM <= 5;-- 求成绩的前5名
```
### 相关子查询

所有的子查询可以分为两类，即相关子查询和非相关子查询

非相关子查询是独立于外部查询的子查询，子查询总共执行一次，执行完毕后将值传递给外部查询。
相关子查询的执行依赖于外部查询的数据，外部查询执行一行，子查询就执行一次。执行查询的时候先取得外层查询的一个属性值，然后执行与此属性值相关的子查询，执行完毕后再取得外层父查询的下一个值，依次再来重复执行子查询；故非相关子查询比相关子查询效率高。

```sql
SELECT sno, score FROM ScoreTable WHERE EXISTS(SELECT * FROM Temp WHERE sno = ScoreTable.sno AND sno = '189074001');
--  同：
SELECT DISTINCT sno, gender, dept, remarks, sname, credit FROM (SELECT a.* FROM Temp a LEFT JOIN ScoreTable b ON a.sno = b.sno WHERE b.sno IS NOT NULL)t;
​
SELECT * FROM Temp WHERE NOT EXISTS (SELECT * FROM Subject WHERE NOT EXISTS (SELECT * FROM ScoreTable WHERE Temp.sno = sno AND cno = Subject.cno));
```
 

### 传统集合运算

#### 并运算


```sql
SELECT sno, cno FROM ScoreTable WHERE cno = '0001' UNION SELECT sno, cno FROM ScoreTable WHERE cno = '0002';
-- 同：SELECT sno, cno FROM ScoreTable WHERE cno IN ('0001', '0002');
SELECT sno FROM ScoreTable UNION SELECT sno FROM Temp;
```
* UNION ALL 不去重


```sql
SELECT sno FROM Temp WHERE sno IN ('189074000', '189074001') 
UNION ALL
SELECT sno FROM Temp WHERE sno IN ('189074000', '189074002');
```
#### 交运算

* mysql只支持UNION运算，不支持INTERSECT运算

```sql
SELECT sno FROM ScoreTable WHERE cno = '0001' INTERSECT SELECT sno FROM ScoreTable WHERE cno = '0002';
-- 同：SELECT sno FROM ScoreTable WHERE cno = '0001' AND sno IN (SELECT sno FROM ScoreTable WHERE cno = '0002'); -- 在mysql中可以这样实现交运算
```
#### 差运算

* mysql中不支持MINUS运算

```sql
SELECT sno FROM Temp MINUS SELECT sno FROM ScoreTable WHERE cno = '0002';
-- ORACLE中用MINUS，SQL SERVER中用EXCEPT
```
查询比计算机专业女生人数还少的专业及女生人数


```sql
SELECT dept, girl 
FROM 
(
    SELECT dept, COUNT(CASE WHEN gender = '女' THEN '1' END) girl 
    FROM Temp 
    GROUP BY dept 
)
WHERE girl < 
(
       SELECT girl 
       FROM 
       (
          SELECT dept, COUNT(CASE WHEN gender = '女' THEN '1' END) girl 
          FROM Temp 
          GROUP BY dept 
       )
      WHERE dept = '计算机'
                                        
);

--  以下代码不正确（当存在某专业女生人数为０时...）
SELECT dept, COUNT(*) FROM Temp WHERE gender = '女' GROUP BY dept HAVING COUNT(*) <= (SELECT COUNT(*)FROM Temp WHERE dept = '计算机' AND gender = '女');

SELECT DISTINCT dept 
FROM Temp 
MINUS 
(
    SELECT dept 
    FROM Temp 
    WHERE gender = '女' 
    GROUP BY dept 
    HAVING COUNT(*) >= (
        SELECT COUNT(*)
        FROM Temp 
        WHERE dept = '计算机' AND gender = '女'
    )
);
```
 

### 分页查询


```sql
--  Oracle:
SELECT * FROM (SELECT ROWNUM rn, sno, sname, dept,  credit FROM Temp WHERE ROWNUM <= currentPage * pageSize)tmp WHERE rn > (currentPage -1) * pageSize;
​
--  mysql:
SELECT * FROM (SELECT @ROWNUM := @ROWNUM + 1 AS ROWNUM, sno, sname, credit FROM (SELECT @ROWNUM := 0)r, Temp)page WHERE ROWNUM BETWEEN ((currentPage - 1) * pageSize + 1) AND (currentPage * pageSize);
```
### 索引


```sql
CREATE INDEX scoreT_score_idx ON ScoreTable(score);
-- CREATE UNIQUE INDEX temp_sname_uniq_idx ON Temp(sname);
CREATE INDEX scoreT_idx ON ScoreTable(sno, cno);-- 复合索引
CREATE INDEX sub_cn_ct_idx ON Subject(cno ASC, ctime DESC);-- 指定索引值的排列顺序
```
### 视图


```sql
CREATE OR REPLACE VIEW temp_CS_view
AS 
SELECT * FROM Temp WHERE dept = '计算机';
CREATE OR REPLACE VIEW score_sno_avg_view（sno, avg_score）-- 要使用别名
AS
SELECT sno, AVG(score) FROM ScoreTable GROUP BY sno;
```
#### 使用视图


```sql
SELECT * FROM score_sno_avg_view WHERE avg_score > 85;
-- UPDATE score_sno_avg_view SET sno = 100 WHERE avg_score = 88;-- 此视图的数据操纵操作非法
UPDATE temp_CS_view SET sname = 'Amilia' WHERE sname = 'Amy';-- 1行已更新。
```
#### 删除视图


```sql
DROP view temp_CS_view;
```
### 同义词

* mysql中没有synonym

```sql
CREATE OR REPLACE PUBLIC SYNONYM score_s_a FOR score_sno_avg_view;-- 公有同义词
CREATE OR REPLACE SYNONYM score_sa FOR score_sno_avg_view;-- 私有同义词
```
### 序列

* mysql中无SEQUENCE

```sql
CREATE SEQUENCE id_seq
START WITH 189074012
INCREMENT BY 1
MAXVALUE 189074999
CACHE 50;

INSERT INTO Temp VALUES(id_seq.NEXTVAL, 'Larry', '男', '计算机', '三无学生', '10', '1');-- 189074012
SELECT id_seq.CURRVAL FROM DUAL;
SELECT id_seq.NEXTVAL FROM DUAL;
INSERT INTO Temp VALUES(id_seq.NEXTVAL, 'Harry', '男', '计算机', '三沙学生', '15', '1');-- 189074014
```
#### 删除序列


```sql
DROP SEQUENCE id_seq;
```
### PL-SQL编程

* MySQL不支持anonymous code block

```sql
/
SET SERVEROUTPUT ON;
DECLARE
  stuno Temp.sno%TYPE;
  stuname Temp.sname%TYPE;
BEGIN
  SELECT sno, sname INTO stuno, stuname FROM Temp WHERE sno = '189074005';
  DBMS_OUTPUT.PUT_LINE(stuno||stuname);
END;
/
SET SERVEROUTPUT ON;
DECLARE
  stu Temp%ROWTYPE;
BEGIN
  SELECT *  INTO stu FROM Temp WHERE sno = '189074005';
  DBMS_OUTPUT.PUT_LINE(stu.sno||stu.sname||stu.dept);
END;
/
set SERVEROUTPUT ON;
DECLARE
    n number;
    result number;
BEGIN
    n:=0;   result:=0;
    while n<=100 loop
        result:=result+n;
        n:=n+1;
    end loop;
    dbms_output.put_line('结果是'||result);
END;
/
```
### PROCEDURE 存储过程

#### 创建存储过程(Oracle)


```sql
CREATE OR REPLACE PROCEDURE sp_calcsum1
as
    n number;   result number;
BEGIN
    n:=0;   result:=0;
    while n<=100 loop
        result:=result+n;
        n:=n+1;
    end loop;
    dbms_output.put_line('结果是'||result);
END;
/*
   在view（视图）中，只能使用as；
   在corsor（游标）中，只能使用is；
   对于procedure（存储过程）, function（函数）, package（程序包）来说，as和is没有区别。
*/
```
#### 执行存储过程(Oracle)


```sql
execute sp_calcsum1;

BEGIN 
sp_calcsum1;
END;
/
CREATE OR REPLACE PROCEDURE del_sc(stuNo IN Temp.sno%TYPE, crsNo IN Subject.cno%TYPE)
IS /*不能加DECLARE*/
BEGIN 
    DELETE FROM ScoreTable WHERE sno = stuNo AND cno = crsNo;
EXCEPTION 
    WHEN NO_DATA_FOUND THEN 
        DBMS_OUTPUT.put_line('没找到数据！');
    WHEN OTHERS THEN
        DBMS_OUTPUT.put_line('产生异常！');
END;        
/
EXECUTE del_sc('189074000', '0008');
SELECT * FROM ScoreTable;
​
/
CREATE OR REPLACE PROCEDURE sel_score(stuNo IN Temp.sno%TYPE, crsNo IN Subject.cno%TYPE, sco OUT ScoreTable.score%TYPE)
IS
BEGIN
    SELECT score INTO sco FROM ScoreTable WHERE stuNo = sno AND crsNo = cno;
EXCEPTION 
    WHEN NO_DATA_FOUND THEN 
        DBMS_OUTPUT.put_line('没找到数据！');
    WHEN OTHERS THEN
        DBMS_OUTPUT.put_line('产生异常！'); 
END;    
/
​
SET SERVEROUTPUT ON;
DECLARE
  score ScoreTable.score%TYPE;
BEGIN 
  sel_score('189074000', '0008', score);/*不需要EXECUTE*/
  DBMS_OUTPUT.PUT_LINE(score);
END;
/
```
 

#### 创建存储过程（Mysql)


```sql
DELIMITER $$
​
CREATE PROCEDURE calcsum(OUT sum INT)
BEGIN 
   DECLARE n INT;
   SET n = 0;
   SET sum = 0;
   WHILE n <= 100 do
      SET sum = sum + n;
      SET n = n + 1;
   END WHILE;
END;
$$
```
#### 执行存储过程（Mysql)


```sql
DELIMITER ;
SET @sum = 0;
CALL calcsum(@sum);
 

FUNCTION


/
CREATE OR REPLACE FUNCTION f_calcsum(n number) return int
as
    i number:=0;
    result number:=0;
BEGIN
    while i<=n loop
        result:=result+n;
        i:=i+1;
    end loop;
    return result;
END;
/
DECLARE
  result int:=f_calcsum(10);
BEGIN
  dbms_output.put_line('结果是'||result);
END;
/
```
### 游标

#### 利用游标变量循环


```sql
SET SERVEROUTPUT ON;
DECLARE
  stu Temp%ROWTYPE;
  CURSOR cur_tmp IS SELECT * FROM Temp;
BEGIN
  OPEN cur_tmp;
  LOOP                  
    FETCH cur_tmp INTO stu;
    EXIT WHEN cur_tmp%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(stu.sno||stu.sname);
  END LOOP;
CLOSE cur_tmp;
END;   
```
#### FOR循环


```sql
/
SET SERVEROUTPUT ON;
DECLARE
  CURSOR cur_tmp IS SELECT * FROM Temp;
BEGIN
-- 无需手动打开关闭游标
  FOR stu IN cur_tmp LOOP
  DBMS_OUTPUT.PUT_LINE(stu.sno||stu.sname);
  END LOOP;
END;
/
​
/
SET SERVEROUTPUT ON;
BEGIN
  FOR stu IN (SELECT * FROM Temp) LOOP
  DBMS_OUTPUT.PUT_LINE(stu.sno||stu.sname);
  END LOOP; 
END;
/
SET SERVEROUTPUT ON;
BEGIN 
    FOR stu IN (SELECT * FROM Temp) LOOP
        IF stu.sname  LIKE '%J%' THEN
            DBMS_OUTPUT.PUT_LINE(stu.sname);
        END IF;
    END LOOP;
END;    
/
SET SERVEROUTPUT ON;
CREATE OR REPLACE PROCEDURE find_name(name Temp.sname%TYPE)
AS
BEGIN
    FOR stu IN (SELECT * FROM Temp) LOOP
        IF stu.sname LIKE '%'||name||'%' THEN 
            DBMS_OUTPUT.PUT_LINE(stu.sname);
        END IF;
    END LOOP;
END;    
/
EXECUTE find_name('J');
/
```
#### 隐式游标


```sql
SET SERVEROUTPUT ON;
BEGIN
  UPDATE Temp SET credit = credit + 1 WHERE credit IS NOT NULL;
  IF SQL%NOTFOUND THEN   
    DBMS_OUTPUT.PUT_LINE('没有记录被更改！');
  ELSE
    DBMS_OUTPUT.PUT_LINE('更改了'||SQL%ROWCOUNT||'条记录。');
  END IF;
END;
/
```
### 异常处理


```sql
/
DECLARE
    stuno Temp.sno%TYPE;
    stuname Temp.sname%TYPE;
BEGIN
    SELECT sno,sname INTO stuno,stuname FROM  Temp ;
    DBMS_OUTPUT.PUT_LINE(stuno||stuname);
EXCEPTION
    WHEN no_data_found THEN
      DBMS_OUTPUT.PUT_LINE('数据没找到');
    WHEN too_many_rows THEN
      DBMS_OUTPUT.PUT_LINE('结果集超过一行');
END;
/
DECLARE
    null_exp EXCEPTION;
    stucj Subject%ROWTYPE;
BEGIN
  stucj.cname:='001241'; stucj.cno:='206';
  INSERT INTO Subject VALUES(stucj.cno,stucj.cname,stucj.ctime);
  IF stucj.ctime IS NULL THEN   
   RAISE null_exp;   
  END IF;
EXCEPTION
    WHEN null_exp THEN
      DBMS_OUTPUT.PUT_LINE('成绩不能为空值');
      ROLLBACK;
    WHEN others THEN
      DBMS_OUTPUT.PUT_LINE('其他异常');
END;
/
```
### 触发器


```sql
CREATE TABLE ScoreT_log
(
    operate_type varchar2(20),   
    operate_date date,
    operate_user varchar2(20)
 );
/
CREATE OR REPLACE TRIGGER tri_ScoreT BEFORE INSERT OR DELETE OR UPDATE ON ScoreTable
DECLARE 
    operate_type varchar2(20);
BEGIN
    IF INSERTING THEN 
        operate_type := 'INSERT';
    ELSIF DELETING THEN 
        operate_type := 'DELETE';
    ELSE
        operate_type := 'UPDATE';
    END IF;
    INSERT INTO ScoreT_log VALUES(operate_type, sysdate, ora_login_user );
END;    
/
UPDATE ScoreTable SET score = 75 WHERE sno = '189074000' AND cno = '0001';
-- SELECT * FROM ScoreT_log;
/
CREATE OR REPLACE TRIGGER tri_del_tmp AFTER DELETE ON Temp
FOR EACH ROW 
BEGIN
    DELETE FROM ScoreTable WHERE sno = :old.sno;
END;
/
​
DELETE FROM Temp WHERE sno = '189074011';
-- SELECT * FROM ScoreTable;
-- SELECT * FROM ScoreT_log;
```
### 事务提交与回退


```sql
SHOW AUTOCOMMIT;
SET AUTOCOMMIT ON;
SET AUTOCOMMIT OFF;
-- 某些SQL语句（DDL语句, DCL语句, ），在它们被执行时会生成隐含的COMMIT命令，将会马上导致事务提交。
​
-- 只撤销一部分事务  
UPDATE Temp SET remarks = '三坏学生' WHERE sno = '189074005'; 
SAVEPOINT t;
UPDATE Temp SET remarks = '三坏学生', dept = '软件工程' WHERE sno = '189074006'; 
ROLLBACK TO t;
```
### JDBC

#### JDBC execute

```java
package priv.Matrix.mine;
​
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
​
public class Execute 
{
    public static void main(String[] args)
    {
​
        String conStr = "jdbc:oracle:thin:@localhost:1521:xe";
        String username = "user1";
        String password = "user1";
​
        Connection con = null;
        Statement stmt = null;
        ResultSet rs = null;
​
        String sql = "CREATE TABLE tmp"
            + "("
            + "     id varchar2(20),"
            + "     info varchar2(30)"
            + ")";
​
​
        try
        {
            Class.forName("oracle.jdbc.driver.OracleDriver");
        }
        catch (ClassNotFoundException e)
        {
            --  TODO Auto-generated catch block
            e.printStackTrace();
        }
​
        try
        {
            con = DriverManager.getConnection(conStr, username, password);
            stmt = con.createStatement();
            boolean isResultSet = stmt.execute(sql);
            int count = 0;
            int rowAffected = 0;
            while(true)
            {
                if(isResultSet)
                {
                    count++;
                    System.out.println("ResultSet:" + count);
                    rs = stmt.getResultSet();
                    while(rs.next())
                    {
                        System.out.println(rs.getString("id") + "\t" + rs.getString("info"));
                    }
​
                }
                else
                {
                    rowAffected = stmt.getUpdateCount();
                    System.out.println("row affected:" + rowAffected);
                    if(rowAffected == -1)
                    {
                        break;
                    }
​
                }
                stmt.getMoreResults();
            }
​
        }
        catch (SQLException e)
        {
            --  TODO Auto-generated catch block
            e.printStackTrace();
        }
​
    }
​
​
}
```

#### JDBC executeBatch

```java
package priv.Matrix.mine;
​
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
​
public class Execute 
{
    public static void main(String[] args)
    {
​
        String conStr = "jdbc:oracle:thin:@localhost:1521:xe";
        String username = "user1";
        String password = "user1";
​
        Connection con = null;
        Statement stmt = null;
​
        String createTab = ("CREATE TABLE tmp"
                + "("
                + "     id varchar2(20),"
                + "     info varchar2(30)"
                + ")");
​
        int[] rowAffected = new int[3];
​
        try
        {
            Class.forName("oracle.jdbc.driver.OracleDriver");
        }
        catch (ClassNotFoundException e)
        {
            --  TODO Auto-generated catch block
            e.printStackTrace();
        }
​
        try
        {
            con = DriverManager.getConnection(conStr, username, password);
            stmt = con.createStatement();
​
            stmt.executeUpdate(createTab);
​
            stmt.addBatch("INSERT INTO tmp VALUES('10001', 'first_info')");
            stmt.addBatch("UPDATE tmp SET info = 'second_info' WHERE id = '10001'");
            stmt.addBatch("DELETE FROM tmp WHERE id = '10001'");
​
            rowAffected = stmt.executeBatch();
​
            for(int i = 0; i < 3; i++)
            {
                System.out.println("row affected:" + rowAffected[i]);
            }
​
        }
        catch (SQLException e)
        {
            --  TODO Auto-generated catch block
            e.printStackTrace();
        }
​
    }
​
​
}
```

#### JDBC PreparedStatement

```java
package priv.Matrix.mine;
​
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
​
public class Execute 
{
    public static void main(String[] args)
    {
​
        String conStr = "jdbc:oracle:thin:@localhost:1521:xe";
        String username = "user1";
        String password = "user1";
​
        Connection con = null;
        Statement stmt = null;
        ResultSet rs = null;
​
        String createTab = ("CREATE TABLE tmp"
                + "("
                + "     id varchar2(20),"
                + "     info varchar2(30)"
                + ")");
​
​
        try
        {
            Class.forName("oracle.jdbc.driver.OracleDriver");
        }
        catch (ClassNotFoundException e)
        {
            --  TODO Auto-generated catch block
            e.printStackTrace();
        }
​
        try
        {
            con = DriverManager.getConnection(conStr, username, password);
​
            stmt = con.createStatement();
            stmt.executeUpdate(createTab);
​
            PreparedStatement pstmtInsert = con.prepareStatement("INSERT INTO tmp VALUES(?, ?)");
            PreparedStatement pstmtSelect = con.prepareStatement("SELECT * FROM tmp WHERE id = ?");
            PreparedStatement pstmtUpdate = con.prepareStatement("UPDATE tmp SET info = ? WHERE id = ?");
            PreparedStatement pstmtDelete = con.prepareStatement("DELETE FROM tmp WHERE id = ?");
​
           -- 该字符串不需要加单引号(')，即"INSERT INTO tmp VALUES('?', '?')"是错的  
            pstmtInsert.setString(1, "10001");
            pstmtInsert.setString(2, "first info");
            pstmtInsert.executeUpdate();
​
            pstmtSelect.setString(1, "10001");
            rs = pstmtSelect.executeQuery();
            while(rs.next())
            {
                System.out.println(rs.getString("id") + "\t" + rs.getString("info"));
            }
​
​
            pstmtUpdate.setString(1, "second info");
            pstmtUpdate.setString(2, "10001");
            pstmtUpdate.executeUpdate();
​
            pstmtDelete.setString(1, "10001");
            pstmtDelete.executeUpdate();
​
​
​
​
​
        }
        catch (SQLException e)
        {
            --  TODO Auto-generated catch block
            e.printStackTrace();
        }
​
    }
​
​
}
```
