# 本地化

- [简介](#introduction)
- [语言文件](#language-files)
- [基本用例](#basic-usage)
- [复数形式](#pluralization)
- [验证消息的本地化](#validation)

<a name="introduction"></a>
## 简介

Laravel `Lang` 类提供非常方便的方法来从不同语言文件中取得字符串，允许在应用程序中支持多语言。

<a name="language-files"></a>
## 语言文件

语言文字存放在 `app/lang` 目录。应用程序所要支持的语言都需要在此目录建立为子目录。

	/app
		/lang
			/en
				messages.php
			/es
				messages.php

语言文件只是返回键值（字符串）对数组。例如：

**语言文件示例**

	<?php

	return array(
		'welcome' => 'Welcome to our application'
	);

应用程序默认语言配置在 `app/config/app.php`配置文件中 `locale` 配置项.你可以在任何时候使用 `App::setLocale` 方法来改变当前激活语言。

**在运行时改变默认语言**

	App::setLocale('es');

<a name="basic-usage"></a>
## 基本用例

**从语言文件中获取文本**

	echo Lang::get('messages.welcome');

传给 `get` 方法字符串第一部分是语言文件名称，第二个部分是要取得文本的名字。

> **注意**: 如果语言不存在该文本，那么 `get` 方法会将键返回。

**文本中替换**

可以在语言文件中定义占位符：

	'welcome' => 'Welcome, :name',

然后，将要替换的值传递给 `Lang::get` 方法的第二个参数：

	echo Lang::get('messages.welcome', array('name' => 'Dayle'));

**判断语言文件是否存在该文本**

	if (Lang::has('messages.welcome'))
	{
		//
	}

<a name="pluralization"></a>
## 复数形式

复数形式是一个复杂的问题，因为不同的语言有着不同的复数形式规则。你可以通过简单的在语言文件中使用”管道“符来分开单数和复数文本形式：

	'apples' => 'There is one apple|There are many apples',

然后你就可以使用 `Lang::choice` 方法来取得文本：

	echo Lang::choice('messages.apples', 10);

由于Laravel翻译机制是用Symfony的翻译组件，你也可以非常简单的创建更加复杂的复数形式规则：

	'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',


<a name="validation"></a>
## 验证

对于验证功能中需要本地化的错误信息和提示信息，请参阅 <a href="/docs/validation#localization">相关文档</a>。