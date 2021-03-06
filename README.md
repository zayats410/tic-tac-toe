## Tic-tac-toe
> Non-working version because of a broken server

**Requirements:**

Выведите список игр и кнопку "Создать игру"

После загрузки страницы, вы должны установить подключение через WebSocket по адресу `ws://xo.t.javascript.ninja/games`

Через этот вебсокет вам будут приходить уведомления следующих форматов:
* `{action: 'add', id: '345678'}` - добавить в список "Существующие игры". 
* `{action: 'remove', id: '345678'}` - удалить из списка "Существующие игры" игру с id ”345678". Она просто должна исчезнуть
* `{action: 'startGame', id: 'PLAYERID'}` - сервер зовет вас на игру на которую вы регистрировались командой register. (понадобится позже)
*  `{error: 'message'}` - вы делаете что-то не так и вебсервер сообщает вам об этом.

При нажатии на кнопку создания игры вы должны выполнить следующие шаги:
* отключить кнопку "Создать игру" (disabled)
* отправить POST запрос на сервер по адресу `http://xo.t.javascript.ninja/newGame`, в ответ вы получите результат следующего вида: `{yourId: 'YOURID'}`
* отправить по WebSocket строку вида {register: "GAMEID" } где GAMEID - ID полученный вами на сервере с предыдущего шага
* если на любом из 2 предыдущих шагов возникла ошибка - вывести на странице сообщение "Ошибка создания игры" и включить кнопку "Создать игру"


При получении команды через вебсокет на начало игры (команда `startGame` из задания 3.5), выполнить следующие шаги:
* **N1** вывести на страницу сообщение Ожидаем начала игры" и отключить кнопку "Создать игру"
* **N2**  отправить POST запрос с содержимым вида `{player: 'PLAYERID', game: ’GAMEID'}` по адресу `http://xo.t.javascript.ninja/gameReady`, где PLAYERID - ID полученный вами по вебсокету (при получении action `startGame`) на шаге W3, a GAMEID - ID игры, который вы получали при выполнение POST-запроса на `http://xo.t.javascript.ninja/newGame`
* **N3** если в ответ на POST запрос из **N2** получен код ответа `410 (Gone)` - вывести на странице сообщение "Ошибка старта игры: другой игрок не ответил"
* **N4** если запрос из **N2** провалился с кодом, отличным от `410` - вывести на странице сообщение "Неизвестная ошибка старта игры"
* **N5** если запрос из **N2** завершился успешно, то Вам прийдет ответ от сервера: `{side: 'х'}` или `{side: 'o'}`, который говорит о том, на какой стороне вы играете

* **J1** У вас есть вариант присоединения к существующей игре. Выводите список игр в списке. При клике по элементу списка - отправьте запрос по вебсокету `{register: "GAMEID" }` - где GAMEID - ID игры, к которой вы хотите присоединиться

Перефразирую процесс подключения. Клик по "новая игра", создает "игровую комнату”. `GAMEID` - это ID игровой комнаты. Когда вы выполняете команду register: GAMEID вы говорите "Сервер, я хочу учавствовать в игре с ID GAMEID". Как только таких участников двое - игра стартует, сервер генерирует вам ID игроков (PLAYERID) и присылает Вам. Поэтому оба игрока получат разные ID в этот момент.

Игра ведется в поле 10x10 (можно генерировать в любой удобный для Вас момент). Игра начинается с чистого поля и хода крестика.

Ведение игры обеспечивается следующей логикой. Если сейчас Ваш ход, то при клике по ячейке вы должны выполнить следующие действия:

* **G1** отправить `POST` запрос с содержимым вида `{move: 2}` и заголовками `Game-ID: GAMEID` и `Player-ID: PLAYERID` (где `GAMEID` и `РLAYERID` ровно те ID, которые получены вами ранее) по адресу `http:/xo.t.javascript.ninja/move`. 2 - номер клетки, в которую выполняется ход. Нумерация от 1 до 100.

* **G2** В случае ответа 200 на **G1** поставить соответствующий крестик либо нолик по ячейке, по которой кликнули. Если в ответе сервера содержится поле win (может принимать значения 'х' либо 'о') - вывести сообщение, которое лежит в поле win. Игра выиграна кем-то.
* **G3** В случае провала запроса **G1** вывести сообщение, в котором либо содержимое поля `message` ответа, либо, если такого нет-текст "Неизвестная ошибка". Прекратить выполнение любой логики, связанной с игрой кроме кнопки новой игры

Если сейчас же не ваш ход то вы должны выполнять Long-Polling GET запрос на адрес `http://xo.t.javascript.ninja/move`.
* **Н1** В случае провала запроса - немедленно повторить запрос (это же Long Polling)
* **Н2** В случае получения ответа - если в ответе содержится поле move - выполнить ход другого игрока в эту клетку. Если в ответе сервера содержится поле win (может принимать значения 'х' либо 'о') - вывести сообщение, которое лежит в поле win. Игра выиграна кем-то.
* **В1** Кнопка новой игры должна иметь текст "Сдаться" если игра еще не выиграна кем-то и не произошла неизвестная ошибка. В противном случае кнопка должна иметь текст "Новая игра"
* **В2** При нажатии на кнопку "Новая игра" необходимо скрыть поле и отобразить список игр
* **ВЗ** При нажатии на кнопку "Сдаться", необходимо выполнить PUT-запрос по адресу `http://xo.t.javascript.ninja/surrender` передав такие же заголовки как в **G1**.
* **В4** При получении ответа 200 на **ВЗ** - скрыть скрыть поле и отобразить список игр
* **В5** При провале запроса **ВЗ** - вывести сообщение на странице: либо содержимое поля message ответа, либо, если такого нет - текст "Неизвестная ошибка". Прекратить выполнение любой логики, связанной с текущей игрой кроме кнопки новой игры.
