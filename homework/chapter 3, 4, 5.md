## 第三章
### P4
(1) 4个表的建立：
S：
```sql
CREATE TABLE S (SNO CHAR(2) UNIQUE, SNAME CHAR(6), STATUS CHAR(2), CITY CHAR(4));
```
P：
```sql
CREATE TABLE P (PNO CHAR(2) UNIQUE, PNAME CHAR(6), COLOR CHAR(2), WEIGHT INT);
```
J：
```SQL
CREATE TABLE J (JNO CHAR(2) UNIQUE, JNAME CHAR(8), CITY CHAR(4));
```
SPJ：
```SQL
CREATE TABLE SPJ (SNO CHAR(2) UNIQUE, PNO CHAR(2), JNO CHAR(2), QTY INT);
```
(2) 查询：
（1）
```sql
SELECT DISTINCT SNO FROM SPJ WHERE JNO='J1';
```
（2）
```sql
SELECT DISTINCT SNO FROM SPJ WHERE JNO='J1' AND PNO='P1';
```
（3）
```sql
SELECT DISTINCT SNO FROM SPJ, P WHERE JNO='J1' AND SPJ.PNO=P.PNO AND COLOR='红';
```
（4）
```sql
SELECT DISTINCT JNO FROM SPJ WHERE JNO NOT IN (SELECT JNO FROM SPJ,P,S WHERE S.CITY='天津' AND COLOR='红' AND S.SNO=SPJ.SNO AND P.PNO=SPJ.PNO);
```
（5）
```sql
SELECT DISTINCT PNO FROM SPJ WHERE SNO='S1';
/* 得到(P1,P2) */
SELECT JNO FROM SPJ WHERE PNO='P1' AND JNO IN (SELECT JNO FROM SPJ WHERE PNO='P2');
```

### P5
（1）
```sql
SELECT SNAME, CITY FROM S;
```
（2）
```sql
SELECT PNAME, COLOR, WEIGHT FROM P;
```
（3）
```sql
SELECT DISTINCT JNO FROM SPJ WHERE SNO='S1';
```
（4）
```sql
SELECT DISTINCT PNAME, QTY FROM SPJ,P WHERE JNO='J2' AND P.PNO=SPJ.PNO;
```
（5）
```sql
SELECT PNO FROM S,SPJ WHERE S.SNO=SPJ.SNO AND CITY='上海';
``````
（6）
```sql
SELECT JNAME FROM S,J,SPJ WHERE S.SNO=SPJ.SNO AND S.CITY='上海' AND J.JNO=SPJ.JNO;
```
（7）
```sql
SELECT JNO FROM S,J,SPJ WHERE JNO NOT IN (SELECT JNO FROM S,J,SPJ WHERE S.SNO=SPJ.SNO AND S.CITY='天津');
```
（8）
```sql
UPDATE P SET COLOR='蓝' WHERE COLOR='红';
```
（9）
```sql
UPDATE SPJ SET SNO='S3' WHERE SNO='S5' AND JNO='J4' AND PNO='P6';
``````
（10）
```sql
DELETE FROM S WHERE SNO='S2';
DELETE FROM SPJ WHERE SNO='S2';
```
（11）
```sql
INSERT INTO SPJ VALUES ('S2', 'J6', 'P4', 200);
```



## 第四章
### P6
(1)
```mysql
GRANT ALL PRIVILEGES ON Student, Class TO U1 WITH GRANT OPTION;
```
(2)
```mysql
GRANT SELECT,UPDATE(家庭住址) ON Student TO U2;
```
(3)
```mysql
GRANT SELECT ON Class TO PUBLIC;
```
(4)
```mysql
GRANT SELECT,UPDATE ON Student TO R1;
```
(5)
```mysql
GRANT R1 TO U1 WITH ADMIN OPRION;
```
### P7
(1)
```mysql
GRANT SELECT ON Staff, Dept TO wm;
```
(2)
```mysql
GRANT INSERT,DELETE ON Staff, Dept TO ly;
```
(3)
```mysql
GRANT SELECT ON Staff WHEN USER()=staffNo TO ALL;
```
(4)
```mysql
GRANT SELECT,UPDATE(工资) ON Staff TO lx;
```
(5)
```mysql
GRANT ALTER TABLE ON Staff, Dept TO zx;
```
(6)
```mysql
GRANT ALL PRIVILEGES ON Staff, Dept TO zp WITH GRANT OPTION;
```
(7)
```mysql
CREATE VIEW DEPT_SALARY AS
	SELECT Dept.deptName,MAX(工资),MIN(工资),AVG(工资)
	FROM Staff,Dept
	WHERE Staff.deptNo=Dept.deptNo
	GROUP BY Staff.deptNo;
GRANT SELECT ON DEPT_SALARY TO yl;
```
### P8
(1)
```mysql
REVOKE SELECT ON Staff, Dept FROM wm;
```
(2)
```mysql
REVOKE INSERT,DELETE ON Staff, Dept FROM ly;
```
(3)
```mysql
REVOKE SELECT ON Staff WHEN USER()=staffNo FROM ALL;
```
(4)
```mysql
REVOKE SELECT,UPDATE(工资) ON Staff FROM lx;
```
(5)
```mysql
REVOKE ALTER TABLE ON Staff, Dept FROM zx;
```
(6)
```mysql
REVOKE ALL PRIVILEGES ON Staff, Dept FROM zp;
```
(7)
```mysql
REVOKE SELECT ON DEPT_SALARY FROM yl;
DROP VIEW DEPT_SALARY;
```


## 第五章
### P6
(1)
```mysql
CREATE TABLE Dept(
	deptNo INT,
	deptName VARCHAR(10),
	manager CHAR(10),
	phone CHAR(12)
	PRIMARY KEY(deptNo)
); 
CREATE TABLE Staff(
	staffNo INT,
	staffName VARCHAR(10),
	age INT,
	CONSTRAINT CK_AGE CHECK(age <= 60)
	job VARCHAR(10),
	salary INT,
	deptNo INT,
	PRIMARY KEY(staffNo)
	CONSTRAINT FK_DEPTNO FOREIGN KEY(deptNo) REFFERENCES Dept(deptNo)
);
```
`

### P7 
关系系统中，当操作违反实体完整性、参照完整性和用户定义的完整性约束条件时，一般是如何分别进行处理的？
答：
	违反实体完整性和用户定义的完整性的操作一般都采用拒绝执行的方式进行处理。
	一般地，当对参照表和被参照表的操作违反了参照完整性时，系统选用默认策略，即拒绝执行。如果想让系统采用其他策略则必须在创建参照表时显示地加以说明。

