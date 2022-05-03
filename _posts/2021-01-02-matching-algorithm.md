---
title: Matching Algorithm
permalink: /:Matching Algorithm/
---

This is by far the longest and most in depth amount of code that I've written to date. I will not share it here since it includes sensitive data that altering would only make more confusing. Instead, I will describe the business case for it.

### **Background**
I was tasked with taking invoicing and Salesforce Opportunity data to match the two together on a client-month level basis. The purpose was to determine if we had under or overbilled any customers in the history of the company (10 years0. Although a seemingly simple task, the messiness of the data proved to be the real challenge. Dates were misaligned. One-off credits were given years ago with no accompanying documentation. End dates before start dates... Needless to say every data error that could occur happened in each dataset. Lastly, there was no common unique ID's between the datasets to index on. The short answer to an exceptionally long solution was to import millions of rows of real-time data, clean it, build-in logic, and derive tables to match based on date, account name, and if amounts were within a certain tolerance limit.

### **Outcome**
Depending on the customer, the code would run and match invoices to opportunities within a matter of seconds and decipher in which months a client was over/underbilled or if the account was flat and for what amount. The script created a mapping table to upload to Salesforce en masse to be used in further deferred revenue calculations in other systems. The entirety of the output was also given to consultants to use in a different project, which sped up their production build. Although difficult to quantify, I can confidently say that without this algorithm it would have taken a team of full-time employees over a year to do by hand with less accuracy.

### **Lessons Learned**
1. Do not take clean data for granted
2. Take steps to implement infrastructure to eliminate the need for data cleanup due to discrepancies
3. Unique ID's shared between systems is incredibly important

**I am happy to share the code over Zoom and walk through at a high-level upon request.**
