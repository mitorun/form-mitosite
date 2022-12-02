# Компонент формы связи form-mitosite

Универсальная форма с отправкой сообщений на емейл. Используются: библиотека PHPMailer, SMTP, JS, AJAX. Валидация HTML, возможность подключить библиотеку с JS-валидацией, кастомная JS-маска для телефонов, защиты от ботов, сообщения об успехе/ошибке.

Этот компонент основан на форме от MaxGraph: [репозиторий](https://github.com/maxdenaro/maxgraph-youtube-source/tree/master/UI-%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D1%8B%20%E2%84%9613.%20%D0%92%D0%B0%D0%BB%D0%B8%D0%B4%D0%B0%D1%86%D0%B8%D1%8F%20%D0%B8%20%D0%BE%D1%82%D0%BF%D1%80%D0%B0%D0%B2%D0%BA%D0%B0%20%D1%84%D0%BE%D1%80%D0%BC%D1%8B%20%D0%BD%D0%B0%20%D0%BF%D0%BE%D1%87%D1%82%D1%83%20%D0%B2%202022), [ролик](https://youtu.be/6Uq6jV04rGM), [дополнительный ролик](https://drive.google.com/file/d/19XTGYqh3RK6Fuc64dNATsc9hpQjYDFHs/view).

- [Форма-1 без JS (работает на PHPMailer с SMTP).](https://ipbro.ru/form-1.html)
- [Форма-2 базовая с JS.](https://ipbro.ru/form-2.html)
- [Форма-3 навороченная (отправка файлов, маска, вторая форма, валидация).](https://ipbro.ru/form-3.html)
- [Исходные файлы на гитхабе.](https://github.com/mitorun/form-mitosite)
- [https://github.com/PHPMailer/PHPMailer](https://github.com/PHPMailer/PHPMailer)

---

## Что нужно сделать

- Сделать скрипт для работы инпута-ловушки для ботов `<input id="honeypot">`. Чтобы при заполнении этого поля скрипт блокировал отправку формы. Нужно ли показать при этом окно с ошибкой `.form__failure .active`?

- В скрипт обработки поля прикрепления файла от MDN нужно добавить код, чтобы при отправке письма и очистке формы (reset) - в этой форме удалялись все элементы '#file-list li' с перечислением загруженных файлов.

- Если отправить форму с непроставленной галочкой согласия на обработку данных, то страница проскроллится до этого чекбокса, но нет никакого видимого оповещения что тут проблема. Нужно придумать как при этом можно зрительно выделить этот инпут.

- Идея: можно сделать чтобы после отправки формы блок с успехом/неудачей удалялся через некоторое время.

---

## Использование компонента

Создаём HTML формы. Обязательно добавить идентификатор `id="form"` (по нему инициализируется JS) и путь к PHP-конфигу `action="/fls/phpmailer/mail.php"`. Еще скрипт использует классы `.form__success` и `.form__failure` с добавлением к ним `.active` для вывода успеха или ошибки отправки, остальные классы и атрибуты используются только для стилизации и их можно менять.

```html
<form class="form" id="form" action="/fls/phpmailer/mail.php" method="POST" enctype="multipart/form-data">
	<div class="form__item">
		<label class="form__label" for="tel">Ваш телефон</label>
		<input class="form__input" id="tel" type="tel" name="tel" placeholder="+7 123 456 78 90" autocomplete="tel" minlength="6" maxlength="20" title="Включая код страны (+7)">
		<span class="form__validation" aria-hidden="true"></span>
	</div>
	
	<button class="button button--form" type="submit">Кнопка</button>

	<p class="form__success">Сообщение отправлено.</p>
	<p class="form__failure">Ошибка! Сообщение не отправлено.</p>
</form>
```

### Настройка PHP-конфига `mail.php`

```php
$title = "Заголовок письма";
$mail->Username = 'example@gmail.com';// Логин на почте my@example.com
$mail->Password = '******';// Пароль ПРИЛОЖЕНИЯ на почте
$mail->setFrom('example@gmail.com', 'Заявка с сайта');// Адрес и имя отправителя.
$mail->addAddress('example@example.com');// Заполнить емейл(-ы) куда будут приходить письма.
```

## Особенности компонента:
- Форма может работать вообще без JS-скрипта, причём для этого ничего не нужно делать. Атрибут `action="/fls/phpmailer/mail.php"` направляет данные в PHP-скрипт и тот отправляет письмо. Только при этом в браузере открывается страница (белая-пустая) этого PHP-скрипта. Идея для улучшения: доработать PHP-скрипт так, чтобы пользователя возвращало на ту же страницу с формой.

## Текущие проблемы конфига mail.php

- Отправка формы-1 (без JS): письмо нормально пришло, но страница перенаправляется на https://ipbro.ru/fls/phpmailer/mail-1.php и пишет такую ошибку:
```
Warning: Undefined array key "file" in /var/www/u0867248/data/www/ipbro. ru/fls/phpmailer/mail-1.php on line 10
```

Загрузка файлов не работает: открывается страница https://ipbro.ru/fls/phpmailer/mail-exp.php с ошибкой:
```
Fatal error: Uncaught TypeError: count(): Argument #1 ($value) must be of type Countable|array, string given in /var/www/u0867248/data/www/mitorun.studio/fls/phpmailer/mail-exp.php:54 Stack trace: #0 {main} thrown in /var/www/u0867248/data/www/mitorun.studio/fls/phpmailer/mail-exp.php on line 54.
```

Дорабатывать вероятно нужно два участка кода, это "Формирование самого письма":
```php
//$title = "Тема письма";
$file = $_FILES['file'];
$c = true;

// Формирование самого письма:
$title = "Сообщение из Mitorun Studio";
$body = "";
foreach ( $_POST as $key => $value ) {
	if ( $value != "" && $key != "project_name" && $key != "admin_email" && $key != "form_subject" ) {
		$body .= "
		" . ( ($c = !$c) ? '<tr>':'<tr style="background-color: #f8f8f8;">' ) . "
			<td style='padding: 10px; border: #e9e9e9 1px solid;'><b>$key</b></td>
			<td style='padding: 10px; border: #e9e9e9 1px solid;'>$value</td>
		</tr>
		";
	}
}
$body = "<table style='width: 100%;'>$body</table>";
```
Тут есть лишний код: прописано условие, что $key сравнивается с project_name, admin_email и form_subject. Автор говорит что это код со старого скрипта и это условие можно удалить. Еще вопросы вызывает частое использование $body и $title - нужны ли они?

<br>

...и "Отправка сообщения":
```php
// Отправка сообщения:
	$mail->isHTML(true);
	$mail->Subject = $title;
	$mail->Body = $body;
	$mail->send();
} catch (Exception $e) {
	$status = "Сообщение не было отправлено. Причина ошибки: {$mail->ErrorInfo}";
}
```

---

<br><br><br><br><br><br>













## Требования к PHP-файлу

- PHP-файл с настройками (обработчик) должен быть один универсальный для всех вариантов отправки сообщений на почту. Он должен поддерживать любые варианты инпутов: текстовые, чекбоксы, радио, селекты, рейнджи и отправку файлов.

- Единственные места где нужно вносить изменения для нового проекта - это емейл отправителя, пароль отправителя, емейл получателя и заголовки-подписи для писем.

- Письма должны отправляться самым лучшим и надежным способом. Насколько знаю это SMTP с указанием емейла отправителя + пароль для приложений к нему. Желательно чтобы емейл отправителя и получателя был олдин и тот же.

- После отправки сообщения нужно чтобы пользователя не перекидывало со страницы с формой, чтобы эта страница не перезагружалась, чтобы JS-скрипт открывал сообщения об успешной или неуспешной отправке. А если скрипта нету, то в идеале добиться максимально похожего поведения с помощью только PHP-скрипта.

- Для отправки файлов вероятно стоит добавить грубую валидацию, чтобы пользователи не могли переслать откровенно огромные или опасные файлы. Для этого ограничить размер вложения 200 Мб и форматы файлов (например указать разрешенные).

---

## Требования к управляющему JS-скрипту

- Модульность: чтобы каждую отдельную функцию можно было просто удалить или наоборот добавить (например код инпута-ловушки, код маски или валидации) и при этом скрипт продолжал работать с остальными функциями.

- Форма должна работать без перезагрузки страницы, оставаться на этой странице, введенная информация не должна удаляться или мигать пока форма успешно не отправится.

- После отправки формы скрипт должен показать сообщение об успешной или неудачной отправке. Форма должна очиститься.

- Работа нескольких форм на странице, включая в модалке (от одних и тех же скриптов JS/PHP и от разных).

- Код современный, лаконичный, поддерживаемый всеми современными браузерами с возрастом до 2 лет (IE не надо).

- Желательно встроить в скрипт простые и полезные функции безопасности, например инпут-приманка, может есть еще что-то?

- Формы на всех этапах должны быть доступными на достаточно высоком уровне.

- Желательно добавить функционал отправки сообщений в телеграм, ватсапп.

---

### Подробнее что сделать в JS-скрипте

#### Скрипт для инпута-приманки

Что отправка работала и с приманкой и без приманки, а когда приманка заполнена, то письмо чтобы не отправлялось. Чтобы можно было просто добавлять или удалять в форме этот инпут, а в скрипте чтобы можно было легко удалить этот участок кода. На бэкенде ваш скрипт обработки может проверить, заполнено ли поле-приманка. Если это так, то отправка, вероятно, была от бота, и можно ее проигнорировать.

```html
<label class="honeypot" for="honeypot" style="display: none;" aria-hidden="true" hidden>Оставьте это поле пустым</label>
<input class="honeypot" id="honeypot" style="display: none;" type="text" autocomplete="off" tabindex="-1" aria-hidden="true" hidden>
```

#### Сообщение об успехе/неудаче отправки

- После успешной отправки сообщения скрипт должен показать пользователю что сообщение отправлено и очистить форму. Не знаю какое тут лучшее решение. Сейчас в HTML добавляю:
```html
<p class="form-success">Сообщение отправлено.</p>
<p class="form-failure">Сообщение не отправлено!</p>
```
И скрипт должен добавить к этим элементам класс ".active" чтобы они появились. Как вариант можно скрыть кнопку отправки и на ее месте показать сообщение об успехе, тогда макет не будет сдвигаться (но надо продумать хорошо ли это, как будет доступность?).

#### Скрипт для отправки файлов внедрить в общий скрипт отправки

Чтобы он работал когда есть поле отправки и не мешал когда его нет. Чтобы имел базовую валидацию отправляемых файлов: максимальный размер, форматы файлов, количество файлов? Или валидацию лучше делать в PHP-скрипте?


