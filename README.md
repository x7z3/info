# Web Test Framework Core (WTFc) 0.2.7.1

Core для построения тестового веб-фреймворка. Версия для **Java 17**.

### Кратко о возможностях фреймворка:

Построен на следующих фреймворках:

- TestNG
- Selenide
- RestAssured
- Allure
- AssertJ
- AspectJ

Все вышеперечисленные зависимости нет необходимости подключать дополнительно в вашем `pom.xml`, они идут вместе с WTFc.

`ExtendedElement` - расширенная версия SelenideElement c дополнительными методами.

`WebView` - Аннотация для тестового метода для предварительной инициализации драйвера, без необходимости использования
методов beforeX и прочих базовых тестовых классов. Тип драйвера можно передать в параметре к аннотации.
Доступны варианты: Chrome, Firefox, Safari, Edge.

`MobileView` - Аннотация для тестового метода для запуска инициализации мобильного отображения драйвера chrome.

`WebView` и `MobileView` можно использовать свободно в одном тестовом классе с любым типом многопоточности. То есть
можно свободно создать сюит из тестовых классов с разными вариациями запускаемых авто тестов.

`AsyncWait` - Асинхронные ожидания. Асинхронное ожидание позволяет запустить параллельное ожидание какого либо условия,
не блокируя основной поток выполнения.

`AsyncRun` - Асинхронный запуск кусков кода. Запуск параллельного исполняемого кода с передачей данных контекста.

`Interruptible` - Аннотация для методов поток которых нужно завершить при истечении таймаута. Тестовые методы по
умолчанию
обрабатываются данной логикой. Время ожидания, с версии 0.2.1.0 можно передать и в аргументах аннотации.

`Loggable` - Аннотация для логирования запускаемых методов. В логе будет отображаться полная сигнатура метода, и
возвращаемое значение.
Тестовые методы автоматически попадают в лог.

`SelenideTimouts` - Аннотация для установки отдельных таймаутов для конкретного тестового метода/класса.

`Page` - Аннотация для инициализации поля класса страницы в тестовом классе.

`Steps` - Аннотация для инициализации поля класса степа в тестовом классе.

`Init` - Аннотация аналогична Page и Steps.

`RetryableListener` - Аннотация для регистрации листенерОВ перезапуска (`IRetryAnalyzer`). По умолчанию вы можете
зарегистрировать,  
только один такой листенер на тест, но с данной аннотацией можно повесить на тест сколько угодно подобных листенеров.

`ChromeFactory`, `FirefoxFactory`, `OperaFactory`, `SafariFactory`, `EdgeFactory` - Аннотации для классов-фабрик
веб-драйверов Selenide.  
Достаточно создать соответствующий класс, изменить его под свои нужды. При использовании аннотации `WebView` будет
использована  
соответствующая версия аннотированной фабрики.

## Пример тестового класса с аннотациями

Пример тестового класса с разными запускаемыми браузерами в одном тестовом классе:

```java

import ru.rtlabs.test.core.Browser;
import ru.rtlabs.test.core.annotation.*;
import ru.rtlabs.test.core.driver.factory.DefaultFirefoxDriverFactory;

class SomeTest {

    @Page // Автоматическая инициализация класса страниц без необходимости явной инициализации
    private SomePage sharedPage;

    @Steps // Автоматическая инициализация класса степпов без необходимости явной инициализации
    private SomeSteps sharedSteps;

    @WebView(Browser.EDGE)
    public void edgeWebTest() {
        // some code
    }

    // Мобильный отображение
    @MobileView
    public void mobileOrientedWebTest() {
        // some code
    }

    // Использование стандартной фабрики, обозначенной либо через аннотацию, либо через
    @WebView(Browser.SAFARI)
    public void safariWebTest() {
        // some code
    }

    // Использование кастомной фабрики, если нужны отличные параметры отличные от стандартных фабрик
    @WebView(value = Browser.CUSTOM, browser = DefaultFirefoxDriverFactory.class)
    public void customFirefoxFactoryWebTest() {
        // some code
    }

    @WebView
    @SelenideTimeouts(
            timeout = 40000, // ожидание элементов для тестового метода ниже
            pageLoadTimeout = 30000 // ожидание загрузки страницы
    )
    public void defaultDriverWithDifferentTimeouts() {
        // some code
    }
}
```

