---
layout: post
title: "Apex Design Patterns - 策略模式"
subtitle: ""
date: 2021-11-28 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-design-patterns.png"
catalog: true
hidden: true
tags:
  - Salesforce
  - Apex
  - Design Patterns
---

> [Design Pattern] is a solution to a problem in a context.

### 策略模式 (Strategy pattern)

**什么是策略模式？**: 是一种行为设计模式，它能让你定义一系列算法，并将每种算法分别放入独立的类中，以使算法的对象能够相互替换

策略模式使得算法可以在不影响到客户端的情况下发生变化。其主要目的是通过定义相似的算法，替换 if else 语句写法，并且可以随时相互替换

**为什么要用策略模式？**: 有时，如果在同一个类中包含了太多代码行为，代码可能会变得复杂，新开发人员很难修复它。此外，如果团队规模很大，多个开发人员在使用混合代码的单个类上工作也可能会很繁琐。因此，为了解决这个问题，我们创建了包含特定类型行为而不是混合行为的单独类，并在运行时调用它们使用策略

问题：您需要提供一种基于地理的搜索引擎解决方案，其中执行代码可以在运行时选择搜索提供者。

```java
geocode('test address')
// => 37.78,-129.39
```

解决方案：

```java
public interface GeocodeStrategy{
  List<Double> getLatLong(String address);
}
```

```java
public class GoogleMapsStrategyImpl implements GeocodeStrategy{
    public List<Double> getLatLong(String address){
      // Web service callout
      return new List<Double>{0,0};
    }
}
```

```java
public class BaiduMapsStrategyImpl implements GeocodeStrategy{
    public List<Double> getLatLong(String address){
        // Web service callout
        return new List<Double>{1,1};
    }
}

```

```java
public class Geocoder {
  public class GeocoderException extends Exception{}
  
  public static final Map<String,GeocodeStrategy> strategies;
  
  static{ 
      // 策略名称列表的集合 比如：GoogleMapsStrategy, BaiduMapsStrategy 等
      List<String> strategyNames = Label.Strategies.split(',');
  
      strategies = new Map<String,GeocodeStrategy>();
      for(String name : strategyNames){
          try{
              strategies.put(name, (GeocodeStrategy)Type.forName(name + 'Impl').newInstance());
          } catch(Exception e){ 
              system.debug(e.getMessage());
          } 
      }
  }
  
  private GeocodeStrategy strategy;
  
  public Geocoder(String name){
      if(!strategies.containsKey(name)) {
        throw new GeocoderException(name);
      }
      strategy = strategies.get(name);
  }
  
  public List<Double> getLatLong(String address){
      return strategy.getLatLong(address);
  }
}
```

```java
Geocoder geocoder = new Geocoder('GoogleMapsStrategy'); 
System.debug('latitudes***' + geocoder.getLatLong('test location1'));
```

```java
Geocoder geocoder = new Geocoder('BaiduMapsStrategy'); 
System.debug('latitudes***' + geocoder.getLatLong('test location1'));
```

策略设计模式使得行为和使用该行为的类之间可以更好地解耦。这允许在不破坏使用该行为的类的情况下改变该行为，并且这些类可以通过改变所使用的具体实现而在不同的行为之间进行切换，而不需要对代码进行任何重大修改。


