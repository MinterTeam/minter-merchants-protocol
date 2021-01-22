# Minter Merchants Protocol

Текущая версия протокола: 0.4.0

- [Status of this document](#status-of-this-document)
- [MMP](#mmp)
- [Usage](#usage)
  * [Aggregator](#aggregator)
  * [User](#user)
  * [Merchant](#merchant)
- [MMP-file](#mmp-file)
  * [Provider](#provider)
  * [Offers](#offers)
  * [Fields](#fields)
    + [Field role Quantity](#field-role-quantity)
    + [Field role Phone](#field-role-phone)
    + [Field role Username](#field-role-username)
- [Aggregation of MMP-feeds](#aggregation-of-mmp-feeds)
- [How to become a merchant](#how-to-become-a-merchant)
- [Checkout page query parameters](#checkout-page)
- [Callback to user](#callback-to-user)
  * [Transaction payload cipher](#transaction-payload-cipher)


## Status of this document

Данный документ является черновиком протокола и находится в активной разработке, любые части документа могут быть изменены в будущем. Документ предназначен для ознакомления будущими мерчантами и сбора мнений от сообщества. Дискуссию по документу предпочтительно вести в GitHub-issues или в чате [@MinterDevChat](https://t.me/MinterDevChat)

## MMP

Протокол MMP позволяет любому мерчанту подключить по единому стандарту свой фид цифровых товаров к кошельку [wallet.minter.org](https://wallet.minter.org) или любым другим приложениям, поддерживающим этот протокол. Подключенные мерчанты имеют возможность предоставлять информацию о своих товарах, их параметрах и наличии, а также совершать продажи и обновлять статусы.

Minter Merchants Protocol подразумевает создание и последующее обновление [MMP-файла](#mmp-file), в котором содержится вся необходимая информация о мерчанте и его товарах, представленная в унифицированном стандарте.


## Usage

### Aggregator
Агрегатор собирает предложения различных мерчантов через их MMP-фиды. Агрегатор раз в 10 минут запрашивает каждый фид, актуализируя информацию о ценах и наличии товаров. 

### User
- попадает на страницу со списком офферов
- выбирает товар, 
- нажимает "перейти к оплате",
- генерируется чек для мерчанта, он действителен до блока +60 от текущего (≈5 минут). 
- юзер вместе с чеком перенаправляется к мерчанту на страницу оплаты, где видит информацию о магазине и товаре.
- юзер нажимает "оплатить" (кнопка оплаты доступны после проверки баланса адреса и наличия товара).
- после оплаты юзер попадает на уникальную страницу с информацией о получении товара

### Merchant
После попадания пользователя в магазин на страницу оплаты, мерчант получает данные о заказе [из query параметров](#checkout-page): id оффера, чек и пароль для обналичивания, публичный ключ для шифрования пейлоада.

Мерчант проверяет что у него есть достаточное количество товара, проверяет что в чеке находится достаточное количество средств, кнопка "оплатить" становится активной, юзер на неё нажимает, мерчант обналичивает чек.

После обналичивания чека мерчант генерирует для пользователя уникальную ссылку на страницу на своём сайте и сразу перенаправляет его на неё. Там пользователь сможет просмотреть информацию о получении своего заказа: статус заказ (готов, в обработке), сам товар, если он быть отгружен в электронном виде, например, redeem code. На этой странице пользователь может произвести какие-то дополнительные действия, если требуются, чтобы получить свой товар: согласовать доставку, связаться с техподдержкой и т. д.

После выполнения заказа мерчант [отправляет уведомление](#callback-to-user) в виде пустой транзакции на адрес пользователя с пейлоадом, содержащим информацию о заказе, чтобы история о заказе была всегда доступна юзеру, и чтобы он понимал, в какой момент заказ был исполнен. Пейлоад должен быть [зашифрован](#transaction-payload-cipher) публичным ключом юзера. Транзакцию нужно отправлять с того же адреса, с которого был обналичен чек, уведомления с других адресов не будут отображаться у юзера, чтобы избежать спама.


## MMP-file
Предоставляется мерчантом в формате .json  
Должен обновляться по мере изменения информации о товарах, либо раз в 5-10 минут, при отсутствии изменений. Фиды возрастом больше 10 минут не будут обрабатывать агрегатором и соответственно не будут отображены пользователю.  
Версия должна соответствовать последней версии протокола, фиды с устаревшими версиями не будут обрабатываться агрегатором.  
Допустимые кодировки MMP-файла: UTF-8  
В качестве разделителя целой и дробной частей любых чисел используется только точка.
```json
{
  "protocol": "mmp",
  "version": "1.0.0",
  "timestamp": "2020-11-10T15:03:07.0196Z",
  "provider": {
    "name": "Minter Push",
    "slug": "minter-push",
    "checkout_url": "https://minterpush.com/mmp-checkout",
    "logo": "https://minterpush.com/logo.png",
    "support": "https://minterpush.com/support",
    "legal": [
      {
        "title": "Privacy policy",
        "url": "https://minterpush.com/privacy"
      },
      {
        "title": "Terms of service",
        "url": "https://minterpush.com/terms"
      }
    ],
    "offer_list": [{
        "id": "0c765fbf-554f-4af0-9463-a348a797c5ea",
        "name": "Product",
        "vendor": "Apple",
        "category": "Games",
        "url": "https://...",
        "picture": "https://...",
        "short_description": "about",
        "description": "about",
        "price_bip": "100",
        "quantity": 1000,
        "field_list": [
          {
            "role": "quantity",
            "name": "Amount",
            "description": "Amount to top up the phone"
          },
          {
            "role": "phone",
            "name": "Mobile phone number",
            "description": "Use international phone format"
          }
        ] 
    }]
  }
}
```

- `version`: версия MMP протокола
- `timestamp`: должен соответствовать дате и времени генерации MMP-файла на стороне провайдера. Дата должна иметь формат [ISO 8601](https://ru.wikipedia.org/wiki/ISO_8601): `YYYY-MM-DDThh:mm:ss[.SSS]Z`
- `provider`: информация о провайдере, который предлагает различные товары.

### Provider 
`provider`
- `name`: название провайдера
- `logo`: ссылка на логотип в формате .SVG, .PNG, .JPG
- `slug`: идентификатор провайдера, будет использоваться при формировании URL, допустимые символы regex: `/[a-z0-9-]/`
- `url`: URL главной страницы
- `support`: URL ссылка на службу поддержки
- `legal`: массив объектов, каждый из которых представляет собой юридические документ, с полями `title` и `url`
- `checkout_url`: [страница оплаты](#checkout-page), куда будет перенаправляться юзер для, данные заказа будут переданы в query параметрах, 
- `offer_list`: информация о предложениях/товарах магазина (пример: Apple iTunes Gift Card). Каждый товар является элементом массива

### Offers 
`provider.offer_list[]`
- `id`: идентификатор товара внутри магазина
- `name`: название товара
- `vendor`: название производителя
- `category`: категория товара, доступны для выбора "Games", "Phone", "Gift cards", остальные значения будут попадать в "Other"
- `url`: ссылка на страницу товара в магазине
- `picture`: URL ссылка на картинку товара
- `short_description`: короткое описание товара, без HTML тэгов
- `description`: полное описание товара, без HTML тэгов, возможны переносы строки
- `price_bip`: стоимость товара в эквиваленте BIP
- `quantity`: количество товара; `1234` - число, сколько максимум пользователь может купить; `0` — out of stock; `-1` — не ограничено, например, при пополнении телефона
- `field_list`: Список полей, для заполнения юзером, например, номер телефона, на котором нужно пополнять баланс

### Fields
`provider.offer_list[].field_list[]`
- `role`: роль, это значение будет использоваться для передачи в query параметрах на страницу оплаты, доступные значения [`quantity`](#field-role-quantity) - целое количество товара, [`phone`](#field-role-phone) - номер телефона в международном формате
- `name`: Заголовок поля отображаемый юзеру
- `description`: Описание, подсказка отображаемая юзеру

#### Field role Quantity
`field_list[].role == 'quantity'`

Целое количество товара.  
Будет отображаться только для бесконечных товаров, у которых задано количество `"quantity": -1`. Если количество товара конечное, то для его выбора будут использоваться стрелочки вверх/вниз и настройка полей `"field_list": [{"role": "quantity"}]` будет игнорироваться.

#### Field role Phone
`field_list[].role == 'phone'`

Номер телефона в международном формате.  
Используется только для указания номера для пополнения баланса.  
Нельзя использовать для получения контактного номера для связи с пользователем.

#### Field role Username
`field_list[].role == 'username'`

Аккаунт пользователя.
Используется для начисления средств по имени пользователя.


## Aggregation of MMP-feeds
Агрегатор собирает фиды у подтверждённых мерчантов из списка //@TODO ссылка на гитхаб

При агрегации фидов нужно брать только те, где:
- таймстамп фида не старше 10 минут
- версия протокола не является устаревшей

Агрегатор предоставляет данные в следующем формате:
```json
{
  "version": "1.0.0",
  "timestamp": "2020-11-10T15:03:07.0196Z",
  "provider_list": [{}, {}]
}
```

- `version` версия MMP протокола
- `timestamp` должен соответствовать дате и времени сбора MMP-файлов. Дата должна иметь формат [ISO 8601](https://ru.wikipedia.org/wiki/ISO_8601): `YYYY-MM-DDThh:mm:ss[.SSS]Z`
- `provider_list`: массив провайдеров, каждый из которых получен из поля [`provider`](#provider) в MMP-файле мерчанта


## How to become a merchant
В репозитории https://github.com/MinterTeam/minter-merchants-protocol будет собиратся список проверенных мерчантов, каждый мерчант должен будет сделать пулл-реквест, чтобы добавить свой фид

merchant-list.json
```json
{
    "version": "1.0.0",
    "feed_list": [
        {
            "name": "Minter Push",
            "feed": "https://…",
            "homepage": "https://",
            "email": "contact@minterpush.com",
            "description": "",
            "company_register_number": "",
            "company_extract": ""
        }
  ]
}
```

- `homepage` - 
- `email` - адрес для связи
- `description` - описание, какие услуги предоставляются

Данные предоставляются, если деятельность ведётся от юридического лица 
- `company_register_number` - регистрационный номер компании
- `company_extract` - публичные данные компании, 

## Checkout page
Страница на сайте мерчанта, на которой он сможет обналичить чек от пользователя. Страница должны быть выполнена в дизайне Minter. [Готовый HTML+CSS шаблон](https://github.com/MinterTeam/mmp-checkout-page)

Пользователь попадает на эту страницу редиректом из кошелька, данные о заказе передаются в query параметрах:

- `check`: "Mc..." - чек для оплаты товара
- `password`: "checkpassword" - пароль для обналичивания чека
- `public_key`: "0x…" - публичный ключ пользователя для шифрования пейлоада
- `offer_id`: "0c765fbf-554f-4af0-9463-a348a797c5ea" - идентификатор оффера
- `quantity`: "1" - количество товара
- `phone`: "79991231234" - опциональное поле, передаётся если в оффере был указан `"field_list": [ {"role": "phone"} ]`

Пример перехода на страницу оплаты

`
https://minterpush.com/mmp-checkout?offer_id=123&quantity=1&check=Mcf89f8869366a795a58372d01830b995a80881bc16d674ec8000080b8416a842d9f3a2e115683191f170f2a044e6a1ab1962afa8ec119a9ced8b9637c0b139316ccf3fb8130b98f50a4b8f1ddc2a1c08f7921a1a79b89e20685d89818a3011ca0ef93008572a207a0d11c6fbae3364ec2f44683af239014073348c3c97b4e4eb9a031c6c5297656c43916e7e3730bcb5fa70459dd626a3ddb9174e9e58955204c57&password=fcViycYgwea4m-iz&public_key=0x66c808e6b5be6d6620934bc6ffa2b8b47f9786c002bfb06d53a0c27535641a5d1
`



## Callback to user
Мерчант посылает уведомление после завершения обработки заказа.  
Уведомления посылаются в виде пустой транзакции на адрес пользователя, сообщение будет находиться в транзакции в поле `payload` и будет [зашифровано](#transaction-payload-cipher) публичным ключом пользователя.  
Транзакцию нужно отправлять с того же адреса, с которого был обналичен чек, уведомления с других адресов не будут отображаться у юзера, чтобы избежать спама.

формат: 
```
mmp{{version}} {{merchant_public_key}}{{encrypted_data}}
```

пример:
```
mmp1.0.0 ASdasdASDASdasd...
```

- `mmp{{version}}` - заголовок с версией протокола, отделяется пробелом от бинарных данные
- `merchant_public_key` - публичный ключ мерчанта для расшифровки сообщения
- `encrypted_data` - зашифрованный [JSON5](https://json5.org/) (чтобы не нужно было название полей оборачивать кавычками, тем самым экономим байты в пейлоаде) с полями `u`, `n`, `m`, `s`

```json5
{
    u: "https://example.com/unique-order-url",
    n: "Offer name",
    m: "Your WorldOfWarcraft account NagibatorPWNZ is fulfilled with 1000 Gold",
    s: "YOUR_REDEEM_CODE"
}
```

`u` - URL, обязательное поле, уникальная ссылка на сайт мерчанта, где юзер может может посмотреть информацию о заказе и получить свой товар 
`n` - name, название товара
`m` - message, обязательное поле, сообщение пользователю
`s` - shipment, опциональное поле, в нём содержится сам товар, используется, если товар является, например, подарочным купоном в виде строки символов


### Transaction payload cipher
Сообщение в `payload`'e транзакции шифруется публичным ключом пользователя.
Будет использоваться гибридная схема шифрования на основе публичного ключа [ECIES](https://cryptobook.nakov.com/asymmetric-key-ciphers/ecies-public-key-encryption) на кривой secp256k1. Кривая выбрана для унификации кода с Minter и Ethereum. Без HMAC[^1].

- мерчант получает от юзера compressed публичный ключ длиной 33 байта, если нужно, восстанавливает из него uncompressed (65 байт)
- для каждого сообщения генерирует свою новую пару приватный-публичный ключ. **Внимание:** нельзя переиспользовать приватный ключ для шифрования, т.к. это может позволить злоумышленнику дешифровать исходное сообщение
- использует [ECDH](https://cryptobook.nakov.com/asymmetric-key-ciphers/ecdh-key-exchange) для получения общего секрета (shared secret, 65 байт) из публичного ключа пользователя и своего сгенерированного приватного ключа
- из общего секрета длиной 65 байт вычленяет Х координату (Px, 32 байта) получает секретный ключ для шифрования с помощью [HKDF](https://en.wikipedia.org/wiki/HKDF) хэширования[^2]. В качестве функция хэширования используется SHA512. Из результата SHA512 берётся первые 32 байта.
- зашифровывает сообщение секретным ключом с использованием симметричного шифрования AES256 CBC, в качестве вектора инициализации IV будет использована версия mmp протокола, например "mmp1.0.0"
- отправляет пользователю свой сжатый (compressed) публичный ключ и зашифрованное сообщение, итоговый пейлоад будет состоять из следующих частей:
  * префикса с версией протокола "mmpX.X.X" (который так же является вектором инициализации), 8+ байт[^3]
  * пробела, отделяющего префикс от бинарных данных, 1 байт
  * сжатого публичного ключа отправителя, 33 байт
  * зашифрованного текста

[^1]: В данной схеме [HMAC](https://en.wikipedia.org/wiki/HMAC) не используется, т.к. наличие транзакций в блокчейне с нужного адреса уже является авторизацией
[^2]: Общий секрет нельзя использовать напрямую, т.к. он недостаточно равномерно распределён. И поэтому нужно брать хэш от него.
[^3]: Версия может занимать больше 8 байт, например mmp12.34.567 - 12 байт

Реализация MMP ECIES на JavaScript
[github.com/MinterTeam/mmp-ecies-js](https://github.com/MinterTeam/mmp-ecies-js)




