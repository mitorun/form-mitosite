# mitosite-form
Базовая форма с отправкой сообщений на емейл.

## Настройка файла mail-exp.php

Дорабатывать нужно два участка кода, это "Формирование самого письма":
```
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
Тут есть лишний код: прописано условие, что $key сравнивается с project_name, admin_email и form_subject. Автор говорит что это код со старого скрипта и это условие можно удалить.

Еще вопросы вызывает частое использование $body и $title - нужны ли они?

____
...и "Отправка сообщения":
```
// Отправка сообщения:
	$mail->isHTML(true);
	$mail->Subject = $title;
	$mail->Body = $body;
	$mail->send();
} catch (Exception $e) {
	$status = "Сообщение не было отправлено. Причина ошибки: {$mail->ErrorInfo}";
}
```
