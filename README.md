
### Задача №1: настройка CI

Напоминаем, CI — это чаще всего отдельная система — сервер, набор серверов, облако — в которой ваш код и ваши автотесты собираются в автоматическом режиме без вашего непосредственного участия. Вы лишь настраиваете CI для того, чтобы при возникновении определённых событий, например, push в репозиторий, стартовал процесс сборки и прогона тестов.

Детальнее про CI вы можете узнать, погуглив «Continuous integration», «Jenkins», «GitLab CI», «AppVeyor», «Travis», «CircleCI», «GitHub Actions».

Важно: иногда можно настроить CI так, что там всегда будет success 😈! Не забывайте убедиться, что в CI сборка действительно падает, если вы запушите в GitHub падающий тест.

Подсказка
* Общая схема работы выглядит следующим образом: CI должен запустить целевой сервис, который вы и тестируете, в фоновом режиме и ваши автотесты. Для этого мы будем на этот раз использовать возможности Bash.

* Для того чтобы запустить целевой сервис, есть несколько вариантов, самый простой из которых — положить JAR-файл прямо в ваш репозиторий. Когда AppVeyor будет выкачивать исходники автотестов, он выкачает и ваш сервис.

Конечно, вы должны понимать, что в реальной жизни артефакты (собранный целевой сервис) хранятся в специальных системах, и процесс выкачивания будет зависеть от того, где и как хранится артефакт.

Ваш целевой сервис (SUT — System under test), расположен в файле app-mbank.jar, этот же файл используется в примерах на лекции. Вам нужно его положить в каталог artifacts вашего проекта, который необходимо создать.

Поскольку файлы с расширением .jar находятся в списках ```.gitignore```, вам нужно принудительно заставить Git следить за ними: ````git add -f artifacts/app-mbank.jar.````

После чего сделать git push. Обязательно удостоверьтесь, что файл попал в репозиторий.

### AppVeyor
AppVeyor — одна из платформ, предоставляющих функциональность Continuous integration. В базовом варианте она бесплатна.

Шаг 0. Конфигурация как код
Поскольку вручную настраивать каждый проект в системе Continuous integration — лишняя трата времени, мы будем хранить всю конфигурацию для AppVeyor в специальном файле с названием .appveyor.yml.

Важно: внимательно посмотрите на структуру деморепозитория из ваших лекций. Большинство инструментов используют подход «Configuration by exception», то есть конфигурируется только то, что не соответствует настройкам по умолчанию. Поэтому у вас всего два пути: либо использовать настройки по умолчанию и писать как можно меньше конфигурации, либо идти против системы и писать много конфигурации, а потом ещё и отлаживать её.

Файл этот должен храниться в самом репозитории на GitHub, тогда AppVeyor будет автоматически подхватывать настройки из него:

![](https://camo.githubusercontent.com/963b14a3833628b932acbc92e969a08e61953f5669e170ec745ca626b1e4ac80/68747470733a2f2f692e696d6775722e636f6d2f476737423936312e706e67)

Yaml — формат данных, используемый многими системами для хранения конфигурации.

AppVeyor предлагает вам два вида серверов, на которых можно проводить сборку вашего приложения: под управлением Windows или под управлением Linux. Можно организовать сборку под несколькими сразу, но для упрощения мы пока остановимся только на одной ОС для каждого вашего проекта.

##### Linux Config
````
image: Ubuntu2004  # образ для сборки

stack: jdk 11  # версия JDK

branches:
only:
- master  # ветка git

build: off  # будем использовать свой скрипт сборки

install:
# Запускаем SUT,
# имя файла SUT будет отличаться в каждой задаче.
# & означает, что в фоновом режиме не блокируем терминал для запуска тестов,
# обязательно должен быть для запуска SUT в CI
- java -jar ./artifacts/app-mbank.jar &
- chmod +x gradlew

build_script:
# Запускаем автотесты
# Для проектов на базе Selenide необходимо добавить параметр для запуска браузера
# в headless режиме -Dselenide.headless=true, параметр --info должен остаться
- ./gradlew test --info
````
Естественно, у вас должен возникнуть вопрос, а что будет, если SUT не успеет стартовать к моменту запуска автотестов?

Тогда ваши тесты упадут. Что с этим делать и как классифицировать подобные случаи, мы поговорим на следующих лекциях.

Напоминаем, ваш build.gradle должен выглядеть вот так:
````
plugins {
id 'java'
}

group 'ru.netology'
version '1.0-SNAPSHOT'

sourceCompatibility = 11
compileJava.options.encoding = 'UTF-8'
compileTestJava.options.encoding = 'UTF-8'

repositories {
mavenCentral()
}

dependencies {
testImplementation 'io.rest-assured:rest-assured:5.3.1'
testImplementation 'org.junit.jupiter:junit-jupiter:5.6.1'
testImplementation 'io.rest-assured:json-schema-validator:4.3.1'
}

test {
useJUnitPlatform()
}
````
Шаг 1. Регистрация

![](https://camo.githubusercontent.com/d640a7ac15e4e848cb53419bff2a418d62705f0fba3f0908066ce76ccb278f45/68747470733a2f2f692e696d6775722e636f6d2f5275676d7a37442e706e67)
Шаг 2. Регистрация через GitHub
AppVeyor предоставляет бесплатный тарифный план для публичных репозиториев GitHub, авторизация — также через GitHub:

![](https://camo.githubusercontent.com/5968ef7f30b221e40a72b4f3b4fa2e76227ce29882e14e5690c10d2fb0c93542/68747470733a2f2f692e696d6775722e636f6d2f6a587666744d622e706e67)

Шаг 3. Разрешение доступа
При подключении необходимо разрешить AppVeyor получать уведомления:

![](https://camo.githubusercontent.com/1cbe9ea7e7fff8cb0d86c99223f08104aa091cfeb436c2414e74c5ffa259bda2/68747470733a2f2f692e696d6775722e636f6d2f324676636a39362e706e67)

Шаг 4. Создание проекта
После авторизации станет доступной панель управления, где можно создать новый проект:

![](https://camo.githubusercontent.com/e2f4e63a9fac7dcbd3ad8b241be5e01accccef7222c02cf1e16a2bba246a6dc0/68747470733a2f2f692e696d6775722e636f6d2f7755424b6259592e706e67)

Авторизуйте AppVeyor в качестве OAuth App:

![](https://camo.githubusercontent.com/20153e55d20c9495ae58c784d52f84c98d243807383c318f8242be0260840642/68747470733a2f2f692e696d6775722e636f6d2f6f5161644c4c6a2e706e67)

Это даст возможность приложению получать уведомления о ваших push в репозиторий, модификации и т. д.

![](https://camo.githubusercontent.com/1ad82e03b892b4401dba33074165c3354587798f0f660e427723147956f9ac35/68747470733a2f2f692e696d6775722e636f6d2f326a77483653612e706e67)

Детальнее об OAuth вы можете прочитать на:

https://oauth.net/2/,
https://auth0.com/docs/protocols/oauth2.

Шаг 5. Выбор репозитория
После авторизации достаточно будет нажать кнопку ADD напротив необходимого репозитория:

![](https://camo.githubusercontent.com/7c4f2fe99fdfcbbe8726b1199a9f74dc60d9f1ece69409ff935bc975ec4b629f/68747470733a2f2f692e696d6775722e636f6d2f3456514d45366a2e706e67)

После настройки всего процесса каждый push в ветку master GitHub-репозитория будет приводить к запуску сборки на AppVeyor.

Шаг 6. Status badge
На странице Settings — Badges AppVeyor предлагает код для бейджика статуса вашего проекта:

![](https://camo.githubusercontent.com/b43f320a0e6e269800d23f16bfebf9a8c96d79e485e5e01b0092ce5ae5833f94/68747470733a2f2f692e696d6775722e636f6d2f444543745a6a672e706e67)

Этот badge необходимо разместить в файле README.md для отображения текущего статуса вашего проекта:

![](https://camo.githubusercontent.com/40e8c2605729193c44c7e51965108265b3a8d50c6de8f65e2e215eb5e183d800/68747470733a2f2f692e696d6775722e636f6d2f5639634f654a4f2e706e67)

**Важно: убедитесь, что вы не скопировали бейджик с другого проекта. За такую хитрость ДЗ будет отправляться на доработку.**

### Задача №2: JSON Schema
JSON Schema предлагает нам инструмент валидации JSON-документов. С описанием вы можете познакомиться по адресу.

Как строится схема:
````
{
"$schema": "http://json-schema.org/draft-07/schema", // версия схемы: https://json-schema.org/understanding-json-schema/reference/schema.html
"type": "array", // тип корневого элемента: https://json-schema.org/understanding-json-schema/reference/type.html
"items": { // какие элементы допустимы внутри массива: https://json-schema.org/understanding-json-schema/reference/array.html#items
   "type": "object", // должны быть объектами: https://json-schema.org/understanding-json-schema/reference/object.html
   "required": [ // должны содержать следующие поля: https://json-schema.org/understanding-json-schema/reference/object.html#required-properties
      "id",
      "name",
      "number",
      "balance",
      "currency"
   ],
"additionalProperties": false, // дополнительных полей быть не должно
"properties": { // описание полей: https://json-schema.org/understanding-json-schema/reference/object.html#properties
   "id": {
      "type": "integer" // целое число: https://json-schema.org/understanding-json-schema/reference/numeric.html#integer
   },
   "name": {
      "type": "string", // строка: https://json-schema.org/understanding-json-schema/reference/string.html
      "minLength": 1 // минимальная длина — 1: https://json-schema.org/understanding-json-schema/reference/string.html#length
   },
   "number": {
      "type": "string", // строка: https://json-schema.org/understanding-json-schema/reference/string.html
      "pattern": "^•• \\d{4}$" // соответствует регулярному выражению: https://json-schema.org/understanding-json-schema/reference/string.html#regular-expressions
   },
   "balance": {
      "type": "integer" // целое число: https://json-schema.org/understanding-json-schema/reference/numeric.html#integer
   },
   "currency": {
      "type": "string" // строка: https://json-schema.org/understanding-json-schema/reference/string.html
   }
  }
 }
}
````
Начнем с изучения проекта, проверим необходимые условия для его реализации.

#### Что нужно сделать:

Шаг 1. Зависимости проекта
````
dependencies {
   testImplementation 'io.rest-assured:rest-assured:5.3.1'
   testImplementation 'io.rest-assured:json-schema-validator:4.3.1'
   testImplementation 'org.junit.jupiter:junit-jupiter:5.7.0'
}

````
Шаг 2. JSON схема
В каталоге resources в src/test и должен лежать файл схемы.

![img.png](https://github.com/netology-code/aqa-homeworks/raw/master/api-ci/pic/schema.png)

Шаг 3. Проверка ответа сервиса на соответствие схеме
Один из классов проекта должен содержать операцию проверки соответствия ответа схеме.
````
      // код теста
      .then()
          .statusCode(200)
          .body(matchesJsonSchemaInClasspath("accounts.schema.json"));
````

В тестовом классе должен быть импорт статического метода
````
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;
````
Удостоверьтесь, что тесты проходят при соответствии ответа схеме и падают, если вы поменяете что-то в схеме, например, тип для id.

Шаг 4. Доработать схему

Изучите документацию на тип object и найдите способ валидации значения поля на два из возможных значений: «RUB» или «USD».

Доработайте схему соответствующим образом, удостоверьтесь, что тесты проходят, в том числе в CI.

Поменяйте «RUB» на «RUR» в схеме и удостоверьтесь, что тесты падают, в том числе в CI.

Пришлите на проверку ссылку на ваш репозиторий. Удостоверьтесь, что в истории сборки были как success, так и fail, иначе будет не видно, как вы проверяли, что сборка падает в CI.

