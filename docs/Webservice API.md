## The PrestaShop Webservice API

###  Подключение API

1. Переходим по URL `${LOCAL_IP}:8080/${PS_FOLDER_ADMIN}` и логинимся используя значения перменных `ADMIN_MAIL` и `ADMIN_PASSWD` (см. [docker-compose](../docker-compose.yml)).
Например, переходим по http://192.168.1.79:8080/admin4577 и вводим `admin@example.com` и `admin123` - попадаем в админку.
2. В админке переходим в Advanced Parameters → Webservice, чекаем `Enable PrestaShop's webservice` и нажимаем Save.
3. Нажимаем `Add new webservice key`, генерируем ключ (например `5QT3FP22HWUB5KBYQE14KLGG9Y693NXK`) и выставляем уровень доступа, 
например, `All` для `contacts`.
4. Теперь можем попробовать получить список всех контактов (по умолчанию данные отдаются в формате XML):
```shell
$ curl -u 5QT3FP22HWUB5KBYQE14KLGG9Y693NXK: 'http://192.168.1.79:8080/api/contacts/'
<?xml version="1.0" encoding="UTF-8"?>
<prestashop xmlns:xlink="http://www.w3.org/1999/xlink">
<contacts>
<contact id="1" xlink:href="http://192.168.1.79:8080/api/contacts/1"/>
<contact id="2" xlink:href="http://192.168.1.79:8080/api/contacts/2"/>
</contacts>
</prestashop>

```
5. Можно получить данные по конкретному контакту и изменить формат на JSON с помощью заголовка `Output-Format: JSON'`:
```shell
$ curl -H 'Output-Format: JSON' -u 5QT3FP22HWUB5KBYQE14KLGG9Y693NXK: 'http://192.168.1.79:8080/api/contacts/1' | jq
{
  "contact": {
    "id": 1,
    "email": "admin@example.com",
    "customer_service": "1",
    "name": "Webmaster",
    "description": "If a technical problem occurs on this website"
  }
}

```

###  Пример создания ресурса

Создадим новый контакт с помощью POST-запроса

1. Получим схему документа, которую нужно заполнить для создания нового контакта:
```shell
$ curl -u 5QT3FP22HWUB5KBYQE14KLGG9Y693NXK: 'http://192.168.1.79:8080/api/contacts?schema=blank'
<?xml version="1.0" encoding="UTF-8"?>
<prestashop xmlns:xlink="http://www.w3.org/1999/xlink">
<contact>
	<id><![CDATA[]]></id>
	<email><![CDATA[]]></email>
	<customer_service><![CDATA[]]></customer_service>
	<name><language id="1"><![CDATA[]]></language></name>
	<description><language id="1"><![CDATA[]]></language></description>
</contact>
</prestashop>

```
2. Для удобства можно сохранить эту схему в файл и изменить необходимые поля, например:
```shell
$ cat contact.xml 
<?xml version="1.0" encoding="UTF-8"?>
<prestashop xmlns:xlink="http://www.w3.org/1999/xlink">
<contact>
	<id><![CDATA[]]></id>
	<email><![CDATA[bruce.wayne@gmail.com]]></email>
	<customer_service><![CDATA[1]]></customer_service>
	<name><language id="1"><![CDATA[Chief Officer]]></language></name>
	<description><language id="1"><![CDATA[General purpose]]></language></description>
</contact>
</prestashop>
```
3. Делаем POST-запрос для создания нового контакта:
```shell
$ curl -X POST  -u 5QT3FP22HWUB5KBYQE14KLGG9Y693NXK: 'http://192.168.1.79:8080/api/contacts' -d "@contact.xml"
<?xml version="1.0" encoding="UTF-8"?>
<prestashop xmlns:xlink="http://www.w3.org/1999/xlink">
<contact>
	<id><![CDATA[3]]></id>
	<email><![CDATA[bruce.wayne@gmail.com]]></email>
	<customer_service><![CDATA[1]]></customer_service>
	<name><language id="1" xlink:href="http://192.168.1.79:8080/api/languages/1"><![CDATA[Chief Officer]]></language></name>
	<description><language id="1" xlink:href="http://192.168.1.79:8080/api/languages/1"><![CDATA[General purpose]]></language></description>
</contact>
</prestashop>
```
4. Проверяем, что контакт успешно создался:
```shell
$ curl -H 'Output-Format: JSON' -u 5QT3FP22HWUB5KBYQE14KLGG9Y693NXK: 'http://192.168.1.79:8080/api/contacts/3' | jq
{
  "contact": {
    "id": 3,
    "email": "bruce.wayne@gmail.com",
    "customer_service": "1",
    "name": "Chief Officer",
    "description": "General purpose"
  }
}
```
