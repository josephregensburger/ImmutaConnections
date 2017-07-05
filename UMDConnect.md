# UMD Canvas Immuta Integration
Immuta  
July 6, 2017  




## Data Source Creation

This section will discuss a data model for the University of Maryland (UMD) Canvas 
dataset within the Immuta environment.  The table below shows the Canvas tables which 
have been loaded into the UMD Postgres.


```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```



Table: Canvas Tables in UMD Postgres

TABLE_CAT     TABLE_SCHEM   TABLE_NAME                                       TABLE_TYPE   REMARKS 
------------  ------------  -----------------------------------------------  -----------  --------
canvas_data   public        account_dim                                      TABLE                
canvas_data   public        assignment_dim                                   TABLE                
canvas_data   public        assignment_rule_dim                              TABLE                
canvas_data   public        communication_channel_dim                        TABLE                
canvas_data   public        communication_channel_fact                       TABLE                
canvas_data   public        conversation_dim                                 TABLE                
canvas_data   public        conversation_message_dim                         TABLE                
canvas_data   public        course_dim                                       TABLE                
canvas_data   public        course_section_dim                               TABLE                
canvas_data   public        discussion_entry_dim                             TABLE                
canvas_data   public        discussion_entry_fact                            TABLE                
canvas_data   public        discussion_topic_dim                             TABLE                
canvas_data   public        discussion_topic_fact                            TABLE                
canvas_data   public        enrollment_dim                                   TABLE                
canvas_data   public        enrollment_fact                                  TABLE                
canvas_data   public        enrollment_term_dim                              TABLE                
canvas_data   public        external_tool_activation_dim                     TABLE                
canvas_data   public        external_tool_activation_fact                    TABLE                
canvas_data   public        file_dim                                         TABLE                
canvas_data   public        module_progression_completion_requirement_fact   TABLE                
canvas_data   public        quiz_dim                                         TABLE                
canvas_data   public        quiz_fact                                        TABLE                
canvas_data   public        role_dim                                         TABLE                
canvas_data   public        user_dim                                         TABLE                
canvas_data   public        wiki_dim                                         TABLE                
canvas_data   public        wiki_fact                                        TABLE                
canvas_data   public        wiki_page_dim                                    TABLE                
canvas_data   public        wiki_page_fact                                   TABLE                

### Course Datasources
We can create Immuta datasources by merging similiar sources into 
more usable views.  For example, we can create a cohesive Course table, 
combining details between the parent organization for the course, the 
academic term in which the course is offered, and the course information.  
This can be done using the following SQL statement:

```sql
/* Course Detail Table */
SELECT c.id, c.name as course_name, 
  c.code, c.workflow_state as course_workflow_state, 
  c.wiki_id, c.enrollment_term_id, t.name as term_name, 
  a.root_account, a.name as account_name, 
  a.parent_account as parent_account,
  a.subaccount1 as subaccount1, 
  a.subaccount2 as subaccount2
FROM course_dim as c
LEFT JOIN account_dim as a
ON c.account_id = a.id
LEFT JOIN enrollment_term_dim as t
ON c.enrollment_term_id=t.id
```


|course_name                                                       |code    |course_workflow_state |term_name   |root_account           |parent_account |subaccount1 |subaccount2    |
|:-----------------------------------------------------------------|:-------|:---------------------|:-----------|:----------------------|:--------------|:-----------|:--------------|
|FMSC899-1501: Doctoral Dissertation Research-Spring 2014 eanders  |FMSC899 |deleted               |Spring 2014 |University of Maryland |Family Science |SPHL        |Family Science |
|FMSC789-1101: Non-Thesis Research-Spring 2014 rzeiger             |FMSC789 |deleted               |Spring 2014 |University of Maryland |Family Science |SPHL        |Family Science |
|FMSC898-1801: Pre-Candidacy Research-Spring 2014 kroy             |FMSC898 |deleted               |Spring 2014 |University of Maryland |Family Science |SPHL        |Family Science |
|FMSC699-0301: Independent Study-Spring 2014 lleslie               |FMSC699 |deleted               |Spring 2014 |University of Maryland |Family Science |SPHL        |Family Science |
|FMSC899-1901: Doctoral Dissertation Research-Spring 2014 shenassa |FMSC899 |available             |Spring 2014 |University of Maryland |Family Science |SPHL        |Family Science |
|FMSC699-1601: Independent Study-Spring 2014 bbraun                |FMSC699 |deleted               |Spring 2014 |University of Maryland |Family Science |SPHL        |Family Science |

A detailed course section table can be built from the above query using the following 
SQL statement:



