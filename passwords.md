git e73c40f0dea4db1205c83584d6c5b544b5ff1683

---

# Сброс пароля 

- [Введение](#introduction)
- [О базе данных](#resetting-database)
- [Роутинг](#resetting-routing)
- [Шаблоны](#resetting-views)
- [После сброса паролей](#after-resetting-passwords)
- [Настройка](#password-customization)

<a name="introduction"></a>
## Введение

> {tip} **Хотите быстро приступить к работе?** Просто запустите `php artisan make:auth` в новом приложении Laravel и перейдите в браузере по адресу `http://your-app.dev/register` или по любому другому URL, который назначен вашему приложению. Эта единственная команда позаботится о создании всей вашей системы аутентификации, включая сброс паролей!

Большинство веб-приложений предоставляют пользователям возможность сбросить забытые пароли. Вместо того, чтобы заставлять вас повторять это в каждом приложении, Laravel предлагает удобные методы для отправки напоминаний о пароле и сброса пароля.

> {note} Чтобы использовать функции сброса пароля Laravel, ваша модель User должна иметь трейт `Illuminate\Notifications\Notifiable`.

<a name="resetting-database"></a>
## О базе данных

Для начала убедитесь, что ваша модель `App\User` реализует контракт `Illuminate\Contracts\Auth\CanResetPassword`. Конечно, модель `App\User` включенная в инфраструктуру, уже реализует этот интерфейс и использует трейт `Illuminate\Auth\Passwords\CanResetPassword`, чтобы включить методы, необходимые для реализации интерфейса.

#### Создание таблицы токенов сброса пароля

Затем необходимо создать таблицу для хранения токенов сброса пароля. Миграция для этой таблицы входит в комплект поставки Laravel и находится в директории `database/migrations`. Итак, все, что вам нужно сделать, это запустить миграцию базы данных:

    php artisan migrate

<a name="resetting-routing"></a>
## Роутинг

Laravel включает классы `Auth\ForgotPasswordController` и `Auth\ResetPasswordController`, которые содержат логику, необходимую для отправки по электронной почте ссылок для сброса пароля и сброса пользовательских паролей. Все роуты, необходимые для выполнения сброса пароля, могут быть сгенерированы командой:

    php artisan make:auth

<a name="resetting-views"></a>
## Шаблоны

Опять же, Laravel сгенерирует все необходимые шаблоны для сброса пароля, когда выполняется команда `make:auth`. Эти шаблоны размещаются в `resources/views/auth/passwords`. Вы можете настроить их по мере необходимости для своего приложения.

<a name="after-resetting-passwords"></a>
## После сброса паролей

После того как вы определили роуты и шаблоны для сброса паролей пользователя, вы можете зайти на эту страницу по урлу `/password/reset`. `ForgotPasswordController` , входящий в состав фреймворка, уже включает в себя логику отправки писем с ссылкой для сброса пароля, в то время как `ResetPasswordController` включает в себя логику сброса пользовательских паролей.

После сброса пароля пользователь автоматически будет зарегистрирован в приложении и перенаправлен на `/home`. Вы можете настроить местоположение переадресации сброса после сброса пароля, указав свойство `redirectTo` в `ResetPasswordController`:

    protected $redirectTo = '/dashboard';

> {note} По-умолчанию токены сброса пароля истекают через один час. Вы можете изменить это с помощью опции сброса пароля `expire` в файле `config/auth.php`.

<a name="password-customization"></a>
## Настройка

#### Настройка гварда аутентификации

В своем конфиге `auth.php` вы можете настроить несколько "гвардов", которые могут использоваться для определения поведения аутентификации для нескольких пользовательских таблиц. Вы можете настроить включенный `ResetPasswordController` для использования гварда по вашему выбору, переопределив метод `guard` на контроллере. Этот метод должен возвращать экземпляр гварда:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Настройка брокера паролей

В конфиге `auth.php` вы можете настроить несколько "брокеров" пароля, которые могут быть использованы для сброса паролей в нескольких пользовательских таблицах. Вы можете настроить включенные функции `ForgotPasswordController` и `ResetPasswordController`, чтобы использовать брокера по вашему выбору, переопределив метод `broker`:

    use Illuminate\Support\Facades\Password;

    /**
     * Получите брокер, который будет использоваться при сбросе пароля.
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### Настройки отправки почты

Вы можете легко изменить класс уведомления, используемый для отправки ссылки сброса пароля пользователю. Для начала переопределите метод `sendPasswordResetNotification` в своей модели `User`. В рамках этого метода вы можете отправить уведомление с использованием любого выбранного вами класса уведомлений. `$token` сброса пароля - первый аргумент, полученный методом:

    /**
     * Отправка уведомления о сбросе пароля.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }

