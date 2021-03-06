yii2-image-uploader
===================

Yii2 behavior for upload image to model

Installation
------------
Add to composer.json in your project
```json
{
	"require":
	{
  		"demi/image": "dev-master"
	}
}
```
then run command
```code
php composer.phar update
```

Configuration
-------------
In model file add image uploader behavior:
```php
public function behaviors()
{
    return [
        'imageUploaderBehavior' => [
            'class' => \demi\image\ImageUploaderBehavior::className(),
            'imageConfig' => [
                'imageAttribute' => 'image',
                'savePathAlias' => '@frontend/web/images/products',
                'rootPathAlias' => '@frontend/web',
                'noImageBaseName' => 'noimage.png',
                'imageSizes' => [
                    '' => 1000,
                    'medium_' => 270,
                    'small_' => 70,
                    'my_custom_size' => 25,
                ],
            ],
        ],
    ];
}
```

Usage
-----
In any view file:
```php
// big(mainly) image
Html::img($model->getImageSrc());
// or shortly
Html::img($model->imageSrc);

// medium image
Html::img($model->getImageSrc('medium_'));

// small image
Html::img($model->getImageSrc('small_'));

// image size "my_custom_size"
Html::img($model->getImageSrc('my_custom_size'));
```

Bonus: Delete Image Action
-----
Add this code to you controller:
```php
public function actions()
{
    return [
        'deleteImage' => [
            'class' => 'demi\image\DeleteImageAction',
            'modelClass' => Post::className(),
            'canDelete' => function ($model) {
                    /* @var $model \yii\db\ActiveRecord */
                    return $model->user_id == Yii::$app->user->id;
                },
            'redirectUrl' => function ($model) {
                    /* @var $model \yii\db\ActiveRecord */
                    // triggered on !Yii::$app->request->isAjax, else will be returned JSON: {status: "success"}
                    return ['post/view', 'id' => $model->primaryKey];
                },
            'afterDelete' => function ($model) {
                    /* @var $model \yii\db\ActiveRecord */
                    // You can customize response by this function, e.g. change response:
                    if (Yii::$app->request->isAjax) {
                        Yii::$app->response->getHeaders()->set('Vary', 'Accept');
                        Yii::$app->response = Response::FORMAT_JSON;

                        return ['status' => 'success', 'message' => 'Image deleted'];
                    } else {
                        return Yii::$app->response->redirect(['post/view', 'id' => $model->primaryKey]);
                    }
                },
        ],
    ];
}
```
In you view file: