RBAC Manager for Yii 2
======================

文档
-------------
> **Important: If you install version 3.x, please see [this readme](https://github.com/mdmsoft/yii2-admin/blob/3.master/README.md#upgrade-from-2x).**


- [Change Log](CHANGELOG.md).
- [Authorization Guide](http://www.yiiframework.com/doc-2.0/guide-security-authorization.html). 这个重要，最好先看一下.
- [Basic Usage](docs/guide/basic-usage.md).
- [Using Menu](docs/guide/using-menu.md).
- [Api](http://mdmsoft.github.io/yii2-admin/index.html).

安装
------------

### 使用Composer安装

建议使用[composer](http://getcomposer.org/download/)安装.

如下

```
php composer.phar require mdmsoft/yii2-admin "~1.0"
或
php composer.phar require mdmsoft/yii2-admin "~2.0"
```
在win7下安装，如
```
项目根路径>composer require mdmsoft/yii2-admin "~2.0"
如: D:\1_web\2015_yii2_study>composer require mdmsoft/yii2-admin "~2.0"
```

或安装dev-master版本

```
php composer.phar require mdmsoft/yii2-admin "2.x-dev"
```

或者，将下面加到

```
"mdmsoft/yii2-admin": "~2.0"
```

`composer.json` 文件的require部分并执行 `php composer.phar update`.

### 使用压缩包安装

从[releases](https://github.com/mdmsoft/yii2-admin/releases)下载最新版本,解压缩到你的项目中.
在应用配置中，添加扩展的路径别名.

```php
return [
    ...
    'aliases' => [
        '@mdm/admin' => 'path/to/your/extracted',
        // for example: '@mdm/admin' => '@app/extensions/mdm/yii2-admin-2.0.0',
        ...
    ]
];
```

用法
-----

当扩展安装好后，只需按以下简单修改应用配置如下:

```php
return [
    'modules' => [
        'admin' => [
            'class' => 'mdm\admin\Module',
            ...
        ]
        ...
    ],
    ...
    'components' => [
        ...
        'authManager' => [
            'class' => 'yii\rbac\PhpManager', // or use 'yii\rbac\DbManager'
        ]
    ],
    'as access' => [
        'class' => 'mdm\admin\components\AccessControl',
        'allowActions' => [
            'site/*',
            'admin/*',
            'some-controller/some-action',
            // The actions listed here will be allowed to everyone including guests.
            // So, 'admin/*' should not appear here in the production, of course.
            // But in the earlier stages of your development, you may probably want to
            // add a lot of actions here until you finally completed setting up rbac,
            // otherwise you may not even take a first step.
        ]
    ],
];
```
详细请看 [Yii RBAC](http://www.yiiframework.com/doc-2.0/guide-security-authorization.html#role-based-access-control-rbac) .
可以通过下面的url访问Auth manager:

```
http://localhost/path/to/index.php?r=admin
http://localhost/path/to/index.php?r=admin/route
http://localhost/path/to/index.php?r=admin/permission
http://localhost/path/to/index.php?r=admin/menu
http://localhost/path/to/index.php?r=admin/role
http://localhost/path/to/index.php?r=admin/assignment
```

如果使用菜单管理器(可选)，执行数据迁移:
```
yii migrate --migrationPath=@mdm/admin/migrations
```

如果你使用数据库(class 'yii\rbac\DbManager') 保存 rbac数据, 执行数据迁移:
```
yii migrate --migrationPath=@yii/rbac/migrations
```

自定Assignment控制器
---------------------------------

Assignment controller properties may need to be adjusted to the User model of your app.
To do that, change them via `controllerMap` property. For example:

```php
    'modules' => [
        'admin' => [
            ...
            'controllerMap' => [
                 'assignment' => [
                    'class' => 'mdm\admin\controllers\AssignmentController',
                    /* 'userClassName' => 'app\models\User', */
                    'idField' => 'user_id',
                    'usernameField' => 'username',
                    'fullnameField' => 'profile.full_name',
                    'extraColumns' => [
                        [
                            'attribute' => 'full_name',
                            'label' => 'Full Name',
                            'value' => function($model, $key, $index, $column) {
                                return $model->profile->full_name;
                            },
                        ],
                        [
                            'attribute' => 'dept_name',
                            'label' => 'Department',
                            'value' => function($model, $key, $index, $column) {
                                return $model->profile->dept->name;
                            },
                        ],
                        [
                            'attribute' => 'post_name',
                            'label' => 'Post',
                            'value' => function($model, $key, $index, $column) {
                                return $model->profile->post->name;
                            },
                        ],
                    ],
                    'searchClass' => 'app\models\UserSearch'
                ],
            ],
            ...
        ]
        ...
    ],

```

- 必需的属性
    - **userClassName** Fully qualified class name of your User model  
        Usually you don't need to specify it explicitly, since the module will detect it automatically
    - **idField** ID field of your User model  
        The field that corresponds to Yii::$app->user->id.  
        The default value is 'id'.
    - **usernameField** User name field of your User model  
        The default value is 'username'.
- 可选属性
    - **fullnameField** The field that specifies the full name of the user used in "view" page.  
        This can either be a field of the user model or of a related model (e.g. profile model).  
        When the field is of a related model, the name should be specified with a dot-separated notation (e.g. 'profile.full_name').  
        The default value is null.
    - **extraColumns** The definition of the extra columns used in the "index" page  
        This should be an array of the definitions of the grid view columns.  
        You may include the attributes of the related models as you see in the example above.  
        The default value is an empty array.
    - **searchClass** Fully qualified class name of your model for searching the user model  
        You have to supply the proper search model in order to enable the filtering and the sorting by the extra columns.  
        The default value is null.


自定义布局
------------------

默认, admin模块使用的是应用级的布局文件.
In that case you have to write the menu for this module on your own.

By specifying the `layout` property, you can use one of the built-in layouts of the module:
'left-menu', 'right-menu' or 'top-menu', all equipped with the menu for this module.

```php
    'modules' => [
        'admin' => [
            ...
            'layout' => 'left-menu', // defaults to null, using the application's layout without the menu
                                     // other avaliable values are 'right-menu' and 'top-menu'
        ],
        ...
    ],
```

If you use one of them, you can also customize the menu. You can change menu label or disable it.

```php
    'modules' => [
        'admin' => [
            ...
            'layout' => 'left-menu',
            'menus' => [
                'assignment' => [
                    'label' => 'Grant Access' // change label
                ],
                'route' => null, // disable menu
            ],
        ],
        ...
    ],
```

While using a dedicated layout of the module, you may still want to have it wrapped in your application's main layout
that has your application's nav bar and your brand logo on it.
You can do it by specifying the `mainLayout` property with the application's main layout. For example:

```php
    'modules' => [
        'admin' => [
            ...
            'layout' => 'left-menu',
            'mainLayout' => '@app/views/layouts/main.php',
            ...
        ],
        ...
    ],
```

[截屏](https://picasaweb.google.com/105012704576561549351/Yii2Admin?authuser=0&feat=directlink)
