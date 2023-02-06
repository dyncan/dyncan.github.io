---
layout: post
title: "Tips for Crafting Efficient and Scalable Code in Salesforce"
subtitle: ""
date: 2023-02-06 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-design-code-in-salesforce.jpeg"
catalog: true
tags:
  - Scalable
  - Design
  - Best Practice
  - English
---

In this post, I will share tips on crafting efficient and scalable code in Salesforce. As an experienced developer, I have been involved in several Salesforce projects and written technical documentation. But I rarely share my knowledge and experiences with others on public platforms like blogs using English. I'm excited to take on this challenge and use English to connect with a broader audience while exploring various aspects of Salesforce and its applications.

---

Salesforce is a customizable platform that is ideal for businesses seeking to optimize their operations. However, with increasing customizations and features added to the Salesforce environment, it becomes difficult to maintain efficient and scalable code. To design high-performing code in Salesforce, here are some tips to consider.

### Clear and Concise Code

It is crucial to maintain clear and concise code when writing it. This makes the code easier to understand and maintain. To achieve this, use appropriate naming conventions, reduce code redundancy, and add proper comments. If the code structure is clear and easy to understand, it will be easier to extend the code in the future.

#### Use descriptive variable and function names

For example, instead of using a variable name like `temp`, use `accountName` if it holds the name of an Account.

```java
// Bad
String temp;

// Good
String accountName;
```

#### Break down complex code into smaller, reusable functions

For example, consider the following code that calculates the sum of a list of integers:

```java
// Bad
public Integer calculateSum(List<Integer> numbers) {
    Integer sum = 0;
    for (Integer number : numbers) {
        sum += number;
    }
    return sum;
}

// Good
public Integer calculateSum(List<Integer> numbers) {
    return sumNumbers(numbers);
}

private Integer sumNumbers(List<Integer> numbers) {
    Integer sum = 0;
    for (Integer number : numbers) {
        sum += number;
    }
    return sum;
}
```

#### Document your code

For example, use comments to explain what the code does and how it works:

```java
// Calculate the average of a list of integers
// Input: List of integers
// Output: Average as a decimal number
public Decimal calculateAverage(List<Integer> numbers) {
    Integer sum = calculateSum(numbers);
    Decimal average = (Decimal) sum / numbers.size();
    return average;
}
```

Check out a more specialized example:

```java
/************************************************************************
* @Description  Calculate the average of a list of integers
* @Param		List of integers - numbers
* @Return   Decimal - Average as a decimal number.
* @Example     
* ApexClass.calculateAverage(new List<Integer>{1, 3, 5})
*************************************************************************/ 
public Decimal calculateAverage(List<Integer> numbers) {
    Integer sum = calculateSum(numbers);
    Decimal average = (Decimal) sum / numbers.size();
    return average;
}
```

#### Avoid duplicate code

For example, instead of repeating the same code in multiple places, create a function that can be called from multiple places:

```java
// Bad
public void updateContactPhoneNumber(Contact contact, String phoneNumber) {
    contact.Phone = phoneNumber;
    update(contact);
}

public void updateLeadPhoneNumber(Lead lead, String phoneNumber) {
    lead.Phone = phoneNumber;
    update(lead);
}

// Good
public void updateRecordPhoneNumber(SObject record, String fieldName, String phoneNumber) {
    record.put(fieldName, phoneNumber);
    update(record);
}
```

### Utilize Apex Reuse

Apex allows you to reuse code to increase efficiency and maintainability. You can use code blocks such as utility, design patterns and trigger handlers to reduce the amount of code and improve code readability. When using reusable code, ensure that the code has been properly tested and that its behavior meets your expectations.

#### Creating utility classes

Create a utility class to hold commonly used methods that can be shared across multiple classes. For example:

```java
public class StringUtils {
    public static Boolean isEmpty(String str) {
        return str == null || str.trim().length() == 0;
    }

    public static Boolean isNotEmpty(String str) {
        return !isEmpty(str);
    }
}
```

#### Using Apex design patterns

Apex design patterns provide a standard way to solve common problems, making it easier to reuse code. For example, the Singleton pattern can be used to ensure that only one instance of a class is created in an application:

```java
public class Config {
    private static Config instance;
    private Map<String, String> configValues;

    private Config() {
        // Load config values from a custom setting
        configValues = ...;
    }

    public static Config getInstance() {
        if (instance == null) {
            instance = new Config();
        }
        return instance;
    }

    public String getConfigValue(String key) {
        return configValues.get(key);
    }

    public void setConfigValue(String key, String value) {
        // persist the updated configuration value to a custom setting or a custom object
        configValues.put(key, value);
    }
}
```

