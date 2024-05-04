---
layout: post
title: Interface Agent
tag: 技术笔记
date: 2021-07-17
category: Technology
---

### 接口动态调用

这几天在写数据插入脚本中学习到了一种接口动态调用的实现方法，当多个接口的功能一致，只是在不同场景下功能体现不同的实现方式时非常实用，能够避免繁琐的控制类管理，统一的控制类，访问路径不同会自动分发请求到不同的逻辑实现中处理，这里简单的总结了一下其实现方式，当然还是以demo的形式来体现了哈。

这里首先需要建立一个spring项目，大家应该已经得心应手便不再赘述

首先定义一个父类接口，这里用Car类举例，在这里Car表示汽车的整个抽象

```java
public interface Car {
    //根据类别获取相应实现
    String getDataType();
    //主要实现的方法
    String introduce();
}
```

接下来我们使用三个组件类去继承Car，分别是SportCar、Truck、Limousine这三个简单的类我们可以理解为汽车类下的三个子种类，为跑车、卡车、豪华轿车，虽然同为汽车，但是它们却有不同的实现方式，我们使用@Component注释，把这些类实例化到spring容器中

```java
@Component
public class SportCar implements Car{
    @Override
    public String getDataType() {
        return "sportCar";
    }

    @Override
    public String introduce() {
        return "this is sport car";
    }
}
```

```java
@Component
public class Truck implements Car{

    @Override
    public String getDataType() {
        return "truck";
    }

    @Override
    public String introduce() {
        return "this is truck";
    }
}
```

```java
@Component
public class Limousine implements Car{

    @Override
    public String getDataType() {
        return "limousine";
    }

    @Override
    public String introduce() {
        return "this is limousine";
    }
}
```

接下来就是比较重要的部分了，首先我们先定义一个接口，CarContext，其定义了一个取得Car实现类的方法，当然我们根据不同的标识会得到不同的Car接口实现，标识便是这里的dataType

```java
public interface CarContext {
    Car getCar(String dataType);
}
```

接着我们定义一个请求分发的逻辑实现类，由于逻辑比较多，所以我将原理以注释的形式写在代码里面，这样更加直观一点

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class DefaultCarDispatch implements ApplicationContextAware, InitializingBean, CarContext {

    //这里定义接收Spring容器，示例化的组件都能在这里找到
    private ApplicationContext applicationContext;
    //将我们得到的不同汽车实例化存储使用的Map
    Map<String, Car> dataCarMap = new HashMap<>();

    //获取spring容器
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    //将实例化的对象存储到dataCarMap当中
    @Override
    public void afterPropertiesSet() throws Exception {
        Map<String, Car> beans = applicationContext.getBeansOfType(Car.class);
        beans.forEach((type, bean) -> {
            dataCarMap.put(bean.getDataType(), bean);
        });
    }

    //根据不同的标识，在Map中查询相应的实例
    @Override
    public Car getCar(String dataType) {
        return dataCarMap.get(dataType);
    }
}
```
接下来就是简单地控制类了，在这里三个不同的服务实现只用到了一个控制类，通过不同的dataType可以得到不同的功能实现

```java
import com.cdw.my.example.demoOne.service.CarContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
public class CarController {

    @Autowired
    private CarContext carContext;

    @GetMapping("/car/{dataType}")
    public String executeDataGenerate(@PathVariable String dataType) {
        return carContext.getCar(dataType).introduce();
    }

}
```
