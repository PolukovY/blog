+++
author = "Yevgen.P"
title = "Все ще конкатенуєш рядки, для того щоб зробити URL в Spring."
date = "2019-03-05"
description = "UriComponentsBuilder in Spring"
tags = [
    "Spring",
]
categories = [
    "Spring",
]
thumbnail = "images/TechWorld-With-Yevgen-URL_builder.jpg"
+++


Скажу чесно я теж інколи таким грішу. <span class="emojify">🙈</span>

Тому якщо у тебе є Spring, а ти ще конкатенуєш рядки доєднуйся буде точно корисну гарантую. Ну що поїхали.

## По перше потрібно подивитися чи є у вас потрібна залежність 

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>...</version>
</dependency>
```

## Ну що ж час прикладів:

### 1. Побудова URI

```java
@DisplayName("Should build URL with schema and host")
@Test
void shouldBuildUrlWithSchemaAndHost() {
    var uriBuilder = UriComponentsBuilder.newInstance()
        .scheme("http")
        .host("TechWorld-With-Yevgen.com")
        .build();

    assertEquals("http://TechWorld-With-Yevgen.com", uriBuilder.toUriString());
}
```

Як ми можемо помітити, ми створили новий екземпляр UriComponentsBuilder, а потім надали тип схеми, хост. 
Цей простий приклад може бути корисним, коли ми хочемо виконати перенаправлення на інше посилання нашого веб-сайту.

### 2. Створення закодованого URI

```java
@DisplayName("Should build encoded URL with schema and host and path")
@Test
void shouldBuildEncodedUrlWithSchemaAndHostAndPath() {
    var uriBuilder = UriComponentsBuilder.newInstance()
        .scheme("http")
        .host("TechWorld-With-Yevgen.com")
        .path("join tech")
        .build()
        .encode();

    assertEquals("http://TechWorld-With-Yevgen.com/join%20tech", uriBuilder.toUriString());
}
```

Різниця в цьому прикладі полягає в тому, що ми хочемо додати пробіл між словом join і числом tech. 
Відповідно до [RFC 3986](https://www.ietf.org/rfc/rfc3986.txt) це було б неможливо. Нам потрібно закодувати посилання, щоб отримати дійсний результат, використовуючи метод encode().

### 3. Створення URI з параметрами запиту

```java
@DisplayName("Should build URL with query params")
@Test
void shouldBuildWithQueryParams() {
    var uriBuilder = UriComponentsBuilder.newInstance()
        .scheme("https")
        .host("www.google.com")
        .query("q={keyword}").buildAndExpand("TechWorld-With-Yevgen");

    assertEquals("https://www.google.com?q=TechWorld-With-Yevgen", uriBuilder.toUriString());
}
```

Запит буде додано в основну частину посилання. Ми можемо надати кілька параметрів запиту, використовуючи дужки {…}. 
Їх буде замінено ключовими словами в методі під назвою buildAndExpand(…).

Також під кінець розкажу коли б не використовув і чому: <span class="emojify">🙊</span>

- В проекті не використовується Spring і не хочеться за раді цього додавати його. 
- В тестах простіше об'єднати рядки, якщо у вас є все й одразу і ніколи не буде змінюватися.
- Можна пошукати бібліотеку яка робить потрібну річ і додати її.
- Звісно якщо є бажання, то можна написати, але я б це розглянув в останній момент.

Використовував точно: <span class="emojify">🙉</span>

- Якщо вже Spring доданий до проекту, то краще використовувати ті інструменти які вже є.


### Висновки

Сподіваюся дані поради по  UriComponentsBuilder були корисні. 
Основними перевагами UriComponentsBuilder є гнучкість використання формування URI.