#### Reusing Apex components

Imagine you have a custom object called "Invoice" and you need to create a method that calculates the total amount of an invoice. You want this method to be reusable, so you can use it in multiple places in your application.

```java
public with sharing class InvoiceHelper {

    public static Decimal calculateTotalAmount(Invoice__c invoice) {
        Decimal totalAmount = 0;

        for (InvoiceLineItem__c lineItem : invoice.InvoiceLineItems__r) {
            totalAmount += lineItem.Quantity__c * lineItem.Price__c;
        }

        return totalAmount;
    }
}
```

Now, you can call this method in multiple places in your application by simply calling `InvoiceHelper.calculateTotalAmount(invoice)`. This way, you can reuse the same calculation logic without having to write the same code multiple times.

### Avoid Hardcoding

To make code more flexible and scalable, you should use Salesforce configuration features instead of hardcoding values in the code. For example, you can store some parameters in Custom Settings and then use these parameters in the code. This way, if you need to change these parameters, you can just change them in the Custom Settings, and the code does not need to be modified.

- **Custom settings:** Use custom settings to store configuration data that can be easily updated without modifying the code.
- **Custom labels:** Use custom labels to store text strings that can be easily translated into multiple languages.
- **Custom metadata types:** Use custom metadata types to store configuration data that can be easily updated without modifying the code.
- **Constants:** Use constants to store values that don't change, such as field names, picklist values, or URLs.
- **Use of dynamic Apex:** Dynamic Apex allows you to dynamically query, update, and delete data at runtime. This can help you to write more flexible and maintainable code.

### Make use of caching

Caching allows frequently used data to be stored in memory, reducing the number of database queries required and improving performance. Salesforce provides a caching mechanism that can be leveraged to store data in memory, so consider using it for frequently accessed data.

The `Cache` class provides a `cache` method that allows you to store a value in cache and retrieve it later. The method takes two arguments: a key, which is a string that represents the cached data, and a value, which is the data to be cached. Here's an example:

```java
Cache.CacheValueWrapper cachedValue = Cache.get('myKey');

if (cachedValue == null) {
    // The data is not in cache, query the database
    List<Account> accounts = [SELECT Id, Name FROM Account];

    // Store the data in cache
    Cache.put('myKey', accounts, 300);

    // Use the data from cache
    cachedValue = Cache.get('myKey');
}

List<Account> accounts = (List<Account>) cachedValue.getValue();
```

In this example, the data is first retrieved from cache using the key `myKey`. If the data is not in cache, it is queried from the database and then stored in cache using the same key and a timeout of 300 seconds.

and then, you can use it to store and retrieve data in cache. Here's an example:

```java
public class AccountService {
    private static Map<String, List<Account>> cache = new Map<String, List<Account>>();

    public static List<Account> getAccounts() {
        List<Account> accounts = cache.get('accounts');

        if (accounts == null) {
            // The data is not in cache, query the database
            accounts = [SELECT Id, Name FROM Account];

            // Store the data in cache
            cache.put('accounts', accounts);
        }

        return accounts;
    }
}
```

You can also check out the official [trailheadapps repos](https://github.com/trailheadapps/apex-recipes) to learn more about cache:

### Use Apex best practices

Apex is the proprietary programming language used in Salesforce, and it's essential to follow its best practices to ensure efficient and scalable code. For example, use bulk methods when processing large amounts of data, avoid SOQL (Salesforce Object Query Language) and DML (Data Manipulation Language) statements within loops, and properly handle exceptions to minimize the risk of error. For more information on these and other best practices, see [Apex Best Practices: The 15 Apex Commandments](https://developer.salesforce.com/blogs/developer-relations/2015/01/apex-best-practices-15-apex-commandments). If you are using Visual Studio Code as your IDE, consider using the [Apex PMD extension](https://marketplace.visualstudio.com/items?itemName=chuckjonas.apex-pmd) to automatically check your code for most best practices.

### Use Asynchronous Processing

When handling a large amount of data, use asynchronous processing to avoid slowing down the performance of your Salesforce instance. You can use Apex batch jobs, queueable Apex, and future methods to process data asynchronously. This improves the user experience and ensures that your code remains scalable and efficient.

### Summary

By following these best practices, you can ensure that your code in Salesforce is efficient, scalable, and can handle the demands of your business. Whether you're building customizations, automating processes, or integrating with other systems, taking the time to design your code properly will pay off in the long run.