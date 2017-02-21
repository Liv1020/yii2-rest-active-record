Rest Active Record
==================
一个基于restful api模型资源的ActiveRecord方案

安装
------------

安装此扩展的首选方式是通过 [composer](http://getcomposer.org/download/).

任一运行

```
php composer.phar require --prefer-dist pavle/yii2-rest-active-record "*"
```

或者添加

```
"pavle/yii2-rest-active-record": "*"
```

到您的 `composer.json` 文件.


用法
-----

1、继承pavle\yii2\rest\ActiveRecord，添加attributes

```php
/**
 * 注意：一定要写这个来让IDE给你提示
 *
 * @property $fans_id
 * @property $store_id
 * @property $open_id
 * @property $member_name
 * @property $wx_name
 * @property $gender
 * @property $language
 */
class Fans extends ActiveRecord
{
    /**
     * 注意：一定要复写返回可访问的数据
     */
    public function attributes()
    {
        return [
            "fans_id",
            "store_id",
            "open_id",
            "member_name",
            "wx_name",
            "gender",
            "language",
            "city",
            "province",
            "country",
            "is_focus",
            "focus_at",
            "refer_channel",
            "unfocus_at",
            "created_at",
            "updated_at",
            "avatar",
            "deleted_at",
        ];
    }

    public function rules()
    {
        return [
            [$this->attributes(), 'safe']
        ];
    }

    /**
     * 注意：一定要复写返回一个唯一ID
     */
    public static function primaryKey()
    {
        return ['fans_id'];
    }

    /**
     * @return \pavle\yii2\rest\ActiveQuery
     */
    public function getStore()
    {
        return $this->hasOne(Store::className(), ['store_id' => 'store_id']);
    }
}
```

2、配置rest接口

```php
public function actions(){
    return ArrayHelper::merge(parent::actions(), [
        'lists' => [
            'class' => 'pavle\yii2\rest\SearchAction'
        ],
        'create' => [
            'class' => 'yii2\rest\CreateAction'
        ],
        'update' => [
            'class' => 'yii2\rest\UpdateAction'
        ],
        'delete' => [
            'class' => 'yii2\rest\DeleteAction'
        ],
        'count' => [
            'class' => 'pavle\yii2\rest\CountAction'
        ]
    ]);
}
```

3、reset api配置完，在原项目配置config/web.php

```php
    ...
    'components' => [
        'rest' => [
            'class' => 'pavle\yii2\rest\Connection',
            'map' => [
                'pavle\yii2\rest\test\Fans' => [
                    'lists' => 'http://baseapi.xxxxxx.cn/fans/lists',
                    'create' => 'http://baseapi.xxxxxx.cn/fans/create',
                    'update' => 'http://baseapi.xxxxxx.cn/fans/update',
                    'count' => 'http://baseapi.xxxxxx.cn/fans/count',
                    'delete' => 'http://baseapi.xxxxxx.cn/fans/delete',
                ],
                'pavle\yii2\rest\test\Store' => [
                    'lists' => 'http://baseapi.xxxxxx.cn/store/lists',
                    'create' => 'http://baseapi.xxxxxx.cn/store/create',
                    'update' => 'http://baseapi.xxxxxx.cn/store/update',
                    'count' => 'http://baseapi.xxxxxx.cn/store/count',
                    'delete' => 'http://baseapi.xxxxxx.cn/fans/delete',
                ],
                'pavle\yii2\rest\test\Pay' => [
                    'lists' => 'http://baseapi.xxxxxx.cn/pay/lists',
                    'create' => 'http://baseapi.xxxxxx.cn/pay/create',
                    'update' => 'http://baseapi.xxxxxx.cn/pay/update',
                    'count' => 'http://baseapi.xxxxxx.cn/pay/count',
                    'delete' => 'http://baseapi.xxxxxx.cn/fans/delete',
                ],
            ],
            'as rest' => RestResponseBehavior::className() //这里可以使用行为来处理接口数据
        ]
    ],
    ...
```

4、然后就可以像db的ActiveRecord一样调用了

```php
$model = Fans::find()->with('store.pay')->one();
$model->wx_name = 'test';
$model->save();
```
也可以使用ActiveForm、ListView、GridView物件了。

5、行为例子：

```php
class RestResponseBehavior extends Behavior
{
    public function events()
    {
        return [
            Connection::EVENT_CURL_SUCCESS => 'success',
            Connection::EVENT_CURL_ERROR => 'error'
        ];
    }

    /**
     * @param CurlEvent $event
     * @throws BusinessException
     */
    public function success(CurlEvent $event)
    {
        /* @var $curl Curl */
        $curl = $event->curl;
        $result = ArrayHelper::toArray($curl->response);
        if (ArrayHelper::getValue($result, 'code') != 0) {
            throw new BusinessException(ArrayHelper::getValue($result, 'message'));
        }

        $data = ArrayHelper::getValue($curl->response, 'data');
        $curl->response = $data;
    }

    /**
     * @param CurlEvent $event
     * @throws ErrorException
     */
    public function error(CurlEvent $event)
    {
        /* @var $curl Curl */
        $curl = $event->curl;
        \Yii::trace(serialize($curl->response), 'rest.connect');
        throw new ErrorException($curl->errorMessage);
    }
}
```
