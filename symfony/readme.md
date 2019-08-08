# 190726
* composer require symfony/maker-bundle --dev して　 php bin/console list make すると　There are no commands defined in the "make" namespace.  となる。https://symfony.com/doc/current/bundles/SymfonyMakerBundle/index.html
* render()が使えない。Undefined property: AppBundle\Controller\DefaultController::$render" 

# 190802
* 同様に generate:doctrine:entity も動かなかったが、mysql.server start で解決。my.cnf に default_authentication_plugin= mysql_native_password しないとダメだの、brew の mysql と mysql@5.6 が競合してるから関連ファイルを削除しないとダメだのは関係なかった。しかし、mysql@5.6 では PIDファイルがないって言われてmysql.server start ができないので詰んでる。
* 

# 190806
* https://symfony-japan.github.io/blog-tutorial
* mysql.server start でサーバーを起動していないとアクセスできない。ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
が出る。停止はmysql.server stop
* form_widgetでフォームがレンダリングされる。
* form_theme は bootstrap系以外は同じようなものの気がする
* 以下のように、複数のフォームも適用できる。この場合、indexが若い者と重複する部分はoverrideされる。
```
{% form_theme form with [
    'foundation_5_layout.html.twig',
    'forms/my_custom_theme.html.twig'
] %}
  ```
* nl2brは{{some_content|nl2br}}などとした時に、some_content内の¥nをbrタグにに変えてくれる。