Как можно заметить описывать кастомную фабрику не так уж удобно, но можно создать свою `...View` аннотацию.

```java

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@WebView(value = Browser.CUSTOM, browser = DefaultFirefoxDriverFactory.class)
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface YourCustomView {
}
```

И после применить ее к тестовому методу или классу.

```java
public class SomeTest {

    @YourCustomView
    public void customViewTest() {
        // some code
    }
}
```

## Специальные Selenoid Capabilities

Selenoid имеет ряд своих спец. опций (https://aerokube.com/selenoid/latest/#_special_capabilities) которые можно  
задать через код.

Каждая опция задается с помощью отдельного класса, который реализует
интерфейс `ru.rtlabs.test.core.driver.capability.Capability`  
и отмечен аннотацией `@SelenoidCapability`.

```java
import ru.rtlabs.test.core.annotation.SelenoidCapability;
import ru.rtlabs.test.core.driver.capability.Capability;

@SelenoidCapability
public class SomeCapability implements Capability {
    @Override
    public String name() {
        return "yourCapabilityName"; // точное имя капабилити
    }

    @Override
    public Object value() {
        return "value of the property above"; // любое допустимое значение которое может принимать данная капабилити
    }
}
```

Возвращаемые значения методов подобных классов используются для задания спец. опций Selenoid.

### Класс AbstractCapability

Кроме варианта с реализацией интерфейса `ru.rtlabs.test.core.driver.capability.Capability` так же можно
воспользоваться  
классом `ru.rtlabs.test.core.driver.capability.AbstractCapability`. Данный класс содержит в себе удобные методы поиска
значения указанной капабилити в порядке:

- Системные переменные
- Методы отмеченные аннотацией `@Parameter` в конфигурационном классе (класс отмеченный аннотацией `@WTFConfig`)
- Файлы пропертей

Соответствие ищется по возвращаемому значению метода `name()`. Единственный нюанс касается файлов пропертей.
Например, значение опции `hostEntries` должно располагаться по пути в директории ресурсов `selenoid/host-entries`.

```java
import ru.rtlabs.test.core.annotation.Parameter;
import ru.rtlabs.test.core.driver.capability.AbstractCapability;

public class SomeCapability extends AbstractCapability {
    @Override
    public String name() {
        return "capabilityName";
    }

    @Override
    public Object value() {
        // данный метод будет искать опцию "capabilityName" сперва в системных переменных
        // Далее в конфигурационном классе по имени метода "capabilityName" (пример ниже)
        // И далее в файле пропертей 'selenoid/capability-name'

        // Использоваться будет первое найденное значение.
        return findStringArrayValue();
    }
}

@WTFConfig
public class Configuration {

    @Parameter
    public Object capabilityName() {
        return "value";
    }
}
```

Списки в значении определяются наличием запятой, точки с запятой, и переносов строк.

### Встроенные SelenoidCapability классы в ядро

`ru.rtlabs.test.core.driver.capability.EnableVideoCapability`
`ru.rtlabs.test.core.driver.capability.EnableVncCapability`
`ru.rtlabs.test.core.driver.capability.HostsEntriesCapability`
`ru.rtlabs.test.core.driver.capability.NameCapability`

Если вам нужна своя реализация данных классов, или капабилитей которые они устанавливают, то нужно выставить
игнорирование
пакета `ru.rtlabs.test.core.driver.capability` в конфигурационном классе.

```java
import ru.rtlabs.test.core.annotation.ScanPackages;

@WTFConfig
@ScanPackages(ignore = "ru.rtlabs.test.core.driver.capability")
public class Configuration {
    // ...
}
```

## Подключение WFTc в качестве зависимости

Подключается зависимость с помощью прописывания зависимости, как указано ниже.

```xml

<project>
    ...
    <dependencies>
        ...
        <dependency>
            <groupId>ru.rtlabs</groupId>
            <artifactId>web-test-framework-core</artifactId>
            <version>0.2.7.1</version>
        </dependency>
        ...
    </dependencies>
    ...
</project>
```

Данный тип использования подразумевает, что вам так же потребуется добавить конфигурацию Maven плагина surefire, и
allure.

```xml

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>ru.rtlabs</groupId>
    <artifactId>your-cool-test-framework</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <!--Параметр ниже это путь до дефолтного тест сюита TestNG. Его необходимо будет создать-->
        <test.suite>${basedir}/src/main/resources/test_suites/default.xml</test.suite>

        <maven.surefire.version>3.0.0-M5</maven.surefire.version>
        <maven.compiler.version>3.8.1</maven.compiler.version>
        <allure-maven.version>2.11.2</allure-maven.version>
        <aspectj.version>1.9.9.1</aspectj.version>
        <web-test-framework-core.version>0.2.7.1</web-test-framework-core.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven.surefire.version}</version>
                <configuration>
                    <reportsDirectory>${basedir}/test-output</reportsDirectory>
                    <suiteXmlFiles>
                        <suiteXmlFile>${test.suite}</suiteXmlFile>
                    </suiteXmlFiles>
                    <testFailureIgnore>true</testFailureIgnore>
                    <argLine>
                        --add-opens java.base/java.lang=ALL-UNNAMED
                        --add-opens java.base/java.lang.reflect=ALL-UNNAMED
                        -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar"
                    </argLine>
                    <systemPropertyVariables>
                        <allure.results.directory>${project.build.directory}/allure-results</allure.results.directory>
                    </systemPropertyVariables>
                </configuration>
            </plugin>
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>${allure-maven.version}</version>
                <configuration>
                    <reportDirectory>${project.build.directory}/report</reportDirectory>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>ru.rtlabs.test.core</groupId>
            <artifactId>web-test-framework-core</artifactId>
            <version>${web-test-framework-core.version}</version>
        </dependency>
    </dependencies>
</project>
```

## Подключение WTFc в качестве родительского pom.xml

WTFc можно подключить как родительский `pom.xml`. Для этого необходимо добавить в ваш `pom.xml` следующую секцию:

```xml

<parent>
    <groupId>ru.rtlabs.test.core</groupId>
    <artifactId>web-test-framework-core-pom</artifactId>
    <version>0.2.7.1</version>
</parent>
```

Что это даст в итоге. Вам не нужно будет конфигурировать surefire и allure плагины. Итоговый `pom.xml` вашего тестового
фреймворка будет выглядеть примерно так.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>ru.rtlabs</groupId>
    <artifactId>your-great-test-framework</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <parent>
        <groupId>ru.rtlabs.test.core</groupId>
        <artifactId>web-test-framework-core-pom</artifactId>
        <version>0.2.7.1</version>
    </parent>
</project>
```

И этого будет достаточно, чтобы запускать тесты с помощью Selenide и RestAssured.

## Классы страниц

Каждый класс страницы должен быть наследован от класса `ru.rtlabs.test.core.ui.Page`. В нем располагаются дополнительные
делегированный функционал, инициализатор PageFactory (то что позволяет полям аннотированным FindBy работать), и метод
`driver()` (важно использовать именно метод, а не созданное поле), для получения мультипоточного экземпляра драйвера.

## Конфигурация

Основные параметры WTFc можно переопределить либо в файле `core.properties`, или через переменные среды.  
Список переменных `core.properties`

```properties
# timeouts
page.load.timeout=30
script.load.timeout=20
explicit.wait.timeout=20
# general
host.url=http://localhost
driver.host=http://localhost:4444/wd/hub
mobile.device=Pixel 2
driver=chrome
suite.name=Default Suite
# system properties
debug=false
threads.count=1
threads.type=METHODS
headless=false
remote=false
test.interrupt.timeout=300
```

Не забываем, что все данные переменные можно передать через Maven `mvn -Dvariable=value`. Переданные системные
переменные  
будут иметь более высокий приоритет.

## TestNG Listeners

WTFc работает на классах слушателях (listeners). Конфигурация запуска слушателей определяется в ресурсном
файле `listeners`.
Это позволяет избавиться от необходимости аннотировать классами-слушателями тестовые классы, или избавиться от базовых
тестовых классов с подобными аннотациями.

Пример дефолтного файла `listeners`:

```
ru.rtlabs.test.core.listener.SuiteListener
ru.rtlabs.test.core.listener.SelenideListener
ru.rtlabs.test.core.listener.RetryListener
ru.rtlabs.test.core.listener.WebViewListener
ru.rtlabs.test.core.listener.MobileViewListener
ru.rtlabs.test.core.listener.FieldsListener
ru.rtlabs.test.core.listener.AllureHostInfoListener
ru.rtlabs.test.core.listener.AttachmentsListener
ru.rtlabs.test.core.listener.QuitDriverListener
```

Порядок слушателей соблюдается при их запуске согласно порядку описания их в файле `listeners`.
В файле `listeners` разрешены комментарии через двойной слеш `//`;

## Скачивание зависимостей NEXUS

Для возможности скачивания зависимостей с WTFc нужно прописать следующие настройки в `~/.m2/settings.xml`

```xml

<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd">

    <mirrors>
        <mirror>
            <id>nexus-public</id>
            <url>https://repo1.maven.org/maven2/</url>
            <!--
                Если нужно чтобы все зависимости шли через NEXUS используем нижний параметр.
                При этом скорость скачивания зависимостей будет ниже.

                <url>http://nexus.gosuslugi.local/content/groups/public</url>
            -->
            <mirrorOf>central</mirrorOf>
        </mirror>
        <mirror>
            <id>nexus-repo-qa-automation-libs</id>
            <url>http://nexus.gosuslugi.local/content/repositories/repo-qa-automation-libs/</url>
            <mirrorOf>repo-qa-automation-libs</mirrorOf>
        </mirror>
    </mirrors>

    <profiles>
        <profile>
            <id>nexus</id>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>http://central</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>repo-qa-automation-libs</id>
                    <url>http://nexus.gosuslugi.local/content/repositories/repo-qa-automation-libs/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>http://central</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>repo-qa-automation-libs</id>
                    <url>http://nexus.gosuslugi.local/content/repositories/repo-qa-automation-libs/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>
</settings>
```

## Настройка пакета поиска классов фабрик

Для изменения области поиска аннотированных классов, можно указать пакеты которые будут входить в нее входить.  
Сделать это можно с помощью конфигурационного класса.

```java
import ru.rtlabs.test.core.annotation.ScanPackages;
import ru.rtlabs.test.core.annotation.WTFConfig;

@WTFConfig // Аннотация указывающая, что данный класс является классом конфигурации WTFc
@ScanPackages({"ru.rtlabs.driver_factory", "ru.rtlabs.test_driver_factory"})
// Пакеты в которых будет осуществляться поиск
public class YourConfigClass { // Имя класса может быть любым
}
```

Без конфигурирования поиск идет по всем доступным пакетам.

## Классы ядра о которых следует знать

`Framework` - хранит в себе базовые константы и методы поиска системных параметров.  
`PropertiesManager` и `PropertiesReader` - классы для работы с файлами параметров.  
`ClassPath` - класс для поиска аннотированных классов.  
`AsyncRun` - класс для асинхронного запуска кода.  
`AsyncWait` - класс асинхронных ожиданий по условию.

`Page` - базовый класс для используемого PageObject. Содержит базовые методы для работы с UI.  
`JsonReader` - класс для работы с Json.  
`XmlReader` - класс для работы с XML.  
`FileUtils` - класс для считывания файлов ресурсов.
`SuppressExceptionUtils` - класс со статическими методами запуска кода с исключениями и их перехвата.

