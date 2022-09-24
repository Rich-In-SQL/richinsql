---
title: Validate NHS Number Using A SQL Server Scalar Function 
date: 2022-09-26T09:00:00.000+01:00
author: Rich
layout: post
categories:
- sqlserver
draft: false
tags:
- SQLServer
- T-SQL
- Advanced
---

I was talking to someone recently about NHS Numbers, if you don't know, an [NHS Number](https://www.datadictionary.nhs.uk/attributes/nhs_number.html) is a unique identifier issued to each person, this number should not differ between organisations or patients and it is recommended that it is recorded in most cases when a patient is seen in most medical settings. 
While we were talking my friend asked me did I know that there is a formula you use against all of the numbers in an NHS Number and the result will be the last digit of that NHS Number and that is how you check if the NHS Number is valid or not. 

I had no idea that this is how you check if an NHS Number was valid, for example, if we take this NHS Number `628 104 7362` calculate the first 9 digits using the [modulus 11](https://www.datadictionary.nhs.uk/attributes/nhs_number.html) algorithm the result should be 2 *(The last digit)* which is the check digit this would result in the NHS Number being returned as valid, pretty neat right?! 

I was fascinated how this would work in reality, so I created a T-SQL Scalar Function for Microsoft SQL Server that will check the NHS Number passed to it and return if it is true - you can find the code for the function [here](https://github.com/Rich-In-SQL/NHSNumberCheck) but in true Rich In SQL fashion, let's break down how the function actually works. 

First, we need to replace any spaces in the NHS Number, sometimes NHS Numbers are in 3 digit blocks which we don't want

```
SET @NHSNumber = REPLACE(@NHSNumber,' ','')
```

Then we are going to make sure that the NHS Number provided is actually numerical

```
IF @NHSNumber NOT LIKE '[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]' SET @NumericCheck = 0 ELSE SET @NumericCheck = 1
```

If the NumericCheck is true we can perform calculations, otherwise set to invalid

```
IF (@NumericCheck = 1)
```

Next, we check the length of the NHS Number to make sure it is 10 digits long

```
SET @NHSLength = LEN(@NHSNumber)
```

Get each value at it's corresponding position, Multiply each of the first nine digits by a weighting factor and add them together

```
SET @Calculation = 
(
    (SUBSTRING(@NHSNumber,1,1) * 10.0) + 
    (SUBSTRING(@NHSNumber,2,1) * 9.0) + 
    (SUBSTRING(@NHSNumber,3,1) * 8.0) + 
    (SUBSTRING(@NHSNumber,4,1) * 7.0) + 
    (SUBSTRING(@NHSNumber,5,1) * 6.0) +
    (SUBSTRING(@NHSNumber,6,1) * 5.0) +
    (SUBSTRING(@NHSNumber,7,1) * 4.0) +
    (SUBSTRING(@NHSNumber,8,1) * 3.0) +
    (SUBSTRING(@NHSNumber,9,1) * 2.0)
)
```

Set the check digit to the 10th digit from the NHS Number itself

```
SET @CheckDigit = SUBSTRING(@NHSNumber,10,1)
```

Divide the total by 11 and establish the remainder

```
SET @Calculation = 11 - (@Calculation % 11)
```

If the NHS Number is 10 digits long and the calculation matches the 10th digit (CheckDigit) it is valid

```
IF (@NHSLength = 10 AND @Calculation = @CheckDigit)
SET @ValidNumber = 1
```

Otherwise, set the NHS Number to invalid

```
ELSE SET @ValidNumber = 0
```

Once we finished, return the validity of the NHSNumber to the user

```
RETURN @ValidNumber
```
