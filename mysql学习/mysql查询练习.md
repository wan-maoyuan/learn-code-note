## MySQL查询练习
### 准备
- 四张表结构

学生表(student)   |课程表(course)  |成绩表(score)  |教师表(teacher)
---               |---             |---            |---
学号(sno)         |课程号(cno)     |学号(sno)      |教师编号(tno) 
姓名(sname)       |课程名称(cname) |课程号(cno)    |教师名字(tname) 
出生年月日        |教师编号(tno)   |成绩(degree)   |教师性别(tsex) 
所在班级(cname)   |                |               |出生年月日 
性别(ssex)        |                |               |职称
  |               |                |               |所在部门(depart) 

1. 查询student表中所有的记录
```sql
select * from student;
```

2. 查询student表中的所有记录sname、ssex和class列
```sql
select sname, ssex, class from student;
```

3. 查询教师所有的单位即不重复的depart列
```sql
select distinct depart from teacher;
```

4. 查询score表中成绩在60到80之间的所有记录
```sql
 select * from score where degree between 60 and 80;
 ```

5. 查询score表中成绩为85，86或88的记录
```sql
select * from score where degree in (85,86,88);
```

6. 查询student表中“95031”班或性别为女的同学记录
```sql
select * from student where class='95031' or ssex='女';
```

7. 以class降序查询student表中所有的记录。
```sql
select * from student order by class desc;
```

8. 以cno升序、degree降序查询score表的所有记录。
```sql
select * from score order by cno asc, degree desc ;
```

9. 查询“95031”班的学生人数。
```sql
select count(*) from student where class='95031';
```

10. 查询score表中的最高分的学生学号和课程号。（子查询或者排序）
```sql
select sno, cno from score where degree=(select max(degree) from score);
```

11. 查询每门课的平均成绩
```sql
select avg(degree) from score group by cno;
```

12. 查询score表中至少有两名学生选修的并且以3开头的课程的平均数
```sql
select cno, avg(degree) from score group by cno having count(*) > 1 and cno like '3%';
```

13. 查询分数大于70，小于90的sno列
```sql
select sno from score where degree between 70 and 90;
```

14. 查询所有学生的sname、cno和degree列。
```sql
select student.sname, score.cno, score.degree 
from student, score 
where student.sno = score.sno;
```

15. 查询所有学生的sno、cname和degree列。
```sql
select score.sno, course.cname, score.degree 
from course, score 
where score.cno = course.cno;
```

16. 查询所有学生的sname、cname和degree列
```sql
select student.sname, course.cname, score.degree
from student, course, score
where student.sno = score.sno and score.cno = course.cno;
```

17. 查询“95031”班学生每门课的平均分
```sql
select cno, avg(degree) from score 
where sno in (select sno from student class='95031')
group by cno;
```

18. 查询选修‘3-105’课程的成绩高于‘109’号同学“3-105”成绩的所有同学记录
```sql
select * from score 
where cno = '3-105'
and degree > (select degree from score where sno='109' and cno='3-105');
```

19. 查询成绩高于学号为‘109’、课程号为“3-105”的成绩的所有记录
```sql
select * from score 
where degree > (select degree from score where sno='109' and cno='3-105');
```

20. 查询和学号108、101的同学同年出生的所有学生的sno、sname和sbirthday列
```sql
select sno, sname, sbirthday from student
where year(sbirthday) in 
(select year(sbirthday) from student where sno in (108, 101));
```

21. 查询张旭老师任课的学生成绩
```sql
select * from score 
where cno in (select cno from course 
where tno=(select tno from teacher where tname='张旭'));
```

22. 查询选修选课程的同学人数多于5人教师姓名。
```sql
select tname from teacher 
where tno in 
(select tno from course 
where cno in
(select cno from score group by cno having count(*) > 5));
```

23. 查询95033班和95031班全体学生的记录
```sql
select * from student where cname in (95033, 95031);
```

24. 查询存在有85分以上成绩的课程cno
```sql
select cno from score where degree>'85';
```

25. 查询计算机系教师所教课程的成绩表
```sql
select * from score 
where cno in 
(select cno from course where tno in 
    (select tno in teacher where depart='计算机系'));
```

26. 查询计算机系和电子工程系不同职称的教师的tname和prof。
```sql
select * from teacher 
where depart='计算机系' and prof 
not in (select prof from teacher where depart='电子工程系')
union
select * from teacher 
where depart='电子工程系' and prof 
not in (select prof from teacher where depart='计算机系'); 
```

27. 查询选修编号为‘3-105’课程且成绩至少高于选修编号为‘3-245’的同学的cno、sno和degree，并且按照degree从高到低进行排序。
```sql
select * from score 
where cno='3-105' 
and degree > any
(select degree from score where cno='3-245'); 
```

28. 查询选修编号为‘3-105’且成绩高于选修编号为‘3-245’课程的同学的cno、sno和degree。
```sql
select * from score 
where cno='3-105' 
and degree > all
(select degree from score where cno='3-245'); 
```

29. 查询所有教师和同学的name、sex和birthday
```sql
select teacher.tname as name, teacher.tsex as sex, teacher.tbirthday as birthday 
from teacher
union
select student.sname as name, student.ssex as sex, student.sbirthday as birthday 
from student;
```

30. 查询所有女教师和女同学的name、sex和birthday。
```sql
select teacher.tname as name, teacher.tsex as sex, teacher.tbirthday as birthday 
from teacher where tsex = '女'
union
select student.sname as name, student.ssex as sex, student.sbirthday as birthday 
from student where ssex = '女';
```

31. 查询成绩比该课程平均成绩低的同学。
```sql
select * from score as a
where degree < (select avg(degree) 
from score as b 
where a.cno=b.cno);
```

32. 查询至少有两名男生的班号。
```sql
select class from student 
where ssex='男' 
group by class having count(*) > 1;
```

33. 查询student表中不姓王的同学记录
```sql
select * from student where sname not like '王%';
```

34. 查询男教师及其上的课程
```sql
select cname from course 
where tno in 
(select tno from teacher where tsex='男');
```