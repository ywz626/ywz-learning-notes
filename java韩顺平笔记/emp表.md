# emp表

| EMPNO | ENAME | JOB  | MGR  | HIREDATE | SAL  | COMM | DEPTNO |
| ----- | ----- | ---- | ---- | -------- | ---- | ---- | ------ |
|       |       |      |      |          |      |      |        |
| 7369 | SMITH | CLERK | 7902 | 1980-12-17 | 800.00 | NULL | 20   |
| ---- | ----- | ----- | ---- | ---------- | ------ | ---- | ---- |
|      |       |       |      |            |        |      |      |
dept

| DEPTNO | DNAME | LOC  |
| ------ | ----- | ---- |
|        |       |      |
salgrade

| GRADE | LOSAL | HISAL |
| ----- | ----- | ----- |
|       |       |       |
19.列出所有"CLERK"(办事员)的姓名及其部门名称,部门的人数

```sql
select ename,dname,count(*)
from emp
join dept
on emp.deptno=dept.deptno
where emp.job='CLERK'
group by ename,dname;
```

