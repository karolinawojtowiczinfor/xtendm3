---
layout: default
title: MI API
parent: API Specification
grand_parent: Documentation
nav_order: 12
---

# MI API
{: .no_toc }

Checklist for requesting approval for your extensions.
{: .fs-6 .fw-300 }
![](/assets/images/warning-24px.svg)

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

**Since**: `0.9.0`  
**Context**: XtendM3 API  
**Category**: Context Specific

---
## Description
MI API allows to read input and write output parameters sent into transactions. Supports both single and multiple transactions, custom logic and validation.  Can be used to abort transaction based on own validation. Provides error managment. 

## Features

### Getting value from data container
mi.in containes In values for Transactions. Contained values are already converted to the proper type.
Takes string key as a parameter and returns value associeted with the key.

Example 1:
```groovy
def <T> Optional<T> getParameter(String name, Class<T> type) {
    if (type == Double.class) {
      String raw = mi.in.get(name)
      if (raw == null || raw.trim().isEmpty()) {
        return Optional.empty()
      }
      return Optional.of((mi.in.get(name) as Double) as T)
    }
    return Optional.empty()
  }
```
<br>

Example 2:
```groovy
private String whlo;
private String itno;
private String whsl;

public void main() {
  whlo = mi.inData.get("WHLO").trim();
  itno = mi.inData.get("ITNO").trim();
  whsl = mi.inData.get("WHSL").trim();
}
```

### Putting value into data container
Takes key and associeted with it value as parameters, returns previous value.

Example 1:
```groovy
public void main() {
  mi.outData.put("WHLO", "This is WHLO Output#1 for QQS001")
  mi.outData.put("ITNO", "This is ITNO Output#2 for QQS002")
  mi.write() 
}
```
<br>

Example 2:
```groovy
void main() {
  mi.outData.put("WHLO", String.valueOf(getParameter("NUMB", Double.class).orElse(null)))
}
```
### Checking date format
Returns date format as a string. Possible formats: YMD8, YMD6, MDY6, DMY6, YWD5.

Example:
```groovy
void main() {
  if (mi.getDateFormat(date1) == "YMD8"){
    return LocalDate.now().format(DateTimeFormatter.ofPattern("yyyyMMdd"))
  }
}
```
### Checking date separator
Returns date separator as a char.

Example:
```groovy
void ReadSeparator () {
sep = mi.getDateSeparator() 
String oldDate = '04-DEC-2012'
Date date = Date.parse( 'dd-MMM-yyyy', oldDate )
String newDate = date.format( 'M'{sep}'d'{sep}'yyyy' )
}
```

### Checking remaining records
Returns the amount of remaining records allowed to be written in current transaction.

Example:
```groovy
void main() {
  if (mi.getRemainingRecords()>= 2) {
    for (int i = 0; i < 3; i++) {
      mi.outData.put("WHLS" + WHSL)
      mi.write()
    }
  }
}
```

### Checking if there are remaining records
Returns true if there are still any additional records to be written or false if not.

Example:
```groovy
void main() {
  if (mi.hasRemainingRecords()){
    mi.outData.put("MMRI" + SLDQ)
    mi.write()
  }
}
```

### Checking maximum of possible records
Returns the maximum amount of possible records to be written as an integer.

Example:
```groovy
public void main() {
  String WHSL = mi.in.get("WHSL")
  for (int i = 0; i < mi.getMaxRecords(); i++) {
    mi.outData.put("WHLS" + WHSL + " " + i)
    mi.write()
  }
}
```

### Clearing previous value
Writes response from data present in outData. Clears all values in outData.

Example:
```groovy
public void main() {
  mi.outData.put("WHLO","Test")
  mi.write()
}
```

### Writting error responses
Writes error responses with parameters:
String errorMessage - error message to display (only supports 240 characters);
String field - field error occurred for (only supports 10 characters);
String errorCode - code for error which occurred (only supports 2 characters).

Example 1:
```groovy
boolean validateInput() {
  if (!getWarehouse(whlo)){
    mi.error("Warehouse " + whlo + " is invalid");
    return false;
  }
}
```
<br>

Example 2:
```groovy
def boolean validResponsible(String Responsible) {
  if (Responsible.isEmpty()){
    mi.error("Responsible must be entered");
    return false;
  }
}
```
## Considerations and Guidelines
It is not possible to add extra fields to transaction using MI API.