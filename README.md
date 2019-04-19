# Node.JS VK CoinAPI

# Установка
### Windows:
* Скачайте и установите последнюю версию [Node.JS](https://nodejs.org/en/download/)
* Создайте в удобном месте папку, например **vkcoin**
* Перейдите в командную строку: Win + R > cmd
* Перейдите в папку: **cd (путь до вашей папки)**
* Пропишите: npm i nodejs-vkcoin-api

### Ubuntu:
* Установите Node.JS по [этому](https://www.digitalocean.com/community/tutorials/node-js-ubuntu-16-04-ru) гайду
* Создайте в удобном месте папку, например **vkcoin**
* Перейдите в папку: **cd (путь до вашей папки)**
* Пропишите: npm i nodejs-vkcoin-api
# Начало работы
Для начала использования, вам нужно создать в своей папке исполняемый файл, пусть это будет **index.js**

Теперь его нужно открыть и импортировать библиотеку:
```js
const VKCOINAPI = require('nodejs-vkcoin-api');

const vkcoin = new VKCOINAPI(options = {});
```

|Опция|Тип|Описание|
|-|-|-|
|key|String|Ключ для взаимодействия с API|
|userId|Number|Ваш айди ВК|
|token|String|Ваш токен|

### Где взять эти значения
* Получение ключа (key): [описано в начале этой статьи](https://vk.com/@hs-marchant-api)
* Получение айди вк (userId):

Откройте свою аватарку и в адресной строке вы увидите подобное: **https://vk.com/fakeman.cat_fmc?z=photo236908027_456259706%2Falbum236908027_0%2Frev**

Вашим айди будет являться число после слова **photo**. В этом случае **236908027**

* Получение токена (token):

Откройте [эту](https://oauth.vk.com/authorize?client_id=6378721&scope=1073737727&redirect_uri=https://api.vk.com/blank.html&display=page&response_type=token&revoke=1) ссылку и нажмите разрешить

После этого в адресной строке будет подобное: **https://api.vk.com/blank.html#access_token=xxxxxxxxxxxx&expires_in=0&user_id=user_id&email=email**

Токеном будет являться строка от **access_token** до **&expires**. В этом случае **xxxxxxxxxxxx**
# Методы
getTransactionList - Получает список ваших транзакций

```js
async function run() {
    const result = await vkcoin.getTransactionList(tx);

    console.log(result);
}

run().catch(console.error);
```

|Параметр|Тип|Описание|
|-|-|-|
|tx|Array<Number>|Массив айди переводов для получения ИЛИ [1] - последняя 1000 транзакций, [2] - 100|
#
sendPayment - Делает перевод другому пользователю (в десятичных долях)

```js
async function run() {
    const result = await vkcoin.sendPayment(toId, amount); // 1 коин = 1000 ед.

    console.log(result);
}

run().catch(console.error);
```

|Параметр|Тип|Описание|
|-|-|-|
|toId|Number|Айди получателя|
|amount|Number|Сумма перевода|
#
getLink - Получает ссылку для перевода

```js
function run() {
    const link = vkcoin.getLink(amount, fixation);

    console.log(link);
}

run().catch(console.error);
```

|Параметр|Тип|Описание|
|-|-|-|
|amount|Number|Сумма перевода|
|fixation|Boolean|Фиксированная сумма или нет|
#
formatCoins - Делает получаемое из API значение коинов читабельным. Например, приходит значение 1234567890. Этот метод сделает значение таким: 1 234 567,890

Это можно использовать в паре с другим методом:
```js
async function run() {
    const trans = await vkcoin.getTransactionList([2]);

    const fixTrans = trans.response.map((tran) => {
        tran.amount = vkcoin.formatCoins(tran.amount);

        return tran;
    });

    console.log(fixTrans);
}

run().catch(console.error);
```
|Параметр|Тип|Описание|
|-|-|-|
|coins|Number|Входящее значение коинов|
#
getBalance - Получает баланс по айди пользователей

getMyBalance - Получает баланс текущего пользователя

```js
async function run() {
    const balances = await vkcoin.getBalance([1, 100, 236908027]);
    const myBalance = await vkcoin.getMyBalance();

    console.log({ balances, myBalance });
}

run().catch(console.error);
```

Среди этих методов аргумент принимает только getBalance:


|Параметр|Тип|Описание|
|-|-|-|
|userIds|Array<Number>|Массив айди пользователей|
# Updates
**updates** - Позволяет "прослушивать" события в VK Coin. Пока что я реализовал перехват входящего платежа, но вскоре придумаю что-нибудь ещё. И да, впервые работаю с сокетами :)
### Запуск
Для запуска прослушивания есть 2 метода: startPolling и startWebHook
#
startPolling - Запускает обмен запросами между клиентом и сервером в режиме реального времени (WebSocket). Является лучшим и быстрым способом получения событий:

```js
async function run() {
    await vkcoin.updates.startPolling(callback);
}

run().catch(console.error);
```

|Параметр|Тип|Описание|
|-|-|-|
|callback|Function|Функция обратного вызова, принимает в себя аргумент **data**|
#
startWebHook - Запускает сервер на 8181 порте для получения событий. Может не работать на Windows и является неоптимальным способом получения событий. В этом случае можно обойтись без асинхронной функции:

```js
vkcoin.updates.startWebHook(options = {});
```

|Опция|Тип|Описание|
|-|-|-|
|url|String|Адрес вашего сервера для получения событий|
|port|Number|Порт для запуска сервера (8181 - по умолчанию)|
# События
updates.onTransfer - Перехватывает входящие платежи, принимает один аргумент

```js
async function run() {
    await vkcoin.updates.startPolling();

    vkcoin.updates.onTransfer((event) => {
        console.log(event);
    });
}

run().catch(console.error);
```

Или

```js
vkcoin.updates.startPolling(async(data) => {
    console.log(data);

    vkcoin.updates.onTransfer((event) => {
        console.log(event);
    });
});
```

Или

```js
vkcoin.updates.startWebHook({
    url: 'myawesomevds.to', // Public IP / URL
});

vkcoin.updates.onTransfer((event) => {
    console.log(event);
});
```

event - Объект, который хранит в себе информацию о платеже:

|Параметр|Тип|Описание|
|-|-|-|
|amount|Number|Количество коинов, которые послупили на счёт|
|fromId|Number|Айди плательщика|
|id|Number|Айди платежа|

Стоит отметить, что startWebHook получает только платежи по ссылке.
