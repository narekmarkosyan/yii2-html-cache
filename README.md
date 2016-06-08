Yii2 HTML cache
===============

Yii2 HTML Cache generates cache file with FULL HTML of your action from `DOCTYPE` to `</html>`.


# Instalation

Add following to your `components` in configs
 
```php
    'htmlcache' => [
        'class' => '\narekmarkosyan\htmlcache\HtmlCache',
        'lifeTime' => 60*60*24, // 1 day in seconds
        'extra_params' => [],
        'disabled' => false,
        'excluded_actions' => [],
        'excluded_params' => [],
    ],
```

# Usage

Add following code at the beginning of `beforeAction` method in your controller

```php
Yii::app()->htmlcache->loadFromCache($this, $action);
```

Where `$action` is a first parameter of `beforeAction`. Or if you don't have `beforeAction` add the following code to your controller

```php
/**
 * @param CInlineAction $action
 * @return bool
 */
protected function beforeAction($action)
{
    Yii::app()->htmlcache->loadFromCache($this, $action);
 
    return parent::beforeAction($action);
}
```

Add the following code at the end of `renderContent` method in your controller

```php
$output = Yii::app()->htmlcache->saveToCache($this, $this->action, $output);
```

Where `$output` is a first parameter of `renderContent`. Or if you don't have `renderContent` add the following code to your controller

```php
/**
  * @param string $output
  * @return string
 */
 public function renderContent($content){
    $content = parent::renderContent($content);
    
    $content  = Yii::$app->htmlcache->saveToCache($this, $this->action, $content);

    return $content;
}
```


# Settings

* **lifeTime** - lifetime of generated cache in seconds. _Default: 1 day_
* **extra_params** - parameters in controller that can affect your final HTML. For example, if you have an action with product description and it variates for different `id_product`, add `extra_params => array('id_product')` to configs
* **disabled** - `true` if disabled, `false` if enabled. _Default: false_
* **excluded_actions** - Actions list that doesn't need to be cached in `array('controller_id'=> array('action1', 'action2'))` format.
* **excluded_params** - Params list that doesn't need to be cached if they exist and are equal to exact value. If value not set then checking should not be a false.
 
> NOTE 1: in `excluded_params` you need to store Controller variables, not `$_GET/$_POST/$_REQUEST` variables

> NOTE 2: you can add excluded action or parameter from controller by calling `excludeActions` and `excludeParams` methods. See **Additional features** section

# Additional features

There are few additional methods you can use in your controller.

## directReplace

If you have some parts in your HTML that need to be loaded dynamically in cache you can use placeholders in view and then replace them with `directReplace` method.

### Params

There are two ways for parameters

* string `$replace_key` - key that needs to be replaced
* string `$value` - HTML part that needs to be placed instead of placeholder

**OR**

* array `$replace_array` - array with `$replace_key => $value` 

### Usage

In view

```HTML
<div>{DATE_PLACEHOLDER}</div>
```

In `beforeAction` method of your controller

```PHP
    Yii::app()->htmlcache->directReplace("DATE_PLACEHOLDER", date("Y-m-d H:i:s"));
    
    // OR
    
    Yii::app()->htmlcache->directReplace(array("DATE_PLACEHOLDER"=> date("Y-m-d H:i:s")));
```

> NOTE: placeholder always needs to be UPPERCASE and in {BRACES}

## excludeActions

Adding action to the list of excluded actions.
 
### Params

* CController  `$controller` - Controller of actions
* string|array `$actions` - name of action or list of actions

### Usage

In `beforeAction` method of your controller

```PHP
    Yii::app()->htmlcache->excludeActions($this, "my_action");
    
    // OR
    
    Yii::app()->htmlcache->excludeActions($this, array("my_action", "my_other_action"));
```

## allowActions

Removes action from excluded actions list.
 
### Params

* CController  `$controller` - Controller of actions
* string|array `$actions` - name of action or list of actions

### Usage

In `beforeAction` method of your controller

```PHP
    Yii::app()->htmlcache->allowActions($this, "my_action");
    
    // OR
    
    Yii::app()->htmlcache->allowActions($this, array("my_action", "my_other_action"));
    
    // OR
    
    Yii::app()->htmlcache->allowActions($this, null); // Removes all actions of this controller
```

## excludeParams

Adding params to the list of excluded params.
 
### Params

* CController  `$controller` - Controller of actions
* string|array `$params` - name of param or list of params

### Usage

In `beforeAction` method of your controller

```PHP
    Yii::app()->htmlcache->excludeParams($this, "my_action"); // Disable cache if $this->my_action != false
    
    // OR
    
    Yii::app()->htmlcache->excludeParams($this, array("my_action" => 1)); // Disable cache if $this->my_action == 1
    
    // OR
    
    Yii::app()->htmlcache->excludeParams($this, array("my_action" => array(1, 2))); // Disable cache if $this->my_action == 1 OR $this->my_action == 2
```

## allowParams

Removes params from the list of excluded params
 
### Params

* CController  `$controller` - Controller of actions
* string|array `$params` - name of param or list of params

### Usage

In `beforeAction` method of your controller

```PHP
    Yii::app()->htmlcache->allowParams($this, "my_action"); // Removes all mentions of my_action of this controller if exsists
    
    // OR
    
    Yii::app()->htmlcache->allowParams($this, array("my_action" => 1)); // Removes my_action == 1 mention from this controller if exsists
    
    // OR
    
    Yii::app()->htmlcache->allowParams($this, array("my_action" => array(1, 2))); // Removes my_action == 1 OR my_action == 1 mentions from this controller if exsists
    
    // OR
    
    Yii::app()->htmlcache->allowParams($this, null); // Removes ALL excluded params of this controller
```

# FAQ

> **I'm doing everythink like described but it doesn't work.**

hmm...