```sql
/* Section Detail */
SELECT cs.id as course_section_id, ct.id as course_id, 
  cs.name as course_section_name, 
  ct.course_name, ct.code, ct.course_workflow_state, 
  ct.wiki_id, ct.enrollment_term_id, ct.term_name, 
  ct.root_account, ct.account_name, 
  ct.parent_account,
  ct.subaccount1, 
  ct.subaccount2  
FROM course_section_dim as cs
INNER JOIN 
  (SELECT c.id, c.name as course_name, 
    c.code, c.workflow_state as course_workflow_state, 
    c.wiki_id, c.enrollment_term_id, t.name as term_name, 
    a.root_account, a.name as account_name, 
    a.parent_account as parent_account,
    a.subaccount1 as subaccount1, 
    a.subaccount2 as subaccount2
  FROM course_dim as c
  LEFT JOIN account_dim as a
  ON c.account_id = a.id
  LEFT JOIN enrollment_term_dim as t
  ON c.enrollment_term_id=t.id
  ) as ct
ON cs.course_id = ct.id
```


|course_section_name                                                                                                                      |course_name                                                                                                                              |code          |course_workflow_state |term_name    |root_account           |parent_account                                 |subaccount1 |subaccount2                                    |
|:----------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------|:-------------|:---------------------|:------------|:----------------------|:----------------------------------------------|:-----------|:----------------------------------------------|
|HIST408G-0101: Senior Seminar; The Environmental History of Eurasia-Fall 2017 scameron                                                   |HIST408G-0101: Senior Seminar; The Environmental History of Eurasia-Fall 2017 scameron                                                   |HIST408G      |claimed               |Fall 2017    |University of Maryland |History                                        |ARHU        |History                                        |
|GVPT729A-0101: Special Topics in Quantitative Political Analysis; Advanced Maximum Likelihood Estimation-Fall 2017 mhanmer               |GVPT729A-0101: Special Topics in Quantitative Political Analysis; Advanced Maximum Likelihood Estimation-Fall 2017 mhanmer               |GVPT729A      |claimed               |Fall 2017    |University of Maryland |Government & Politics                          |BSOS        |Government & Politics                          |
|PSYC318D-0101/WMST498A-0101: Community Interventions: Theory and Research; Community Interventions: Domestic Violence-Fall 2017 kmobrien |PSYC318D-0101/WMST498A-0101: Community Interventions: Theory and Research; Community Interventions: Domestic Violence-Fall 2017 kmobrien |PSYC318D      |claimed               |Fall 2017    |University of Maryland |Psychology                                     |BSOS        |Psychology                                     |
|SPAN415-0101: Commercial Spanish II-Fall 2017 fabian                                                                                     |SPAN415-0101: Commercial Spanish II-Fall 2017 fabian                                                                                     |SPAN415       |claimed               |Fall 2017    |University of Maryland |School of Languages, Literatures, and Cultures |ARHU        |School of Languages, Literatures, and Cultures |
|Summer 2017 LSAMP Undergraduate Research Program                                                                                         |Summer 2017 LSAMP Undergraduate Research Program                                                                                         |Summer2017URP |available             |Default Term |University of Maryland |ENGR                                           |ENGR        |Organizations-ENGR                             |
|KNES360-0201,0202,0203,0204,0205,0206: Physiology of Exercise-Fall 2017 rlindle                                                          |KNES360-0201,0202,0203,0204,0205,0206: Physiology of Exercise-Fall 2017 rlindle                                                          |KNES360       |claimed               |Fall 2017    |University of Maryland |Kinesiology                                    |SPHL        |Kinesiology                                    |

## Enrollment Datasources

Enrollment data is captured in to tables in Canvas: **enrollment_fact** and **enrollment_dim**.  The **enrollment_dim** 
table contains most of the columns stored in **enrollment_fact** table.  The exception are two columns: *computed_final_score* and *computed_current_score*.  However since 2017 these columns have been deprecated.  As a 
result only 40% of the entries contain data.  As a result it may be simplier to simply import **enrollment_dim** 
alone into Immuta.  The role of the user can be augmented with data contained in the role table using the following 
SQL statement:


```sql
SELECT e.id, e.canvas_id, e.root_account_id, e.course_section_id, e.type, 
  e.workflow_state as enrollment_workflow_state, e.created_at, e.updated_at, 
  e.start_at, e.end_at, e.completed_at, e.self_enrolled, 
  e.sis_source_id, e.course_id, e.user_id, r.name as sub_role_name,
  r.workflow_state as role_workflow_state
FROM enrollment_dim as e
LEFT JOIN role_dim as r
ON e.role_id = r.id
```



