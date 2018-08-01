# Backend Coding Exercise
The following is intended as a coding exercise for backend developers.

## Avero Point of Sale System API

A restaurant Point of Sale (POS) is a system for managing the transactions within a restaurant. If you have ever seen a server enter your table’s order into a touch screen and then run your credit card through it at the end of the meal, they were interacting with the POS.  Many POS systems expose an API for 3rd party integration.
For this exercise you will be retrieving sales and labor data for a set hypothetical restaurants from an API provided by the fictional Avero Point of Sale System.

## Table of contents

- [POS Entities](#POS-Entities)
- [Report Types](#Report-Types)
- [Exercise](#Exercise)
- [POS API Documentation](#POS-API-DOCUMENTATION)
- [Reporting API Documentation](#Reporting-Data-API-DOCUMENTATION)

---

## POS Entities
### Businesses 
- A business represent a restaurant is a POS.
```json
{
     "id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
     "name":"Red BBQ",
     "hours":[11,12,13,14,15,16,17,18,19,20,21,22],
     "updated_at":"2018-07-27T16:34:12.910Z",
     "created_at":"2018-07-27T16:34:12.910Z"
}
```

### Check
checks represents:
- Party at the restaurant.
- Single guest who placed an order at restaurant.
```json
{
     "id":"f749b9ff-601b-4992-8b51-fc918310ee35",
     "business_id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
     "employee_id":"b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
     "name":"Andy Bolden",
     "closed":true,
     "closed_at":"2018-04-01T17:35:00.000Z",
     "updated_at":"2018-07-31T00:56:16.972Z",
     "created_at":"2018-04-01T17:03:00.000Z"
}
```

#### Ordered Item
- Represents a menu item that has been added to a check.
- Ordered items contains data such as the ```(employee_id, check_id, business_id, menu_id)``` so that you we can reference back to the corresponding entities that relates to the ordered item.
- ```cost``` is the amount of money that it cost the business to prepare.
- ```price``` is the amount of money that is charged to the customer.
- Ordered items can be ```voided```. If and ordered item is voided the value of ```price``` of this item is not computed into the total of the check.
- Ordered items contains the cost of the related menu item at the time an item was ordered and added to a check.

You can think of an ordered item as a snapshot in time of a menu item added to a check.
Ordered items are used to calculate sales related metrics,
Ordered items are used to calculate menu item cost & performance metrics as well. 

```json
{
    "id": "cb889c5c-c969-4ac1-a25b-70cd28c83f76",
    "business_id": "b2aeb27b-c85c-4ad8-83d4-d9511063d418",
    "employee_id": "b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
    "check_id": "5b99a5eb-d426-4867-990a-80ebe1e0617f",
    "item_id": "d1b1299c-54dc-4163-8ee1-04488f50c071",
    "name": "Reds Brisket",
    "cost": 10,
    "price": 17,
    "voided": false,
    "updated_at": "2018-05-02T20:30:00.000Z",
    "created_at": "2018-07-31T21:14:41.413Z"
} 
```  

### Menu Items
menu items represents:
- some food or beverage that can be order and added to a check as an ordered item as seen above under check.
- ```cost``` is the amount that it cost the business to make prepare the a menu item.
- ```price``` is the amount that is charged to the customer.
```json
{
     "id":"a7c9f9f0-4936-4a4a-88d7-f9f76ebce1bd",
     "business_id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
     "name":"Smokin Ribs",
     "cost":12,
     "price":20,
     "updated_at":"2018-07-27T16:46:19.847Z",
     "created_at":"2018-07-27T16:46:19.847Z"
}
```
  
### Employees
employees represents
- individual who is employed at a business.
- employees work at one business.
- employees have labor entries indicating the time worked.
```json
{
     "id":"f749b9ff-601b-4992-8b51-fc918310ee35",
     "business_id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
     "employee_id":"b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
     "name":"Andy Bolden",
     "closed":true,
     "closed_at":"2018-04-01T17:35:00.000Z",
     "updated_at":"2018-07-31T00:56:16.972Z",
     "created_at":"2018-04-01T17:03:00.000Z"
}
```
    
### Labor Entries
labor entries represents:
- a ```clock_in``` time and ```clock_out``` time for a given employee including the employee ```pay_rate```.
```json
{
    "id": "feb99737-83e5-4989-98ad-e8ad3ad228d7",
    "business_id": "f21c2579-b95e-4a5b-aead-a3cf9d60d43b",
    "employee_id": "b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
    "name": "Andy Bolden",
    "clock_in": "2018-05-03T15:00:00.000Z",
    "clock_out": "2018-05-03T21:00:00.000Z",
    "pay_rate": 11,
    "updated_at": "2018-07-30T19:23:32.265Z",
    "created_at": "2018-07-30T19:23:32.265Z"
}
```
You can think of an labor entry as a work shift and the given rate of pay at the time the employee worked that shift.

Labor entries are used to calculate labor related reports. 
     

## Report Types
#### Labor Cost Percentage
 - Abbreviated as **LCP**
 - Labor cost percentage is the percentage of your revenue that pays for labor. 
 - Calculate labor cost percentage: **Labor Cost Percentage = Labor / Sales**
 
     Here’s an example: 
      
     Let’s say you’ve added all money paid out to hourly employees over a week. 
     The total was $5,500. During that same time period, you brought in $22,000 worth of revenue. 
       
     Labor Cost Percentage = $5,500 / $22,000
      
     Labor Cost Percentage = 25%

#### Food Cost Percentage 
 - Abbreviated as **FCP**
  - Food cost percentage is the difference between what it costs to produce an item and its price on the menu. 
  - Calculate food cost percentage: **Food Cost Percentage = Item Cost / Selling Prices**
  
      Here’s an example: 
      
      A burger is priced at $13 and costs $4 to make.
     
      Food Cost Percentage = $4 / $13
      
      Food Cost Percentage = 31% 

#### Employee Gross Sales
 - Abbreviated as **EGS**
 - Employee gross sales is the sum of the of the price charged for each item the employee sold. Voided Items excluded.
 - Calculate employee gross sales: **Employee Gross Sales = Sum(Selling Prices)**
 
     Here’s an example:
     
     A burger is priced at $13 and a shake $7.
     
     Employee Gross Sales = $13 + $7
     
     Employee Gross Sales = $20
     


## Exercise
- Extract all data from the POS API. [POS API Documentation](#POS-API-DOCUMENTATION)
    - The api endpoints are documented below. 
    - There are a fixed number of businesses with a fixed number of menu items, employees, labor entries, checks and ordered items.
    - You will need to persist this data in a database of persistence layer of your choice.
    
- You will need to develop a model for reporting and aggregation. 
  The [report types](#Report-Types) that you will need to calculate are provided above along with the corresponding formula for calculating it. [Report Types](#Report-Types)

API users will be consuming your Reporting API to build our reporting dashboard.
      
- You will need to develop the http Reporting API which is documented below. [Reporting API Documentation](#Reporting-Data-API-DOCUMENTATION) 
    - Some Assertions         
        - Each business has a ```hours``` property and is open every day. All checks and labor will be within these business hours. 
        - Each check is associated with one and only one business and employee.
        - Each check is closed and has a ```closed_at``` property indicating the time it was closed.
        - Each check is closed in the same hour it was open. ie. 2:00pm - 2:59pm
        - Each check may have any number of ordered items associated with it.
        - Each ordered item has a set boolean property ```voided``` indicating if the check is voided. 
        - Each ordered item has a ```created_at``` date. created_at is to be used when calculating reports.
        
Given a start and end dates, a business id, a report type, and a time interval you will need to calculate the report for the report type, constrained by start date, end date and business_id, aggregated by the time interval.

## Expectations:
- You may use any language, library, database, and framework that you wish. Ultimately, you should choose whatever tools will best enable you to deliver a quality product.
- The API endpoints that we have provided should be all that you need.
- There may some ambiguity in the business requirements. You are free to make assumptions where you feel like the requirements are unclear. If you do, please document these assumptions in the README.
- This exercise should take about 4-8 hours to complete. It should represent your best effort - but we understand that it is a side project. We do not expect perfection. =)
- There is no one approach we are looking for, however you should be prepared to discuss the decisions and design choices made.

## Delivery
- Your final output should be a link to a publicly hosted repo (e.g. GitHub, bitbucket) which includes all of your code, assets, etc. Anything that we need to run your project and wish us to evaluate.
- Your code should include a README file with clear instructions for running your solution. Any dependencies, build commands, etc - every step after `git clone …` should be clearly documented. If we can’t run it, we can’t evaluate it!
- If you wish to include any other resources (design docs, planning breakdown, etc), you may reference and link them from the README.
- When you're ready for us to review, you should email whoever you have been in contact with at Avero.

## You will be evaluated on (in roughly this order)
- Our ability to access and run your code. If we can’t run it, we can’t evaluate it.
- Your adherence to the business and technical requirements outlined above. Beautiful code doesn’t matter if it doesn’t work correctly!
- The quality of your code. Think about good engineering practices - legibility, maintainability, separation of concerns, usability. This should be code that you are proud of.

## API Bugs
If you are reading this then you are one of the first people to do this code exercise and there are probably bugs in the API!

If you come across a bug, please help us out and [file an issue](https://github.com/AveroLLC/reporting-api-exercise/issues).

# POS API DOCUMENTATION

## Connecting
All URIs in this document have the following base:
```https://AVOS-api.herokuapp.com```

## Content-Type
Any data in request or response bodies should be JSON.

## Authentication
For every request, you must send an access token in the _Authorization_ header.  If you're planning on submitting this exercise to us, you should have already received an access token, so let us know if you haven't.

### GET /businesses - list businesses
##### Query parameters:
- **limit** (number) - The amount of results to return is 100 by default and and the max is 500.
- **offset** (number) - The amount of results to skip the default is 0.

example response body:
```json

{
   "count":3,
   "data":[
      {
         "id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
         "name":"Red BBQ",
         "hours":[
            11,12,13,14,15,16,17,18,19,20,21,22
         ],
         "updated_at":"2018-07-27T16:34:12.910Z",
         "created_at":"2018-07-27T16:34:12.910Z"
      }
   ]
}

```

### GET /menuItems  - list menu items
list the menu items 
##### Query parameters:
- **limit** (number) - The amount of results to return is 100 by default and and the max is 500.
- **offset** (number) - The amount of results to skip the default is 0.
- **business_id** (uuid) - The business_id of record used to constrain the results.

example response body:
```json
{
   "count":1,
   "data":[
      {
         "id":"a7c9f9f0-4936-4a4a-88d7-f9f76ebce1bd",
         "business_id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
         "name":"Smokin Ribs",
         "cost":12,
         "price":20,
         "updated_at":"2018-07-27T16:46:19.847Z",
         "created_at":"2018-07-27T16:46:19.847Z"
      }
   ]
}
```

### GET /checks - list all checks with basic info
- **limit** (number) - The amount of results to return is 100 by default and and the max is 500.
- **offset** (number) - The amount of results to skip the default is 0.
- **business_id** (uuid) - The business_id of record used to constrain the results.

example uri: ```/checks```

example response body:
```json
{
   "count": 1,
   "data":[
      {
         "id":"f749b9ff-601b-4992-8b51-fc918310ee35",
         "business_id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
         "employee_id":"b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
         "name":"Andy Bolden",
         "closed":true,
         "closed_at":"2018-04-01T17:35:00.000Z",
         "updated_at":"2018-07-31T00:56:16.972Z",
         "created_at":"2018-04-01T17:03:00.000Z"
      }
   ]
}
```

### GET /orderedItems - list all checks with basic info
- **limit** (number) - The amount of results to return is 100 by default and and the max is 500.
- **offset** (number) - The amount of results to skip the default is 0.
- **business_id** (uuid) - The business_id of record used to constrain the results.

example uri: ```/orderedItems```

example response body:
```json
{
   "count": 1,
   "data":[
      {
          "id": "cb889c5c-c969-4ac1-a25b-70cd28c83f76",
          "business_id": "b2aeb27b-c85c-4ad8-83d4-d9511063d418",
          "employee_id": "b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
          "check_id": "5b99a5eb-d426-4867-990a-80ebe1e0617f",
          "item_id": "d1b1299c-54dc-4163-8ee1-04488f50c071",
          "name": "Reds Brisket",
          "cost": 10,
          "price": 17,
          "voided": false,
          "updated_at": "2018-05-02T20:30:00.000Z",
          "created_at": "2018-07-31T21:14:41.413Z"
      }
   ]
}
```

### GET /employees - list all employees
- **limit** (number) - The amount of results to return is 100 by default and and the max is 500.
- **offset** (number) - The amount of results to skip the default is 0.
- **business_id** (uuid) - The business_id of record used to constrain the results.

example uri: ```/employees?business_id=b2aeb27b-c85c-4ad8-83d4-d9511063d418```

example response body:
```json
{
   "count": 1,
   "data":[       
      {
         "id":"f749b9ff-601b-4992-8b51-fc918310ee35",
         "business_id":"b2aeb27b-c85c-4ad8-83d4-d9511063d418",
         "employee_id":"b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
         "name":"Andy Bolden",
         "closed":true,
         "closed_at":"2018-04-01T17:35:00.000Z",
         "updated_at":"2018-07-31T00:56:16.972Z",
         "created_at":"2018-04-01T17:03:00.000Z"
      }
   ]
}   
```

### GET /laborEntries - list all labor entries
- **limit** (number) - The amount of results to return is 100 by default and and the max is 500.
- **offset** (number) - The amount of results to skip the default is 0.
- **business_id** (uuid) - The business_id of record used to constrain the results.
- **employee_id** (uuid) - The employee_id of record used to constrain the results.

example uri: ```/laborEntries?employee_id=b63d9159-b2f9-4fd0-bb53-5dfeaee89e85```

example response body:
```json
{
   "count": 1,
   "data":[
      {
        "id": "feb99737-83e5-4989-98ad-e8ad3ad228d7",
        "business_id": "f21c2579-b95e-4a5b-aead-a3cf9d60d43b",
        "employee_id": "b63d9159-b2f9-4fd0-bb53-5dfeaee89e85",
        "name": "Andy Bolden",
        "clock_in": "2018-05-03T15:00:00.000Z",
        "clock_out": "2018-05-03T21:00:00.000Z",
        "pay_rate": 11,
        "updated_at": "2018-07-30T19:23:32.265Z",
        "created_at": "2018-07-30T19:23:32.265Z"
      }
   ]
}   
```

# Reporting Data API DOCUMENTATION

### Report Type Abbreviations
- [Labor Cost Percentage = **LCP**](#Labor-Cost-Percentage)
- [Food Cost Percentage = **FCP**](#Food-Cost-Percentage )
- [Employee Gross Sales = **EGS**](#Employee-Gross-Sales ) 


### GET /reporting - get reporting data
##### Query parameters:
- **business_id** (uuid) - The id of the business to run this report for.
- **report** (LCP | FCP | SBE) - The name abbreviated name of the report to run.
- **timeInterval** (hour | day | week | month) - The time interval to aggregate the data.
- **start** (date) - The start date used to constrain the results. ISO-8601 date
- **end** (date) - The end date used to constrain the results. ISO-8601 date.

Given a start and end dates, a business id, a report type, and a time interval you will need to calculate the report for the report type, constrained by start date, end date and business_id, aggregated by the time interval.

### LCP:
example uri: ```/reporting?busisness_id=f21c2579-b95e-4a5b-aead-a3cf9d60d43b&report=LCP&timeInterval=hour&start=2018-05-03T15:00:00.000Z&end=2018-05-03T18:00:00.000Z```

example response body of LCP by hour:
```json
{
  "report": "LCP",
  "timeInterval": "hour",
  "data": [
    {
      "timeFrame": {
          "start": "2018-05-03T15:00:00.000Z",
          "end": "2018-05-03T16:00:00.000Z"
      },
      "value": 13.0 
    },
    {
      "timeFrame": {
          "start": "2018-05-03T16:00:00.000Z",
          "end": "2018-05-03T17:00:00.000Z"
      },
      "value": 54.0 
    }, 
    {
      "timeFrame": {
          "start": "2018-05-03T17:00:00.000Z",
          "end": "2018-05-03T18:00:00.000Z"
      },
      "value": 23.0 
    }
 ]
}
```

example uri: ```/reporting?busisness_id=f21c2579-b95e-4a5b-aead-a3cf9d60d43b&report=LCP&timeInterval=day&start=2018-05-01T00:00:00.000Z&end=2018-05-02T00:00:00.000Z```

example response body of LCP by day:    
```json
{
  "report": "LCP",
  "timeInterval": "day",
  "data": [
    {
      "timeFrame": {
          "start": "2018-05-01T00:00:00.000Z",
          "end": "2018-05-02T00:00:00.000Z"
      },
      "value": 40.0
    }
 ]
}
```

### FCP:
example uri: ```/reporting?busisness_id=f21c2579-b95e-4a5b-aead-a3cf9d60d43b&report=FCP&timeInterval=hour&start=2018-05-03T15:00:00.000Z&end=2018-05-03T18:00:00.000Z```

example response body of FCP by hour:
```json
{
  "report": "FCP",
  "timeInterval": "hour",
  "data": [
    {
      "timeFrame": {
        "start": "2018-05-03T15:00:00.000",
        "end": "2018-05-03T16:00:00.000Z"
      },
      "value": 50.0
    },
    {
      "timeFrame": {
        "start": "2018-05-03T16:00:00.000",
        "end": "2018-05-03T17:00:00.000Z"
      },
      "value": 70.0
    },
    {
      "timeFrame": {
        "start": "2018-05-03T17:00:00.000",
        "end": "2018-05-03T18:00:00.000Z"
      },
      "value": 10.0
    }
    
 ]
}
```



### EGS:

example uri: ```/reporting?busisness_id=f21c2579-b95e-4a5b-aead-a3cf9d60d43b&report=EGS&timeInterval=hour&start=2018-05-03T15:00:00.000Z&end=2018-05-03T18:00:00.000Z```

example response body of EGS by hour:
```json
{
  "report": "EGS",
  "timeInterval": "hour",
  "data": [
    {
      "timeFrame": {
        "start": "2018-05-03T15:00:00.000",
        "end": "2018-05-03T16:00:00.000Z"
      },
      "employee": "Andy Bolden",
      "value": 200.00  
    },
    {
      "timeFrame": {
        "start": "2018-05-03T16:00:00.000",
        "end": "2018-05-03T17:00:00.000Z"
      },
      "employee": "Andy Bolden",
      "value": 100.00  
    },
    {
      "timeFrame": {
        "start": "2018-05-03T15:00:00.000",
        "end": "2018-05-03T16:00:00.000Z"
      },
      "employee": "Sam Wise",
      "value": 80.00  
    },
    {
      "timeFrame": {
        "start": "2018-05-03T16:00:00.000",
        "end": "2018-05-03T17:00:00.000Z"
      },
      "employee": "Sam Wise",
      "value": 300.00  
    }
    
 ]
}
```
    

    