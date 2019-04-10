<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Документация по API</title>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
    </head>
    <body>
        <p>
            Базовый url для запросов - <strong>https://uxcandy.com/~shapoval/test-task-backend</strong>
        </p>

        <p>
            Ожидаемый MIME-type для POST-запросов - multipart/form-data
        </p>
    
        <p>
            Ответ сервера - в формате json.
            <br>
            Ответ может содержать два поля:
            <ul>
                <li><strong>status</strong> - текстовая строка - "ok" в случае успешного запроса, "error" в случае ошибки</li>
                <li><strong>message</strong> - текстовая строка или ассоциативный массив - сообщение с результатами запроса (в случае успешного выполнения), сообщение об ошибке (в случае ошибки), поля может не быть или оно может быть пустым</li>
            </ul>
        </p>

        <p>
            <strong>Для всех ответов GET-параметр "developer" является обязательным.</strong>
            <br>
            Просьба указывать в этом параметре своё имя.
            <br>
            Если параметр не получен, будет возвращено сообщение об ошибке:
            <pre>
    {
        "status": "error",
        "message": "Не передано имя разработчика"
    }
            </pre>
        </p>


        <h3>Список задач (/):</h3>

        <p>
            Обратите внимание - есть разница между <strong>https://uxcandy.com/~shapoval/test-task-backend?developer=Name</strong> и <strong>https://uxcandy.com/~shapoval/test-task-backend/?developer=Name</strong>.
            Правильным является последний вариант.
        </p>

        <p>
            Допустимые параметры (GET):
            <ul>
                <li><strong>sort_field</strong> <i>(id | username | email | status)</i> - поле, по которому выполняется сортировка</li>
                <li><strong>sort_direction</strong> <i>(asc | desc)</i> - направление сортировки</li>
                <li><strong>page</strong> - номер страницы для пагинации</li>
            </ul>
        </p>

        <p>В ответе сервер в поле "message" передаёт два параметра - "tasks" (список задач на странице) и "total_task_count" (общее количество задач)</p>

        <p>
        Пример ответа:
            <pre>
    {
        "status": "ok",
        "message": {
            "tasks": [
                {
                    "id": 1,
                    "username": "Test User",
                    "email": "test_user_1@example.com",
                    "text": "Hello, world!",
                    "status": 10,
                },
                {
                    "id": 3,
                    "username": "Test User 2",
                    "email": "test_user_2@example.com",
                    "text": "Hello from user 2!",
                    "status": 0,
                },
                {
                    "id": 4,
                    "username": "Test User 3",
                    "email": "test_user_3@example.com",
                    "text": "Hello from user 3!",
                    "status": 0,
                }
            ],
            "total_task_count": "5"
        }
    }
            </pre>
        </p>


        <h3>Добавление задачи (/create):</h3>

        <p>
            Обязательные параметры (POST):
            <ul>
                <li><strong>username</strong> - текстовое поле - имя пользователя, который добавляет задачу</li>
                <li><strong>email</strong> - текстовое поле - email-адрес пользователя, который добавляет задачу, email-адрес должен быть валидным</li>
                <li><strong>text</strong> - текстовое поле - текст задачи</li>
            </ul>
        </p>

        <p id="create_example_request">
            Пример запроса (jquery ajax):
            <pre>
    $(document).ready(function() {
        var form = new FormData();
        form.append("username", "Example");
        form.append("email", "example@example.com");
        form.append("text", "Some text");

        $.ajax({
            url: 'https://uxcandy.com/~shapoval/test-task-backend/create?developer=Example',
            crossDomain: true,
            method: 'POST',
            mimeType: "multipart/form-data",
            contentType: false,
            processData: false,
            data: form,
            dataType: "json",
            success: function(data) {
                console.log(data);
            }
        });
    });
            </pre>
        </p>

        <p>
            Пример ответа (успешное добавление):
            <pre>
    {
        "status": "ok",
        "message": {
            "id": 8,
            "username": "Example user",
            "email": "123@example.com",
            "text": "Some text",
            "status": 0,
        }
    }
            </pre>
        </p>

        <p>
            Пример ответа (ошибка при добавлении):
            <pre>
    {
        "status": "error",
        "message": {
            "username": "Поле является обязательным для заполнения",
            "email": "Неверный email",
            "text": "Поле является обязательным для заполнения",
        }
    }
            </pre>
        </p>


        <h3>Редактирование задачи (/edit/:id):</h3>

        <p>
            Для редактирования задачи нужно передать в POST, помимо самих редактируемых полей, специальный токен (используйте строку "beejee" в качестве значения этого поля)
            и дополнительный параметр-подпись "signature".
        </p>

        <p>
            Для генерации подписи нужно:
            <ul>
                <li>собрать все поля, отправляемые в POST, в том числе token (кроме самого поля signature),</li>
                <li>отправлять можно только редактируемые поля + token + signature</li>
                <li>отсортировать по алфавиту все поля, кроме token (поле token должно быть последним),</li>
                <li>выполнить URL-кодированние параметров запроса (имени параметры и значения параметра), кодирование происходит по стандарту RFC 3986 (например, строка "example@example.com" кодируется в "example%40example.com")</li>
                <li>собрать отсортированные URL-кодированые поля в одну строку запроса (params_string), разделитель между имененем параметра и его значением - символ "=", между разными параметрами - символ "&" (получится, например, status=0&text=SomeText&token=beejee). Напоминаем, что параметры должны быть кодированы по стандарту RFC 3986,</li>
                <li>рассчитать md5-хеш от URL-кодированной строки запроса (md5(params_string)) и отправить этот md5-хеш в поле 'signature' в POST вместе с другими параметрами</li>
            </ul>
        </p>

        <p>
            Допустимые параметры редактирования:
            <ul>
                <li><strong>text</strong> - тестовое поле - текст задачи</li>
                <li><strong>status</strong> - числовое поле - статус выполнения задачи (0 - задача не выполнена, 10 - задача выполнена)</li>
            </ul>
        </p>


        <p>
            Пример ответа:
            <pre>
    {
        "status": "ok"
    }
            </pre>
        </p>
    </body>
</html>