|user_id             |course_section_id |course_id         |type              |enrollment_workflow_state |created_at              |updated_at              |sub_role_name     |
|:-------------------|:-----------------|:-----------------|:-----------------|:-------------------------|:-----------------------|:-----------------------|:-----------------|
|244654844961525305  |11330000001557796 |11330000001227361 |StudentEnrollment |deleted                   |2017-04-17 22:22:44.404 |2017-05-06 12:33:04.47  |StudentEnrollment |
|497117251635255794  |11330000001557796 |11330000001227361 |StudentEnrollment |deleted                   |2017-04-19 22:22:44.061 |2017-05-06 12:33:04.422 |StudentEnrollment |
|-509236400883090242 |11330000001557796 |11330000001227361 |StudentEnrollment |deleted                   |2017-04-20 16:24:04.501 |2017-05-06 12:33:03.923 |StudentEnrollment |
|-1092152539084524   |11330000001557796 |11330000001227361 |StudentEnrollment |deleted                   |2017-04-23 00:24:03.232 |2017-05-06 12:31:41.417 |StudentEnrollment |
|547152121835594887  |11330000001557796 |11330000001227361 |StudentEnrollment |deleted                   |2017-04-23 00:24:03.376 |2017-05-06 12:33:04.715 |StudentEnrollment |
|332078304411371458  |11330000001557792 |11330000001227361 |StudentEnrollment |deleted                   |2017-05-23 00:33:02.471 |2017-05-27 22:32:40.808 |StudentEnrollment |

### Wiki and Wiki Page Tables

Wikis stored in Canvas are drawn from two parent types classes: *Courses* and *Groups*.  Within the Canvas ecosystem
*Group* wikis are references in the the **file_dim** table and **group_dim** tables.  Elsewhere only
*Course* wikis are referenced.  For simplicity we may want to only create a wiki datasource in Immuta 
which has been subsetted to only **Course** wikis.

If this is the case we can effectively ignore the **wiki_fact** and only load in the **wiki_dim**.  We can join the 
**wiki_dim** data to the course data on the *wiki_id* column.  Here is a SQL statement which can 
be used to generate the datasource

```sql
SELECT * 
FROM wiki_dim
WHERE parent_type = 'Course';
```


|id                |parent_type |title                                                                                              |front_page_url    |
|:-----------------|:-----------|:--------------------------------------------------------------------------------------------------|:-----------------|
|11330000000525474 |Course      |CCJS331-SG91: Contemporary Legal Policy Issues-Fall 2017 adrew12 Wiki                              |NA                |
|11330000000528418 |Course      |Dr. Zeng Wiki                                                                                      |home-page         |
|11330000000523994 |Course      |COMM400-0201: Research Methods in Communication-Fall 2017 sobae Wiki                               |NA                |
|11330000000526938 |Course      |LBSC602-SG10: Serving Information Needs-Summer I 2017 rgammons Wiki                                |table-of-contents |
|11330000000528794 |Course      |BSST399T-0101: Individual Study in Terrorism Studies; Teaching Terrorism-Spring 2017 kworboys Wiki |NA                |
|11330000000523869 |Course      |BERC601-PLR1: White-Collar Crime and Victimization-Fall 2017 dmaimon Wiki                          |home-page         |

If the *Group* wikis need to added at a later date they can be created within the following
SQL statement:


```sql
SELECT * 
FROM wiki_dim
WHERE parent_type = 'Group';
```

Wiki Pages can be loaded in a similar manner, splitting by *Group* and *Course* parent types.  We need 
to join **wiki_page_dim** and **wiki_page_fact** to be able to connect the page data to the parent wiki and 
course:


```sql
SELECT wpd.id, wpd.title, wpd.body, wpd.workflow_state as page_workflow_state, 
   wpd.created_at, wpd.updated_at, wpd.url, wpd.protected_editing, wpd.protected_editing, 
   wpd.editing_roles, wpd.revised_at, wpd.could_be_locked, wpf.wiki_id as wiki_id, wpf.view_count
FROM wiki_page_dim as wpd
LEFT JOIN wiki_page_fact as wpf
ON wpd.id = wpf.wiki_page_id
WHERE wiki_id IN (SELECT id FROM wiki_dim WHERE parent_type = 'Course')
```


|id                |title                                        |url                                         | view_count|
|:-----------------|:--------------------------------------------|:-------------------------------------------|----------:|
|11330000002088192 |Exercise Gathering Career Info Activity      |exercise-gathering-career-info-activity     |          5|
|11330000002077219 |Home                                         |home                                        |        505|
|11330000002052278 |Links for class                              |links-for-class                             |          1|
|11330000002064153 |Assignment: Current Issues in Macroeconomics |assignment-current-issues-in-macroeconomics |        158|
|11330000002063013 |Resources for Economics Coverage and Policy  |resources-for-economics-coverage-and-policy |          3|
|11330000002080118 |Lecture 4 - Notes                            |lecture-4-notes                             |         20|

