---
title: "SEMIJOIN (U-SQL) | Microsoft Docs"
ms.custom: ""
ms.date: "03/10/2017"
ms.reviewer: ""
ms.service: "data-lake-analytics"
ms.suite: ""
ms.tgt_pltfrm: ""
ms.topic: "reference"
ms.assetid: 920e3027-3006-4a95-9a4a-909c63128df4
caps.latest.revision: 14
author: "MikeRys"
ms.author: "mrys"
manager: "ryanw"
---
# SEMIJOIN (U-SQL)
Semijoins are U-SQL’s way filter a rowset based on the inclusion of its rows in another rowset. Other SQL dialects express this with the `SELECT * FROM A WHERE A.key IN (SELECT B.key FROM B)` pattern. There are two variants: **`LEFT SEMIJOIN`** and **`RIGHT SEMIJOIN`**.  
  
A `LEFT SEMIJOIN` (or just `SEMIJOIN`) gives only those rows in the left rowset that have a matching row in the right rowset.  
  
The `RIGHT SEMIJOIN` gives only those rows in the right rowset that have a matching row in the left rowset.  
  
The join expression in the ON clause specifies how to determine the match.  
  
### Examples 
Given the following rowsets:  
  
| **EmpName** |   **DepID**        |  
|------------|-----------|  
| Rafferty   | 31        |  
| Jones      | 33        |  
| Heisenberg | 33        |  
| Robinson   | 34        |  
| Smith      | 34        |  
| Williams   | *null*    |  
  
| **DeptID** |    **DepName**         |  
|--------------|-------------|  
| 31           | Sales       |  
| 33           | Engineering |  
| 34           | Clerical    |  
| 35           | Marketing   |  
  
The following query finds all employees that are in valid departments by finding all the employees in the left @employees rowset that have a depID that is listed in the right @departments rowset):  
  
```sql
@employees = SELECT *  
               FROM (VALUES   
                      ("Rafferty", (int?) 31)  
                    , ("Jones", (int?) 33)  
                    , ("Heisenberg", (int?) 33)  
                    , ("Robinson", (int?) 34)  
                    , ("Smith", (int?) 34)  
                    , ("Williams", (int?) null)) AS E(EmpName, DepID);  
                      
@departments = SELECT *  
                FROM (VALUES  
                       ((int) 31, "Sales")  
                     , ((int) 33, "Engineering")  
                     , ((int) 34, "Clerical")  
                     , ((int) 35, "Marketing")) AS D(DepID, DepName);  
                       
@emps_in_valid_dept =  
    SELECT e.EmpName, e.DepID  
    FROM @employees AS e  
         LEFT SEMIJOIN (SELECT (int?) DepID AS DepID, DepName FROM @departments) AS d  
         ON e.DepID == d.DepID;  
  
OUTPUT @emps_in_valid_dept   
TO "/output/rsLeftSemiJoinEmployeesInValidDept.csv"  
USING Outputters.Csv();
```
  
The resulting rowset looks like:  
  
| **EmpName** |   **DepID**        |  
|------------------------|-----------|  
| Rafferty               | 31        |  
| Jones                  | 33        |  
| Heisenberg             | 33        |  
| Robinson               | 34        |  
| Smith                  | 34        |  
  
The following query finds all departments with at least one employee:  
  
```sql
@employees = SELECT *  
               FROM (VALUES   
                      ("Rafferty", (int?) 31)  
                    , ("Jones", (int?) 33)  
                    , ("Heisenberg", (int?) 33)  
                    , ("Robinson", (int?) 34)  
                    , ("Smith", (int?) 34)  
                    , ("Williams", (int?) null)) AS E(EmpName, DepID);  
                      
@departments = SELECT *  
                FROM (VALUES  
                       ((int) 31, "Sales")  
                     , ((int) 33, "Engineering")  
                     , ((int) 34, "Clerical")  
                     , ((int) 35, "Marketing")) AS D(DepID, DepName);  
                       
@depts_with_emps =  
    SELECT d.DepName, d.DepID  
    FROM @employees AS e  
         RIGHT SEMIJOIN (SELECT (int?) DepID AS DepID, DepName FROM @departments) AS d   
         ON e.DepID == d.DepID;  
  
  
OUTPUT @depts_with_emps   
TO "/output/rsRightSemiJoinDepartmentsWithEmployees.csv"  
USING Outputters.Csv();
```
  
The query return the rowset:  
  
| **DepName**  |    **DepID**       |  
|---------------------------------------------|-----------|  
| Sales                                       | 31        |  
| Engineering                                 | 33        |  
| Clerical                                    | 34        |  

**Additional Example**  
An example of comparing SEMIJOIN against a subquery with IN is available at [Subqueries with IN/NOT IN and SEMIJOIN/ANTISEMIJOIN](common-sql-expressions-in-u-sql.md#subQuery).

### See Also 
* [U-SQL SELECT Selecting from Joins](u-sql-select-selecting-from-joins.md)  
* [SELECT Expression (U-SQL)](select-expression-u-sql.md) 
* [Query Statements and Expressions (U-SQL)](query-statements-and-expressions-u-sql.md)  
* [Data Modification Language (DML) Statements (U-SQL)](data-modification-language-dml-statements-u-sql.md)   
* [Output Statement (U-SQL)](output-statement-u-sql.md)
* [U-SQL Primary Rowset Expressions](query-statements-and-expressions-u-sql.md#pri_row_exp)  






