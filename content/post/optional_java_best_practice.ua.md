+++
author = "Yevgen.P"
title = "Готуємо Optional правильно, 5 корисних порад"
date = "2022-10-02"
description = "Типові помилки і як не попастися на них"
tags = [
    "Java",
    "Optional",
]
categories = [
    "Java",
]
thumbnail = "images/optional-java.webp"
+++


Ну що почнемо, сподіваюся що таке NPE тут розповідати не потрібно думаю кожен його отримував і знаєш, що з ним робити.
А ось Optional прийшов на допомогу і мав допомогти в цьому. Тут ми спробуємо зрозуміти чи допоміг він чи ...

## Ну що поїхали 🙈. Топ 5 корисних порад

### 1. Чи можу я з Optional отримати NPE?

І тут на жаль чудес немає, звісно можете. Приведу типовий приклад коли це можливо:

```java
Optional<User> getUserBad(int id) {
    Optional<User> user = null;
    //logic query db or external service
    return user;
}
```

Доволі типовий приклад витягнути користувача з БД, а якщо його немає повернути null.
Така типова помилка виникає коли починаєш використовувати Optional, але ще пишеш код в старому стилі.

Вірний варіант буде використовувати empty.
```java
Optional<User> getUserBad(int id) {
    Optional<User> user = Optional.empty();
    //logic query db or external service
    return user;
}
```

#### Висновок з цього прикладу ніколи не призначайте значення null Optional змінній.


### 2. Повернути значення якщо є чи константу

```java
public static final String UNDEFINED = "Undefined";

String userStatusBad(int id) {
    Optional<String> userStatus = Optional.empty(); // query db and get data could be empty
    // String status = UNDEFINED;
    if (userStatus.isPresent()) {
        status = userStatus.get();
    }

    return status;
}
```

Цей приклад повстю робочий, але можна не використовувати isPresent і get.

#### Висновок для того що б це зробити потрібно використати orElse

```java
String userStatus(int id) {
    Optional<String> userStatus = Optional.empty(); // query db and get data could be empty
    return userStatus.orElse(UNDEFINED);
}
```

Але не все так просто, як на перший погляд. Перший нюанс це те, що в orElse завжди виконується не зважаючи є значення чи ні.
Тому тут теж потрібно його використовувати з тими значеннями які не потрібно вираховувати. 
Гарний варіант це константи як у нас в прикладі.

### 3. Не використовуємо orElse якщо вам там потрібно сходити в БД чи ще кудись.

```java
User getFromCacheOrDb(int id) {
    return getFromCache(id)
        .orElse(getFromDB(id).orElseThrow(() -> new UserNotFoundException("User with id" + id)));
}

Optional<User> getFromCache(int id) {
    System.out.println("search in cache with id " + id);
    return Optional.of(new User("levik", new ArrayList<>()));
}

Optional<User> getFromDB(int id) {
    System.out.println("search in DB with id " + id);
    return Optional.empty();
}
```

Вивід виклику метода getFromCacheOrDb спочатку піде в кеш, а потім незалежно до результату сходить в бд. 
І вивід буде такий. Це може призвести для уповільнення нашої аплікації.

```java
search in cache with id 1
search in DB with id 1
```

Це вирішується методом orElseGet він якраз буде викликаний коли даних в кеші не буде.

```java
User getFromCacheOrDb(int id) {
    return getFromCache(id)
        .orElseGet(() -> getFromDB(id).orElseThrow(() -> new UserNotFoundException("User with id" + id)));
}
```

#### Висновок якщо у вас похід в БД чи ще щось краще використовуйте orElseGet для запобігання проблем продуктивністю.

### 4. Повернути результат чи помилку якщо його немає

```java
User findUser(int id) {
    Optional<User> user = getFromDB(id);
    if (user.isPresent()) {
        return user.get();
    } else {
        throw new NoSuchElementException();
    }
}
```

Доволі типовий приклад коли ви, щось витягли з БД але даних немає і вам потрібно викинути помилку.
Але і тут можна зробити не використовуючи isPresent і get є аналогічний метод який це робить orElseThrow

```java
User findUser(int id) {
    return getFromDB(id).orElseThrow();
}
```

Все чудово, але зазвичай. Ми створюємо власний виняток і повертаємо його. Чи є якісь варіанти? 


### 5. Повернути результат чи власну помилку якщо його немає

Думаю як написати з isPresent і get, а також своєю помилкою не виникне проблем. Але можливо є інші варіанти. 

```java
User getFromCacheOrDb(int id) {
    return getFromCache(id)
        .orElseGet(() -> getFromDB(id).orElseThrow(() -> new UserNotFoundException("User with id" + id)));
}
```

Наш попередній приклад з кешем якщо немає сходимо в БД і якщо ще там немає тоді повернемо помилку.
Використовуючи orElseThrow приймає Supplier.

Можна ще зробити трохи по іншому якщо використати дефолтний конструктор в помилці тоді код буде виглядати так.

```java
User getFromCacheOrDb(int id) {
    return getFromCache(id).orElseGet(() -> getFromDB(id).orElseThrow(UserNotFoundException::new));
}
```

#### Висновок доволі зручна і проста альтернатива isPresent і get. Покращує читабельність коду.


### Висновки сподіваюсь дані поради були корисними. Рекомендую читати JavaDoc якщо є певні сумніви він вам може стати в пригоді. 