The **file_dim** contains attributes for reference files.  
This table contains keys to link the files to courses, quizzes, and assignments.
Given the number of potential use cases for this table it is probably a good idea to import the table directly into
Immuta.

### Assignment and Quiz Tables

Assignment data are stored in **assignment_dim** and **assignment_rule_dim**.  The **assignment_rule_dim**
table contains of only two columns: *assignment_id* and *drop_rule*.  We can merge these two tables simply
using the following SQL statement:


```sql
SELECT * 
FROM assignment_dim
LEFT JOIN assignment_rule_dim
ON id=assignment_id
```

|id                |course_id         |title                                    |points_possible |workflow_state | position|
|:-----------------|:-----------------|:----------------------------------------|:---------------|:--------------|--------:|
|11330000004031507 |11330000001179220 |Reading #7: Land Treatment and Bioswales |0               |published      |        7|
|11330000004058621 |11330000001175160 |Exam 3                                   |150             |published      |        4|
|11330000004058627 |11330000001175160 |report                                   |50              |published      |        7|
|11330000004023545 |11330000001190682 |Library Day 1                            |20              |published      |       10|
|11330000004023541 |11330000001190682 |Final Exam Study Guide                   |NA              |unpublished    |        1|
|11330000004021677 |11330000001177444 |Syllabus Quiz                            |0               |published      |        1|

Quiz data is also separated into **quiz_dim** and **quiz_fact** tables.  We can merge those into a 
single view using the following SQL statement.  This removes redundant columns in the **quiz_fact** 
table.  

```sql
SELECT *
FROM quiz_dim as qd
LEFT JOIN (
  SELECT quiz_id, time_limit, allowed_attempts, unpublished_question_count,
    question_count
  FROM quiz_fact
) AS qf
ON id=quiz_id;
```


|id                |course_id         |assignment_id |quiz_type     |points_possible |workflow_state | question_count|scoring_policy |
|:-----------------|:-----------------|:-------------|:-------------|:---------------|:--------------|--------------:|:--------------|
|11330000001176257 |11330000001206001 |NA            |practice_quiz |12              |published      |              8|keep_highest   |
|11330000001176258 |11330000001206001 |NA            |practice_quiz |13              |published      |             11|keep_highest   |
|11330000001176264 |11330000001206001 |NA            |practice_quiz |11              |published      |              2|keep_highest   |
|11330000001176267 |11330000001206001 |NA            |practice_quiz |10              |published      |             10|keep_highest   |
|11330000001176269 |11330000001206001 |NA            |practice_quiz |6               |published      |              6|keep_highest   |
|11330000001176283 |11330000001206001 |NA            |practice_quiz |10              |published      |             10|keep_highest   |

The quiz and assignment data can be joined on *assignment_id* on the quiz table and *id* on the assignment table, 
but for flexiblity it is probably a good idea to keep these two tables separate tables in Immuta and 
joined within an analytics environment.

The **module_progression_completion_requirement_fact** contains measures related to the module completion 
requirements.  This table contains keys to link the module progression to wiki, wiki pages, quizzes, and assignments.
Given the number of potential use cases for this table it is probably a good idea to import the table directly into
Immuta.

### Communications

Blank

### User Table

The user information is stored in the **user_dim** table.  Like **module_progression_completion_requirement_fact** 
and **file_dim**, this table will potentially be used in numerous use cases.  As a result it should be imported 
directly into Immuta.



## Connecting RStudio and Immuta

R has libraries to interface with a PostgreSQL data source used by Immuta. These libraries include **RODBC** and **RPostgeSQL**.  This section will detail how **RODBC** can be used to connect to Immuta data sources.

Prior to installing **RODBC**, Postgres ODBC Drivers must be installed on your system, and ODBC datasources must be setup.  There are step by step guides on doing this in Windows ([Windows Instructions](https://cran.r-project.org/web/packages/RODBC/vignettes/RODBC.pdf)).  On Windows, download an ANSI PostgreSQL Driver ([here](https://www.postgresql.org/ftp/odbc/versions/msi/)).  Install the driver.  Open the ODBC Data Sources and add a new data source using the ANSI Postgres Driver.  Enter the server name, port, user name, password, and database.  Set *SSL Mode* to *Require*.

The Immuta server name and your Immuta user name can be seen in the SQL Credential screen discussed in the **Getting Started** section.

In the case of UMD's Immuta instance the connection details are:

Username = immuta_admin
Password = XXXXXXXXXXXXXXXXXXX
Server Name = immuta1.it-prod-cots.aws.umd.edu
Port = 5432
Database = immuta
SSLMode = require

Once this is edited install **RODBC** using the following command from the command line:


```eval
R CMD INSTALL RODBC_1.3-8.tar.gz
```


## Sample Analysis with R and Immuta

