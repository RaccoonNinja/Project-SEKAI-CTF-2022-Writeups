## Skills involved: PHP __wakup bypass, messing with PHP in general

This challenge proves I'm not familiar with PHP despite having worked with it for some time. If I persist on double checking my gathered info and try different things I had, I could have solved it quickly.

## Solution:

Upon visiting the site, we are greeted with the PHP source code (version 7.4.5 from header).

```php
<?php
include('./flag.php');
class Sekai_Game{
    public $start = True;
    public function __destruct(){
        if($this->start === True){
            echo "Sekai Game Start Here is your flag ".getenv('FLAG');
        }
    }
    public function __wakeup(){
        $this->start=False;
    }
}
if(isset($_GET['sekai_game.run'])){
    unserialize($_GET['sekai_game.run']);
}else{
    highlight_file(__FILE__);
}

?> 
```

From it we know that we have to bypass the \_\_wakeup magic method, which is invoked for deserializing in normal situations - but there are a lot of abnormal situations especially in php:
- https://bugs.php.net/bug.php?id=72663
- https://github.com/php/php-src/issues/9618
- https://github.com/php/php-src/issues/8938
- https://bugs.php.net/bug.php?id=81151
- https://inhann.top/2022/05/17/bypass_wakeup/ (in simplified chinese; while not technically a bypass it's creative)

![image](https://user-images.githubusercontent.com/114584910/193476421-fc928a00-8c4b-497a-aa3d-0c09cb9c48ec.png)

Nonetheless when we try the payload, we are greeted by the same highlighted source code again. This means that the query *key* isn't working as otherwise we would have got a blank page.

There can be 2 possibilities:
- nginx rewrite rules
- PHP somehow rewrites the query key

And since it's likely hard to learn about the nginx rewrite rules, we can research on `php query period bypass` and get https://www.secjuice.com/abusing-php-query-string-parser-bypass-ids-ips-waf/

The 2 halves are then combined to give the flag.

P.S. I was actually blind and only tried `C:10:"Sekai_Game":1:{s:5:"start";b:1;}` without changing the body, along with other attempts using other resources.
