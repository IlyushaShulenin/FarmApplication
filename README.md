# FarmApplication

REST приложение с микросервисной архитектурой для автоматизации работы с 
отчетамии на ферме

**Микросервис владельца:** https://github.com/IlyushaShulenin/FarmOwnerApi.git

**Микросервис работника:** https://github.com/IlyushaShulenin/FarmWorkerApi.git

----

## Стек технологий
### Из обязательных

- Java 17;
- Spring Boot 3;

### Из рекомендуемых

- Spring Security;
- JWT;
- Spring Data;
- PostgreSQL;
- Redis;
- Kafka;
- Docker;
- JSON;

### Дополнительные

- Liquibase для версионирования баз данных;

----

## Реализованный функционал

### Для владельца фермы

- Добавление новых продуктов;
- Удаление продуктов;
- Начисление баллов работнику;
- Удаление баллов работника;
- Создание плана для работника на день;
- Удаление плана
- Получение информации об объемах полученной продукции в сравнении с запланированным
    - В целом по ферме
    - Для заданного рабочего;
    - Для заданного рабочего за указанный месяц;
    - Получение данных в письма на электронную почту владельца;
- Аутентификация владельца с помощью JWT токена;
- Кэширование данных с помощью Redis;

### Для работника

- Создание отчета о проделанной работе и отправка его владельцу с помощью Kafka
- Аутентификация рабочего;
- Получение персольнальной информации для аутентифицированного пользователя:
    - Не закрытые планы планы;
    - Начисленные баллы
    - Данные о произведенной продукции в сравнении с заданными планами
        - За все время;
        - За заданный месяц;
    
# Демонстрация работы

---

## FarmOwnerApi
Микросервис, отвечающий за работу на стороне владельца фермы

---

### Аутентификация владельца фермы

Данные владельца:

![URL](imgs/sign-in.png)

Полученный JWT токен:

![Token](imgs/token.png)

### Получить список всех рабочих

![URL](imgs/worker-get.png)

![Response](imgs/worker-get-result.png)

При создании бина workerService внутри него инициализируется Redis Repository
и при этом заполняется данными полученными из основной БД, поэтому при запросе данные о рабочих
достаются из кэша.

### Получить рабочего по id

![URL](imgs/worker-3-get.png)

![Response](imgs/worker-get-3-response.png)

Проверяется наличие данные о рабочем в кэше. Если там его нет, то проверяется БД. При обнаружении
нужного рабочего, он записывается к кэш и возвращается в качестве результата. Иначе - исключение.

### Зарегистрировать нового рабочего

![URL and request](imgs/worker-post.png)

Результат запроса:

![Response](imgs/worker-post-result.png)

При регистрации рабочего указываеются уникальный email, пароль для входа, имя, фамилия.
Поле isWorking по умолчанию инициализируется значением true. Поле означает,что работник трудится на ферме.
В случае увольнение рабочего поле принимает значение false, это необходимо для сохранения планов и отчетов уволенного 
сотрудника.

При выполнении запроса переданные данные с помощью Kafka отправляются на микросервис FarmWorkerApi и сохраняются
в БД.

![Saved worker](imgs/worker-post-result-in-db.png)

### Увольнение рабочего

![URL](imgs/worker-delete.png)

При увольнениии рабочего isWorking меняет значение с true на false. В дальнейшем, при получении данных о всех рабочих,  
он не будет отображаться. Так же этот рабочий не будет доступен по id и не сможет заходить в FarmWorkerApi.

![Result in database](imgs/worker-delete-db.png)

### Получение списка всех продуктов

![URL](imgs/product-get.png)

По аналогии с workerService все продукты достаются из кэша.

![Response](imgs/product-get-response.png)

### Получение продукта по id

![URL](imgs/porduct-get-2.png)

![Response](imgs/product-2-get.png)

### Добавление нового продукта

![URL](imgs/product-post.png)

Тело запроса

![Response](imgs/product-body-post.png)

