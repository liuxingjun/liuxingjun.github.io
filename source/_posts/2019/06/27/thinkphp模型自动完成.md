---
title: thinkphp模型自动完成
date: 2019-06-27 11:15:24
categories: php
tags:
  - thinkphp
---
创建时间 `create_time` 使用的 `timestamp` 类型 
`额外参数` 仅在 `附加规则` 为 `function` 或 `callback` 时有效
thinkphp 文档内没有说明如何使用带参函数或者带参回调
```
namespace Home\Model;
class UserModel extends Model {
    protected $_auto = array (
        //array('field','填充内容','填充条件','附加规则',[额外参数])
        array('status','1'),
        array('password','md5',self::MODEL_BOTH,'function') ,
        array('name','getName',self::MODEL_BOTH,'callback'),
        array('create_time','date',self::MODEL_INSERT,'function','Y-m-d H:i:s')
    );
}
```

