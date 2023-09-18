### Полезное по Yii
[Сборник советов для Yii1Сборник советов для Yii1](http://docs.mirocow.com/doku.php?id=yii:tips)

### Формат для множественных форм
[Источник](https://www.yiiframework.com/doc/guide/1.1/ru/topics.i18n)

```php
echo Yii::t('app', 'товар|товара|товаров|товара', 1);
```

Порядок выражений:
1. 1, 21, 31, 41, 51, 61, 71, 81, 101...
2. 2\~4, 22\~24, 32\~34, 42\~44, 52\~54, 62, 102...
3. 0, 5\~19, 100, 1000, 10000, 100000...
4. 0.0\~1.5, 10.0, 100.0...

### jQuery Yii GridView plugin file
Методы
* $.fn.yiiGridView.getChecked
* $.fn.yiiGridView.getSelection
* $.fn.yiiGridView.update
* $.fn.yiiGridView.getColumn
* $.fn.yiiGridView.getRow
* $.fn.yiiGridView.getUrl
* $.fn.yiiGridView.getKey

Файл-источник для понимания работы функций: `protected/vendor/yiisoft/yii/framework/zii/widgets/assets/gridview/jquery.yiigridview.js`

#### Примеры использования
##### Получить значения выбранных чекбоксов
```php
$this->widget('zii.widgets.grid.CGridView', [
    'id'           => 'my-grid-id',
    'dataProvider' => $dataProvider,
    'columns'      => [
        [
            'class' => 'CCheckBoxColumn',
            'id'    => 'my-checkbox-id',
        ],
        // other columns
    ],
]);
```
Для получения выбранных значений чекбоксов необходимо использовать `$.fn.yiiGridView.getChecked`
```html
<script>
    // Выбираем отмеченные чекбоксы
    $(document).on('click', '.class-btn', function (e) {
        e.preventDefault();

        // Собираем ID всех отмеченных чекбоксов
        $.fn.yiiGridView.getChecked('my-grid-id', 'my-checkbox-id');
        // Преобразовываем в строку (разделитель — запятая)
        var ids = selectedIds.join(',')
        // Переход на страницу в парметры которой передаём несколько ID
        window.location.href = '<?php echo Yii::app()->createUrl('/admin/', ['ap' => $a, 'action' => 'action',]) ?>/ids/' + ids;
    });
</script>
```


### Удаление строки в CGridView
```php
[
    'class'    => 'CButtonColumn',
    'template' => '{remove}',
    'buttons'  => [
        'remove' => [
            'url'     => '"/admin/log/delete/id/" . $data->id',
            'label'   => '<i class="fas fa-times"></i>',
            'visible' => 'Yii::app()->user->role == "administrator"',
            'options' => [
                'title' => 'Удалить',
                'ajax'  => [
                    'type'       => 'post',
                    'url'        => 'js:$(this).attr("href")',
                    'context'    => 'js:this',
                    'beforeSend' => 'js:function(data) {
                        if(confirm("Вы уверены, что хотите удалить данный элемент?")) $(this).closest("tr").remove();
                        else data.abort();
                    }',
                ],
            ],
        ],
    ],
],
```

1. Вся суть заключается в `beforeSend` - удаление строки происходит ещё до выполнения `ajax`
1. В контролере оставляем только удаление `$this->loadModel($id)->delete();`
4. `$(this).closest("tr").remove();`
`$(this)` - элемент на который нажали
`$(this).closest("tr")` - находим ближайший родительский элемент `tr`
`$(this).closest("tr").remove();` - удаляем элемент `tr`



### Кликабельная строка в CGridView
Нужно добавить параметр `selectionChanged`
Класс `.selected` применяется к строке в момент нажатия на строку. Первый столбец должен содержать ссылку, которая будет применена ко всей строке.
```php
<?php $this->widget('zii.widgets.grid.CGridView', [
    ...
    'selectionChanged' => 'function(){ location.href =($(".selected td a:first").attr("href")); }',
    'columns'          => [
        [
            'name'  => 'ord_id',
            'value' => 'CHtml::link("Заказ № " . $data->ord_id, ["/admin/lnd/' . $lnd . '/order/view/$data->ord_id"])',
            'type'  => 'raw',
        ],
```
Класс .selected применяется к строке в момент нажатия на строку. Первый столбец должен содержать ссылку, которая будет применена ко всей строке.

#### Формирование ссылки в YII
```php
Yii::app()->createUrl(sprintf('/admin/lnd/%s/prodShop/update/id/%s', $lnd, $id))
```

#### Notice с указанием контроллера
```php
Notice::set('ok', 'Успешно!', ['cntr' => 'ProdShop']);
```


#### dropDownList с data атрибутами
```php
foreach ($ftrs as $ft) {
    $res[$ft->pf_id] = $ft->pf_name;
    $opt['options'][$ft->pf_id] = ['data-type' => $ft->pf_type];
}

echo CHtml::dropDownList('ProdFeatureShop[pfs_feature]', $select, $res, $opt);
```


#### Гибкий поиск в фильтре CGridView. Символ % (LIKE)
Для того, чтобы в столбце таблицы можно было использовать гибкий поиск, например: **%BRIDGESTONE%летняя%** необходимо параметр `escape` переключить в `false`

```php
$criteria->compare('ps_name', $this->ps_name, true, 'AND', false);
```


#### Обернуть в span свойство в CGridView
```php
'value' = >function($data) {
    return '<span>' . CHtml::encode($data->column_name) . '</span>';
},
```

#### Валидация ENUM

```php
['yml_price_update', 'in', 'range' => ['0', '15', '30', '60', '180', '360']],
```


#### NOT LIKE в condition
Найти все категории, кроме той у которой `cat_yml_id == 777`
```php
PriceYmlCat::model()->deleteAll([
    'condition' => 'cat_price = :cat_price AND cat_yml_id NOT LIKE :cat_yml_id',
    'params'    => [':cat_price' => $price, ':cat_yml_id' => 777],
]);
```

#### Повторяющиеся одинаковые значения в таблице
```php
SELECT sfo_value, COUNT(*) AS CNT
   FROM cho_src_feature_opt
   GROUP BY sfo_value
   HAVING COUNT(*) > 1;
```

#### Валидация CheckBoxList
Необходимо в модели использовать **beforeValidate**
```php
public function beforeValidate()
{
    parent::beforeValidate();

    $post = Yii::app()->request->getPost('TaskCheck');
    if ( ! empty($post['tc_right_dep'])) $this->tc_right_dep = implode(",", $post['tc_right_dep']);;
    if ( ! empty($post['tc_right_staff'])) $this->tc_right_staff = implode(",", $post['tc_right_staff']);;

    return true;
}
```



#### Почистить ассоциированный массив от дублей

```php
// Исходный массив
$attr = [
    ['attr' => 1, 'value' => 'Яблоко', 'dict' => 17],
    ['attr' => 2, 'value' => 'Груша', 'dict' => 17],
    ['attr' => 3, 'value' => 'Яблоко', 'dict' => 17],
    ['attr' => 4, 'value' => 'Груша', 'dict' => 17],
    ['attr' => 3, 'value' => 'Яблоко', 'dict' => 17],
    ['attr' => 4, 'value' => 'Груша', 'dict' => 17],
];

foreach ($attr as $index_a => $a) {
    foreach ($attr as $index_b => $b) {
        if ($index_a != $index_b && $a['attr'] == $b['attr'] && $a['value'] == $b['value'])
            unset($attr[$ib]);
    }
}

// Итоговый массив
$attr = [
    ['attr' => 1, 'value' => 'Яблоко', 'dict' => 17],
    ['attr' => 2, 'value' => 'Груша', 'dict' => 17],
    ['attr' => 3, 'value' => 'Яблоко', 'dict' => 17],
    ['attr' => 4, 'value' => 'Груша', 'dict' => 17],
];
```



#### Количество записей на странице CGridView
**Модель**
```php
$pageSize = Yii::app()->user->getState('pageSize', Yii::app()->params['defaultPageSize']);
Yii::app()->user->setState('pageSize', $pageSize);

return new CActiveDataProvider($this, [
    'criteria'   => $criteria,
    'pagination' => [
        'pageSize' => $pageSize,
    ],
]);
```

**Контроллер**
```php
if (isset($_GET['pageSize'])) {
    Yii::app()->user->setState('pageSize', (int)$_GET['pageSize']);
    unset($_GET['pageSize']);
}
```

**Вьюшка**
```html
<?php
$pageSize         = Yii::app()->user->getState('pageSize', Yii::app()->params['defaultPageSize']);
$pageSizeDropDown = CHtml::dropDownList(
    'pageSize',
    $pageSize,
    [10 => 10, 50 => 50, 100 => 100, 200 => 200, 300 => 300, 500 => 500],
    [
        'class'    => 'change-pagesize form-select form-select-sm form-select-solid',
        'onchange' => "$.fn.yiiGridView.update('prod-shop-grid',{data:{pageSize:$(this).val()}});",
    ]
);
?>

<div class="row mb-3">
    <div class="col-sm-12 col-md-5 d-flex align-items-center justify-content-center justify-content-md-start">
        <div class="me-2">
            На странице:
        </div>
        <div>
            <label>
                <?=$pageSizeDropDown;?>
            </label>
        </div>
    </div>
</div>
```


#### AJAX-формы, кнопки, ссылки

**Кнопка**

```php
<?=CHtml::htmlButton('<span class="indicator-label">' . ($model->isNewRecord ? 'Создать' : 'Сохранить') . '</span>
        <span class="indicator-progress">Сохранение...
		    <span class="spinner-border spinner-border-sm align-middle ms-2"></span>
        </span>', [
        'type'                  => "button",
        'class'                 => "btn btn-primary ajax-form-btn",
    ]);?>
```

**Ссылка**

####  CGridView filter relation
[Источник](https://www.yiiframework.com/wiki/281/searching-and-sorting-by-related-model-in-cgridview)

1. Добавить в модель переменную
`public $search_cs_name;`
2. Добавить `relation()`
```php
function relations()
{
    return [
        'cat_shop' => [self::BELONGS_TO, 'CategoryShop', 'ps_imp_cat'],
    ];
}
```

3. Добавить атрибут в `rules()`
```php
public function rules()
{
    return [
        ...
        ['xxx, yyy, search_cs_name', 'safe', 'on' => 'search'],
    ];
}
```

4. Добавить атрибут в `search()`
Если в ячейке используется **одно поле**:
```php
$criteria->with = ['cat_shop'];
$criteria->compare('cat_shop.search_cs_name', $this->search_cs_name, true);
```
Если в ячейке используется **несколько полей**:
```php
$criteria->with = ['cat_shop'];
$criteria->compare('cat_shop.search_cs_name', $this->search_cs_name, true);
$criteria->compare('cat_shop.search_cs_offer', $this->search_cs_name, true, 'OR');
```

5. CGridView 
Если в ячейке используется **одно поле**:
```php
$this->widget('zii.widgets.grid.CGridView', [
    'dataProvider' => $model->search(),
    'filter'       => $model,
    'columns'      => [
        'title',
        'post_time',
        [
            'name' => 'author_search',
            'value' => '$data->cat_shop->search_cs_name',
            'filter' => 'Prod[author_search]',
        ],
    ],
]);
```
Если в ячейке используется **несколько полей**:
```php
$this->widget('zii.widgets.grid.CGridView', [
    'dataProvider' => $model->search(),
    'filter'       => $model,
    'columns'      => [
        'title',
        'post_time',
        [
            'name' => 'author_search',
            'value' => '$data->cat_shop->search_cs_name. "<br><span class=\"fw-bold\">Артикул:</span> " . $data->cat_shop->search_cs_offer'
            'filter' => 'ProdShop[search_cs_name]',
        ],
    ],
]);
```


### MySQL / MariaDb
####  Зайти в MariaDb по SSH (ispmanager)
Чтобы получить доступ к контейнеру нужно знать его имя или идентификатор ID. Получить список всех контейнеров можно командой:
```bash
docker ps
```
Столбцы **CONTAINER ID** или **NAMES** помогут нам в идентификации контейнера с которым будем производить манипуляции. Чтобы подключиться к контейнеру:
```bash
docker exec -it CONTAINER_ID bash
```
