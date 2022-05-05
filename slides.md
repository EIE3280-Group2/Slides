---
theme: seriph
highlighter: shiki
lineNumbers: false
info: |
  ## An auction model for the course enrolling process in CUHK-Shenzhen
drawings:
  persist: false
layout: cover
background: loading.jpg
---

<h1 class="pri-title"> Course Enrollment </h1>

<h1 class="sec-title"> ── A Smarter Way </h1>

<p class="thi-title">An auction model for the course enrolling process in CUHK-Shenzhen</p>

<div class="abs-bl mx-14 my-12 flex">
  <div class="ml-3 flex flex-col text-left">
    <div><b>GROUP 2</b> Song Chen, Zefeng Song, Siwei Zhang</div>
    <div class="text-sm opacity-50">May 7th, 2022</div>
  </div>
</div>

<style>
.pri-title {
  margin-right: 240px;
}
.sec-title {
  margin-left: 240px;
}
.thi-title {
  opacity: 0.7;
  font-size: 1.2rem;
}
</style>

---

# Problems

The current course registration model in CUHK(SZ) exists many problems.

- <OouiNetworkOff class="ib" /> **Laggy course enrolling system** - Single threaded server written in COBOL. 
- <MdiScaleUnbalanced class="ib" /> **Unfair enrolling method** - Result mainly depends on response time and network delay.
- <FadRandom1dice class="ib" /> **Random behaviour** - Unstable operation of the system outside standard usage.

<br>

<v-click>

# Our model's features

- <IcBaselineAutoAwesome class="ib" /> **No need to register courses concurrently** - Result is based on an auction. Say goodbye to get up early.
- <IcBaselineAutoAwesome class="ib" /> **No stress** - Even a raspberry pi <SimpleIconsRaspberrypi class="ib"/> can handle the registration process (in theory).
- <IcBaselineAutoAwesome class="ib" /> **Express student's willingness** - Quantify your demand using willingness points.
- <IcBaselineAutoAwesome class="ib" /> And even more...

</v-click>

<style>
.ib {
  display: inline-block;
}
</style>

---

# Simple model

We will show you a simplified model first.

We will assume that for all courses, the students' willingness points weighs the same. 

And the more willingness points posts to the course, the higher rank you will get.

In the simplified model, we will give 100 willingness points to each students.

<div class="flex justify-center">

<div>

minimize $p$

subject to $RANK(p) \leq \text{Course Quota}$

variables p

</div>

</div>

**Condition** - Assume that **25 students** will register **5 courses** with each **19 quota** (Not enough).

**Restriction** - The **maximum number of course** allowed to register is **4** due to the restriction of credit unit.

**Target** - And each students should try to **maximize the course number** that they need to successfully register.

---

# Assume you are a student among them...

What will you choose?

> Assume that 25 students will register 5 courses with each 19 quota.

| Course    | A    | B    | C    | D    | E    |
| ------    | ---- | ---- | ---- | ---- | ---- |
| You       | 10   | 10   | 10   | 10   | 10   |
| 5 student | 0    | 25   | 25   | 25   | 25   |
| 5 student | 25   | 0    | 25   | 25   | 25   |
| 5 student | 25   | 25   | 0    | 25   | 25   |
| 5 student | 25   | 25   | 25   | 0    | 25   |
| 4 student | 25   | 25   | 25   | 25   | 0    |

<v-click>

Hence, we should distribute the willingness points to 4 courses rather than 5 or more courses, otherwise, we will lose all the courses.

</v-click>

---

# Assume you are a student among them...

> Assume that 25 students will register 5 courses with each 19 quota.

| Course    | A    | B    | C    | D    | E    |
| --------- | ---- | ---- | ---- | ---- | ---- |
| You       | 26   | 26   | 24   | 24   | 0    |
| 5 student | 0    | 25   | 25   | 25   | 25   |
| 5 student | 25   | 0    | 25   | 25   | 25   |
| 5 student | 25   | 25   | 0    | 25   | 25   |
| 5 student | 25   | 25   | 25   | 0    | 25   |
| 4 student | 25   | 25   | 25   | 25   | 0    |

<v-click>

We will definately lose C and D if we choose to increase the points in A and B.

Hence, the optimized solution is to distribute the points evenly to 4 courses.

</v-click>

---

# Weighted model

As we have different types of students, and different types of courses, simplified model may not perfectly meet our needs.

And for simplicity, we only consider major required, major elective, free elective three types of courses. University core is much more complicated, which will be clarified in the report.

|        | Major Required | Major Elective | Free Elective |
| ------ | -------------- | -------------- | ------------- |
| Weight | 1.9            | 1.5            | 1             |

<p class="magrin-bottom-0">A course can either be major required or major elective, or even free elective to different students.</p>

<p class="magrin-top-0">By common sense, the quota will larger than the sum of major required and major elective students number.</p>

|                                     | Major Required | Major Elective | Free Elective |
| ----------------------------------- | -------------- | -------------- | ------------- |
| Points needed to ensure (in theory) | 14             | 17             | 25            |

<p class="magrin-top-0">And the spare points can be posts to other courses that students willing to take, which means it gives chances to student to post more than 4 courses.</p>

<style>
.magrin-top-0 {
  margin-top: 0;
}
.magrin-bottom-0 {
  margin-bottom: 0;
}
</style>

---

# How the system works

>   Priority:
>   1. For different type of courses: Major Required > Major Elective > Free Elective
>   2. For the same type of courses: According to the willingness points posted.
>   3. All the same: Choose randomly.

Steps

1. Students posts their willingness points to the courses.
2. Generate a ranked queue for each course
3. If some students will register more than 4 courses, kick it out of the course queue which course has a less priority to the student.
4. If the student with the same rank exceeds the quota, choose between them according to the priority.

---

# How the system works

Reference implementation

```sql
INSERT INTO ENROLLMENT (STU_ID, COURSE_ID, POINTS) VALUE (?, ?, ?);
```

```sql
CREATE VIEW RANKED_ENROLLMENT AS
SELECT ENROLLMENT.STU_ID, ENROLLMENT.COURSE_ID, COURSE_TYPE.CTYPE_ID, ENROLLMENT.POINTS, 
(ENROLLMENT.POINTS * COURSE_TYPE.WEIGHT) AS WEIGHTED_POINT 
FROM ENROLLMENT 
INNER JOIN STU_INFO ON ENROLLMENT.STU_ID = STU_INFO.STU_ID
INNER JOIN COURSE_TYPE ON STU_INFO.MAJOR_ID = COURSE_TYPE.MAJOR_ID AND ENROLLMENT.COURSE_ID = COURSE_TYPE.COURSE_ID
WHERE ENROLLMENT.COURSE_ID = ?
ORDER BY WEIGHTED_POINT DESC, COURSE_TYPE.CTYPE_ID DESC, ENROLLMENT.POINTS DESC;
```

```sql
SELECT STU_ID, COURSE_ID, CTYPE_ID, POINTS, WEIGHTED_POINT 
FROM RANKED_ENROLLMENT WHERE COURSE_ID = ? LIMIT ?;
```

```sql
DELETE FROM RANKED_ENROLLMENT WHERE STU_ID = ? AND COURSE_ID = ?;
```

```sql
SELECT STU_ID, COURSE_ID, CTYPE_ID, POINTS, WEIGHTED_POINT 
FROM RANKED_ENROLLMENT WHERE COURSE_ID = ? 
ORDER BY WEIGHTED_POINT DESC, CTYPE_ID DESC, POINTS DESC LIMIT ?;
```

---

## Reference

Guofu Tan, "Auction Theory,” in *Research Frontier in Economics and Finance*, edited by Guoqiang Tian, 267-330, Commercial Affair Press, 2002.

Undergraduate Education, P. U. (2021, June). *Regulations and Measures for the Management of Undergraduate Course Selection at Peking University - RULES*. Retrieved April 20, 2022, from http://www.dean.pku.edu.cn/web/rules\_info.php?id=17

## Contribution

-  Siwei Zhang: 34%
-  Song Chen: 33%
-  Zefeng Song: 33%
