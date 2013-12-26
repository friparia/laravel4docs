# 安全

- [配置](#configuration)
- [保存密码](#storing-passwords)
- [验证用户](#authenticating-users)
- [手动登陆用户](#manually)
- [保护路由](#protecting-routes)
- [HTTP基本验证](#http-basic-authentication)
- [密码提示和重置](#password-reminders-and-reset)
- [加密](#encryption)

<a name="configuration"></a>
## 配置

Laravel 旨在让验证实现简单。事实上，大部分都已经配置好了。验证配置文件位于 `app/config/auth.php`，该文件包含了一些文档说明非常齐全的配置选项，通过它们可以调整用户验证的具体形式。

一般情况下，Laravel在`app/models`目录下包含有一个`User` 模型，通常被作为默认的Eloquent验证驱动所使用。请注意在构建该数据库结构模型时确保密码字段至少能容纳60个字符。

如果你的应用中不使用 Eloquent，你可以使用 Laravel 查询构造器来使用 `数据库` 验证驱动。

<a name="storing-passwords"></a>
## 保存密码

Laravel `Hash` 类提供了可靠的<a href='http://en.wikipedia.org/wiki/Bcrypt'>Bcrypt</a>散列算法：

**使用Bcrypt散列密码**

	$password = Hash::make('secret');

**验证散列密码**

	if (Hash::check('secret', $hashedPassword))
	{
		// 密码匹配...
	}

**检查密码是否需要重新散列**

	if (Hash::needsRehash($hashed))
	{
		$hashed = Hash::make('secret');
	}

<a name="authenticating-users"></a>
## 用户验证

要在应用程序中登陆用户，使用 `Auth::attempt` 方法。

	if (Auth::attempt(array('email' => $email, 'password' => $password)))
	{
		return Redirect::intended('dashboard');
	}

注意 `email` 不是必需选项，这里仅作为示例。你应该使用任意数据库字段名来作为"用户名"。  `Redirect::intended` 方法会将用户请求重定向至被用户验证过滤器拦截之前用户试图访问URL中去。可以给该方法提供个回退URI，用于在访问目的无效时，重定向到该URI。

在调用 `attempt` 方法时，`auth.attempt` [事件](/docs/events) 将会触发。如果验证成功以及用户登陆了，`auth.login` 事件也会被触发。

要在应用程序中判断用户是否已经登陆，可以使用 `check` 方法：

**判断用户是否已经验证**

	if (Auth::check())
	{
		// 用户已经登陆...
	}
	
如果你想在应用中提供“记住我”功能，你可以传递true作为第二个参数传递给attempt方法，应用程序将会无期限地保持用户验证状态（除非手动退出）：	

**验证用户并且“记住”她们**

	if (Auth::attempt(array('email' => $email, 'password' => $password), true))
	{
		// 用户状态永久保存...
	}

**注意：** 如果 `attempt` 方法返回 `true`， 用户就已经成功登陆应用程序了。

你也可以在用户验证查询中加入其它条件：

**指定其它条件验证用户**

    if (Auth::attempt(array('email' => $email, 'password' => $password, 'active' => 1)))
    {
        // 用户激动状态，没有暂停，并且存在。
    }

一旦用户验证通过了，就可以查看 User 模型/记录：

**查看登陆用户**

	$email = Auth::user()->email;

要简单使用用户ID来登陆应用程序，可以使用 `loginUsingId` 方法：

	Auth::loginUsingId(1);
`validate` 方法可以在不登录应用程序的情况下来验证用户的信息：

**验证用户但不登陆**

	if (Auth::validate($credentials))
	{
		//
	}

也可以使用 `once` 方法将一个用户登录到系统中做一个单次请求。这样的方式不会使用Session或Cookie来进行状态保持。

**登陆用户做单次请求**

	if (Auth::once($credentials))
	{
		//
	}

**应用程序中注销用户登陆状态**

	Auth::logout();

<a name="manually"></a>
## 手动登陆用户

如果想登陆一个存在的用户实例，只需要简单使用`login`方法并传入该实例即可：

	$user = User::find(1);

	Auth::login($user);

这与使用 `attempt` 方法验证用户登陆是一样的。


<a name="protecting-routes"></a>
## 保护路由

路由过滤器可以保证只有通过了验证的用户才能访问指定路由。Laravel 默认提供了 `auth` 过滤器，它定义在 `app/filters.php`文件。

**保护路由**

	Route::get('profile', array('before' => 'auth', function()
	{
		// 只有验证用户可以访问……
	}));

### 跨站请求伪造（CSRF）保护

Laravel 提供便捷的方法来避免应用程序受到跨站伪造请求的攻击。

**表单中插入 CSRF Token**

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

**提供表单时验证 CSRF Token**

    Route::post('register', array('before' => 'csrf', function()
    {
        return 'You gave a valid CSRF token!';
    }));

<a name="http-basic-authentication"></a>
## HTTP 基本验证
HTTP基本验证可以在不设定专门的登录页面的情况下便捷的对应用程序中的用户进行验证。要使用该功能，在路由中附加 `auth.filter` 过滤器：

**HTTP 基本验证来保护路由**

	Route::get('profile', array('before' => 'auth.basic', function()
	{
		// 只有验证用户可以访问……
	}));

默认情况下，`basic` 过滤器验证时会使用用户记录里的 `email` 字段。如果你想使用其他字段，你可以通过传递该字段名字给 `basic` 方法作为第一个参数：

	return Auth::basic('username');

你也可以使用 HTTP 基本验证时不将用户cookie标识写入 session，对 API 验证极其有用。要实现该功能，请定义一个返回 `onceBasic` 方法的过滤器：

**设置无状态的 HTTP 基本过滤器**

	Route::filter('basic.once', function()
	{
		return Auth::onceBasic();
	});

如果你正在使用PHP FastCGI，那么HTTP 基本验证在默认情况下是不会正常工作的。应该将下面的代码加入到`.htaccess`文件中：

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## 密码提示和重置

### 发送密码提示

大多web应用程序提供了让用户重置密码功能。与其让你在每个应用程序中重复去实现该功能，Laravel 提供非常方便的方法来发送密码提示以及执行密码重置。在开始前，请确认你的 `User` 模型实现了 `Illuminate\Auth\Reminders\RemindableInterface` 接口。 当然，框架中的默认的 `User` 模型已经实现了该接口。

**实现 RemindableInterface 接口**

	class User extends Eloquent implements RemindableInterface {

		public function getReminderEmail()
		{
			return $this->email;
		}

	}

接下来，要创建一个表来存储密码重置 token。要为该表生成迁移，简单执行 Artisan 命令 `auth::reminders`：

**为提示表建立迁移**

	php artisan auth:reminders

	php artisan migrate

要发送密码提示，我们可以使用 `Password::remind` 方法：

**发送密码提示**

	Route::post('password/remind', function()
	{
		$credentials = array('email' => Input::get('email'));

		return Password::remind($credentials);
	});

请注意，传递给 `remind` 方法的参数和 `Auth::attempt` 方法中的是相似的。该方法将会找到这个用户并且通过邮箱发送密码重置链接。e-mail视图中会传入一个用来构造重置密码表单链接的 `token` 变量。`user` 对象也会传入该视图中。

> **注意:** 你也可以通过改变 `auth.reminder.email` 配置项来改变邮件消息视图。当然，Laravel已经提供了个默认视图。.

你可以通过传递一个 Closure 作为 `remind` 方法的第二个参数来改变发送给用户的邮件实例：

	return Password::remind($credentials, function($message, $user)
	{
		$message->subject('Your Password Reminder');
	});

你也可能已经注意到我们我们直接从路由中返回 `remind` 方法的结果。默认情况下，`remind` 方法将会返回一个指向当前URI的`Redirect` 重定向. 当在尝试重置密码而出现错误时， 一个 `error` 变量将会 flash 到session中，还有个 `reason` 变量，可以从 `reminders` 语言文件中取得文本。如果密码重置成功，`success` 变量会 flash 到session。所以，你的密码重置表单视图类似如下：

	@if (Session::has('error'))
		{{ trans(Session::get('reason')) }}
	@elseif (Session::has('success'))
		An e-mail with the password reset has been sent.
	@endif

	<input type="text" name="email">
	<input type="submit" value="Send Reminder">

### 重置密码

一旦用户从提示邮件中点击了重置链接，她们将被重定向到一个包含 `token` 隐藏域、一个 `password` 以及 `password_confirmation` 域的表单。以下是一个重置表单路由示例：

	Route::get('password/reset/{token}', function($token)
	{
		return View::make('auth.reset')->with('token', $token);
	});

重置表单类似这样：

	@if (Session::has('error'))
		{{ trans(Session::get('reason')) }}
	@endif

	<input type="hidden" name="token" value="{{ $token }}">
	<input type="text" name="email">
	<input type="password" name="password">
	<input type="password" name="password_confirmation">

再次强调，我们使用 `Session` 来显示任何重置密码时框架检测出的错误。接下来，我们可以定义一个 `POST` 路由来处理重置：

	Route::post('password/reset/{token}', function()
	{
		$credentials = array(
		    'email' => Input::get('email'),
		    'password' => Input::get('password'),
		    'password_confirmation' => Input::get('password_confirmation')
		);

		return Password::reset($credentials, function($user, $password)
		{
			$user->password = Hash::make($password);

			$user->save();

			return Redirect::to('home');
		});
	});

如果密码重置成功，`User` 实例以及密码将会传递到 Closure，允许你实际执行保存操作。然后，你可以在Closure闭包中返回一个`Redirect` 重定向或者任何其他类型的应答。这些应答将会由这个闭包外面的reset方法返回。请注意， `reset` 方法会自动检查请求中 `token` 和 用户信息的有效性，并且匹配密码。

默认情况下，密码重置tokens的有效期是一个小时。你可以通过修改`app/config/auth.php` 文件中的`reminder.expire`选项来更改有效期。

同 `remind` 方法一样， 如果在重置密码时发生错误， `reset` 方法将会 `Redirect` 到当前URI，并带有 `error` 和 `reason` 变量。

<a name="encryption"></a>
## 加密

Laravel通过 mcrypt PHP 的扩展提供了强大的AES-256 加密组件：

**加密**

	$encrypted = Crypt::encrypt('secret');

> **注意:** 确认在 `app/config/app.php` 文件设置了一个32随机字符给 `key` 项。否则，加密的值是不安全的。

**解密**

	$decrypted = Crypt::decrypt($encryptedValue);

你也可以设置在encrypter中使用的 cipher 和 mode：


**设置 Cipher 和 Mode**

	Crypt::setMode('crt');

	Crypt::setCipher($cipher);