Результат запроса

![Request](imgs/product-post-result.png)

При добавлении нового продукта он сохраняется в БД и отправляется сообщение на FarmWorkerApi
и там тоже сохраняется в БД. По умолчанию поле isProduced принимает значение true и обозначает, что продукт производится.
При удалении продукта isProduced становится false.

БД на стороне FarmWorkerApi

![Database](imgs/product-post-result-db.png)

### Удаление продукта

![URL](imgs/product-delete.png)

isProduced становится false при этом сохраняются планы и отчеты для данного продукта

БД на стороне FarmWorkerApi

![Database](imgs/product-delete-db.png)

### Получение списка всех планов

![URL](imgs/plan-get.png)

Все планы достаются из кэша

Полученный список содержит информацию о рабочем, о продукте, необходимый объем и дату

![Reponse](imgs/plan-get-response.png)

### Получение плана по id

![URL](imgs/plan-get-10.png)

![Reponse](imgs/plan-get-10-response.png)

### Создание нового плана

![URL](imgs/plan-post.png)

Тело запроса

![Request](imgs/plan-post-request.png)

Результат запроса

![Response](imgs/plan-post-response.png)

Новый план сохраняется и на стороне FarmWorkerApi

![Database](imgs/plan-post-db.png)

### Удаление плана

![URL](imgs/plan-delete.png)

При удалении плана на FarmWorkerApi отправляется сообщение.

БД на стороне FarmWorkerApi

![URL](imgs/plan-delete-db.png)

### Получение списка всех баллов

![URL](imgs/score-get.png)

Баллы достаются из кэша

![Response](imgs/score-get-response.png)

### Получение баллов по id

![URL](imgs/score-get-8.png)

![Response](imgs/score-get-8-response.png)

### Добавление баллов рабочему

![URL](imgs/score-post.png)

Тело запроса 

![Request](imgs/score-post-request.png)

Результат запроса

![Response](imgs/score-post-response.png)

Баллы сохраняются в БД у FarmWorkerApi

![Database](imgs/score-post-db.png)


### Удаление баллов по id

![URL](imgs/score-delete.png)

Баллы удаляются в том числе и на стороне рабочего

![Database](imgs/score-delete-db.png)

### Получение продуктивности рабочего

![URL](imgs/stat-get.png)

![Response](imgs/stat-get-response.png)

### Отправка отчетности на почту

![URL](imgs/send-url.png)

![Response](imgs/send-response.png)

---

## FarmWorkerApi

Микровсервис, отвечающий за работу на стороне рабочего.

---

### Аутентификация рабочего

Данные рабочего

![URL](imgs/worker-sign-in.png)

Сгенерированный JWT токен

![Token](imgs/worker-token.png)

### Составление отчета

![URL](imgs/report-post.png)

Тело запроса

![Request](imgs/report-post-request.png)

Результат запроса

![Response](imgs/report-post-response.png)

После сохранения отчета на стороне FarmWorkerApi на сторону владельца отправляется сообщение,
содержащее данные из отчета. Поле planIsCompleted принимает значение true, если план по заданному продукту
и в заданный день выполнен, иначе false(в том числе если соответствующего плана нет).

БД у владельца:

![Database](imgs/report-post-db.png)

### Получение отчетности для аутентифицированного рабочего

Незакрытые планы рабочего:

![URL](imgs/worker-personal-info-plan.png)

Ответ:

![Response](imgs/worker-personal-info-plans.png)

Ответ пуст так как для вошедшего рабочего все планы закрыты.

Баллы рабочего:

![URL](imgs/worker-personla-info-score.png)

![Response](imgs/worker-personal-info-score-reponse.png)

Отчетность по продуктивности рабочего:

![URL](imgs/worker-personal-info-productivity.png)

Ответ:

![Response](imgs/worker-personal-info-prod-response.png)

# PS

Видео снять не получилось, так как мой несчастный ноутбук не вывозит такую "сумасшедшую" нагрузку(.

