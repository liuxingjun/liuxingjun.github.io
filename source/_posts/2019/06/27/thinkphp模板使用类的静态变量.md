---
title: thinkphp模板使用类的静态变量
date: 2019-06-27 10:56:24
categories: php
tags: 
  - thinkphp
---

> 变量输出使用的函数可以支持内置的PHP函数或者用户自定义函数，甚至是静态方法。

```
namespace Home\Model;
class UserModel extends Model {
    const STATUS_YES=1;
    const STATUS_NO=2;
    static public $statusArray = array(
        self::STATUS_YES => '是',
        self::STATUS_NO => '否',
    );
    public static function status_string($code){
        return self::$statusArray[$code];
    }
}
```

```
{:\\Home\\Model\\UserModel::status_string(1)}
```