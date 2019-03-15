# 开始

首先，请先[安装 Deployer](installation.md)。在命令行中运行如下指令：

```sh
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```

现在你可以使用来`dep`命令来使用 Deployer。

在你的项目目录下打开一个命令行，运行：

```sh
dep init
```

这行指令将在你当前目录下创建`deploy.php`文件。它包含部署时的配置以及任务，被称为*配方 (recipe)*。

默认情况下，所有的配方都继承自 [普通 (common)](https://github.com/deployphp/deployer/blob/master/recipe/common.php)配方。将_deploy.php_文件放在你项目的根目录下，并执行`dep` 或 `dep list` 命令。你将看到一系列可用的*任务 (task)*。

> 你可以在你项下的任何子目录中执行`dep`指令。

定义任务十分简单：

```php
task('test', function () {
    writeln('Hello world');
});
```

想要运行这个任务，执行命令：

```sh
dep test
```

输出：

```text
➤ Executing task test
Hello world
✔ Ok
```

现在，我们来创建一个在远程主机上执行的任务，为此我们必须配置`deployer.php`文件。你创建的`deploy.php`文件中应该包含如下对`主机 (host)`的申明：

```php
host('domain.com')
    ->stage('production')    
    ->set('deploy_path', '/var/www/domain.com');
```

> 你也可以在一个单独的 yaml 文件中申明主机，阅读[inventory](hosts.md#inventory-file)了解更多。

你可以在 [这里](hosts.md)了解到更多关于主机的配置。现在让我创建一个将在远程主机执行`pwd`命令并输出返回结果的任务：

```php
task('pwd', function () {
    $result = run('pwd');
    writeln("Current dir: $result");
});
```

执行`dep pwd`，你将看到如下输出：

```text
➤ Executing task pwd
Current dir: /var/www/domain.com
✔ Ok
```

现在让我们准备我们的第一次部署。你需要先设置`repository`，`shared_files`以及其它参数：

```php
set('repository', 'git@domain.com:username/repository.git');
set('shared_files', [...]);
```

你可以在每个任务中使用`get`函数来返回参数的值。同时也可以覆盖每个主机的设置：

```php
host('domain.com')
    ...
    ->set('shared_files', [...]);
```

了解更多关于部署 [配置](configuration.md) 。

现在让我们来部署你的项目：

```sh
dep deploy
```

若想输出中包含额外的信息，你可以使用`—verbose`选项来增加输出的信息量。

* `-v`  普通输出
* `-vv`  更复杂的输出
* `-vvv`  调试输出

Deployer 将在主机上创建如下目录：

* `releases`  包含已发布的文件夹，
* `shared` 包含共享的文件和文件夹，
* `current` 软链接到当前发布的版本。

配置你的主机使其将`current`当做站点根目录。

> 注意 Deployer 默认使用 [ACL](https://en.wikipedia.org/wiki/Access_control_list) 来设置权限。
>
> 你可以通过修改`writable_mode`设置来改变。

Deployer 默认保留最近的 5 个发布版本，但你也可以通过修改相关的参数来增加：

```php
set('keep_releases', 10);
```

如果部署过程中出现了错误，或者在你新发布的版本中出现了问题，你可以简单地执行下面的指令来回滚到上一个发布版本：

```sh
dep rollback
```

你可能想在某个任务开始/结束时执行其它任务。配置起来十分容易！

比如，让我们在部署完成后重启 php-fpm：

```php
task('reload:php-fpm', function () {
    run('sudo /usr/sbin/service php7-fpm reload');
});

after('deploy', 'reload:php-fpm');
```

如果你想连接到远程主机，Deployer 有一个快速连接的捷径：

~~~sh
dep ssh
~~~

这行指令将会使你连接到选择的主机上，并切换到`current_path`目录下。

了解更多关于部署 [配置](configuration.md) 。