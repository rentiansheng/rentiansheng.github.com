---
layout: post
title: Golang EXCEL column, cell data validation (drop-down list, numeric text length check)
category: [golang]
tags: [golang, excel, data validation]
---



## 1. remise
---

Recently used in Golang development projects used excel import, export. Data export is no problem, just write excel on the line. However, importing templates has encountered some problems.

Questions are as follows:
   
    1. The data in db is defined as a string. If the field is filled with numbers, the field is a number when the data is acquired. The backend conversion is required.
    2. Enumerate the type fields, fill in too difficult. Users need to know the corresponding value
    3. The field range value, fill in more difficult, (such as user, type, etc.)
    3  can not limit the length of the field to fill in the content
    4. Cannot limit the range of numbers

Why not directly put an excel file, the data in the project is imported, the export field is not fixed, you need to import according to the personal configuration real field, export

## 2. Golang EXCEL Class Library Selection
---

After the Internet search found two more use now more golang operation excel class library
1. [excelize](https://github.com/360EntSecGroup-Skylar/excelize)
2. [xlsx](https://github.com/tealeg/xlsx)

In most cases, xlsx is selected, and xlsx is more in star and fork than excelle. However, excelize is more active in recent activity and the document is more detailed (in Chinese)


## 3. Use EXCEL data validation
---
In the use of xlsx, there is no problem with basic functions, but when doing advanced functions, it is found that many advanced functions do not have time, and some implementations are too rude. The data verification function is not implemented at all. Originally wanted to copy excelize the code, and found that they did not achieve, still lying in the todo list. Helplessness can only be achieved by yourself.

Implement the function code in:
The [support data validation](https://github.com/rentiansheng/xlsx) code has been submitted to the PR but has not been merged.
Now it's simple and crude to implement the list of excel data validation, rang validation (time, date need to be converted to numbers, now unimplemented) has confirmed that available data validation has list (display dorp box), rang (number, decimal, text length ), User input will be prompted for incorrect content.

An example of a test code for data validation is at [data validation test code](https://github.com/rentiansheng/xlsx/blob/dev_master/datavalidation_test.go)
You can go test to view the production excel content.

## 4. Need to prepare for the development of EXCEL
---

Excel is a zip file described by openxml.
The files in the office are all similar. Microsoft has published the document.

tool:
Open xml sdk 2.0 tool, can view the openxml code of excel file, provide verification, documentation, generate code function (not golang)