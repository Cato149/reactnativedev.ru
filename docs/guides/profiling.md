# Профилирование

Используйте встроенный профилировщик, чтобы получить подробную информацию о работе, выполняемой в потоке JavaScript и основном потоке рядом друг с другом. Доступ к нему можно получить, выбрав Perf Monitor в меню Debug.

Для iOS Instruments - бесценный инструмент, а для Android вам следует научиться использовать [`systrace`](profiling.md#profiling-android-ui-performance-with-systrace).

Но сначала [**убедитесь, что режим разработки выключен!**](performance.md#running-in-development-mode-devtrue) Вы должны увидеть `__DEV__ === false`, предупреждения уровня разработки выключены, оптимизации производительности включены в логах вашего приложения.

Другой способ профилирования JavaScript - использовать профилировщик Chrome во время отладки. Это не даст вам точных результатов, поскольку код выполняется в Chrome, но даст вам общее представление о том, где могут быть узкие места. Запустите профилировщик на вкладке Chrome `Performance`. График пламени появится в разделе `User Timing`. Чтобы просмотреть более подробную информацию в табличном формате, нажмите на вкладку `Внизу вверх`, а затем выберите `DedicatedWorker Thread` в верхнем левом меню.

## Профилирование производительности пользовательского интерфейса Android с помощью `systrace`.

Android поддерживает 10k+ различных телефонов и обобщен для поддержки программного рендеринга: архитектура фреймворка и необходимость обобщения для многих аппаратных целей, к сожалению, означает, что вы получаете меньше бесплатно по сравнению с iOS. Но иногда есть вещи, которые можно улучшить - и во многих случаях это вовсе не вина нативного кода!

Первый шаг для отладки этой ерунды - ответить на фундаментальный вопрос, куда тратится ваше время в течение каждого 16-минутного кадра. Для этого мы будем использовать стандартный инструмент профилирования Android под названием `systrace`.

`systrace` - это стандартный инструмент профилирования Android на основе маркеров (он устанавливается при установке пакета Android platform-tools). Профилированные блоки кода окружены маркерами начала/конца, которые затем визуализируются в виде красочной диаграммы. И Android SDK, и фреймворк React Native предоставляют стандартные маркеры, которые вы можете визуализировать.

### 1. Сбор трассировки

Во-первых, подключите устройство, демонстрирующее заикание, которое вы хотите исследовать, к компьютеру через USB и доведите его до точки непосредственно перед навигацией/анимацией, которую вы хотите профилировать. Запустите программу `systrace следующим образом:

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

Краткое описание этой команды:

-   `time` - продолжительность времени, в течение которого будет собираться трасса, в секундах.
-   `sched`, `gfx` и `view` - это теги android SDK (коллекции маркеров), о которых мы заботимся: `sched` дает вам информацию о том, что работает на каждом ядре вашего телефона, `gfx` дает вам графическую информацию, такую как границы кадра, а `view` дает вам информацию об измерении, расположении и передаче рисунка.
-   `-a <имя_вашего_пакета>` включает маркеры, специфичные для приложения, в частности, встроенные в фреймворк React Native. Имя `вашего_пакета` можно найти в `AndroidManifest.xml` вашего приложения и выглядит оно как `com.example.app`.

Как только трассировка начнет собираться, выполните анимацию или взаимодействие, которое вас интересует. В конце трассировки systrace выдаст вам ссылку на трассировку, которую вы можете открыть в браузере.

### 2. Чтение трассы

После открытия трассировки в браузере (предпочтительно Chrome) вы должны увидеть что-то вроде этого:

![Пример]SystraceExample.png)

!!!note "Подсказка"

    Используйте клавиши WASD для перемещения и масштабирования.

Если файл трассировки .html открывается неправильно, проверьте консоль браузера на наличие следующих ошибок:

![ObjectObserveError](ObjectObserveError.png)

Поскольку функция `Object.observe` в последних версиях браузеров была устаревшей, возможно, вам придется открыть файл из инструмента трассировки Google Chrome. Вы можете сделать это следующим образом:

-   Открываем вкладку в chrome chrome://tracing
-   Выбор загрузки
-   Выбираем html-файл, сгенерированный предыдущей командой.

!!!info "Включить подсветку VSync"

    Установите этот флажок в правом верхнем углу экрана, чтобы выделить границы 16 мс кадров:

    ![Enable VSync Highlighting](SystraceHighlightVSync.png)

    Вы должны увидеть полоски зебры, как на скриншоте выше. Если этого не произошло, попробуйте выполнить профилирование на другом устройстве: Известно, что у Samsung были проблемы с отображением vsync, в то время как устройства серии Nexus обычно довольно надежны.

### 3. Найдите свой процесс

Прокручивайте до тех пор, пока не увидите (часть) имени вашего пакета. В данном случае я профилировал `com.facebook.adsmanager`, который отображается как `book.adsmanager` из-за глупых ограничений на имена потоков в ядре.

Слева вы увидите набор потоков, которые соответствуют строкам временной шкалы справа. Для наших целей важны несколько потоков: поток UI (который имеет имя вашего пакета или имя UI Thread), `mqt_js` и `mqt_native_modules`. Если вы работаете на Android 5+, нам также важен поток Render Thread.

-   **UI Thread.** Здесь происходит стандартное измерение/раскладка/рисунок android. Имя потока справа будет именем вашего пакета (в моем случае book.adsmanager) или UI Thread. События, которые вы видите в этом потоке, должны выглядеть примерно так и иметь отношение к `Choreographer`, `traversals` и `DispatchUI`:

    ![Пример UI Thread](SystraceUIThreadExample.png)

-   **JS Thread.** Здесь выполняется JavaScript. Имя потока будет либо `mqt_js`, либо `<...>` в зависимости от того, насколько кооперативно ядро вашего устройства. Чтобы определить его, если у него нет имени, ищите такие вещи, как `JSCall`, `Bridge.executeJSCall` и т.д:

    ![Пример JS Thread](SystraceJSThreadExample.png)

-   **Поток нативных модулей.** Здесь выполняются вызовы нативных модулей (например, `UIManager`). Имя потока будет либо `mqt_native_modules`, либо `<...>`. Чтобы определить его в последнем случае, ищите такие вещи, как `NativeCall`, `callJavaModuleMethod` и `onBatchComplete`:

    ![Native Modules Thread Example](SystraceNativeModulesThreadExample.png)

-   **Бонус: поток рендеринга.** Если вы используете Android L (5.0) и выше, в вашем приложении также есть поток рендеринга. Этот поток генерирует фактические команды OpenGL, используемые для рисования вашего пользовательского интерфейса. Имя потока будет либо `RenderThread`, либо `<...>`. Чтобы определить его в последнем случае, ищите такие вещи, как `DrawFrame` и `queueBuffer`:

    ![Пример Render Thread](SystraceRenderThreadExample.png)

## Определение виновника

Плавная анимация должна выглядеть примерно следующим образом:

![Smooth Animation](SystraceWellBehaved.png)

Каждое изменение цвета - это кадр. Помните, что для отображения кадра вся работа нашего пользовательского интерфейса должна быть выполнена к концу этого периода в 16 мс. Обратите внимание, что ни один поток не работает близко к границе кадра. Приложение, выполняющее такой рендеринг, работает со скоростью 60 кадров в секунду.

Однако если бы вы заметили отбивку, вы могли бы увидеть нечто подобное:

![Choppy Animation from JS](SystraceBadJS.png)

Обратите внимание, что поток JS выполняется почти все время, причем через границы кадров! Это приложение не рендерится со скоростью 60 FPS. В этом случае **проблема кроется в JS**.

Вы также можете увидеть нечто подобное:

![Choppy Animation from UI](SystraceBadUI.png)

В этом случае потоки UI и рендеринга - это те потоки, которые работают, пересекая границы кадров. UI, который мы пытаемся рендерить на каждом кадре, требует слишком много работы. В этом случае **проблема заключается в рендеринге нативных представлений**.

На этом этапе у вас будет очень полезная информация для ваших дальнейших действий.

## Решение проблем с JavaScript

Если вы определили проблему с JS, ищите подсказки в конкретном JS, который вы выполняете. В приведенном выше сценарии мы видим, что `RCTEventEmitter` вызывается несколько раз за кадр. Вот увеличенное изображение потока JS из приведенной выше трассировки:

![Слишком много JS](SystraceBadJS2.png)

Это кажется неправильным. Почему он вызывается так часто? Действительно ли это разные события? Ответы на эти вопросы, вероятно, будут зависеть от кода вашего продукта. И во многих случаях вы захотите посмотреть на [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate).

## Решение проблем нативного пользовательского интерфейса

Если вы обнаружили проблему с нативным пользовательским интерфейсом, обычно есть два сценария:

1.  пользовательский интерфейс, который вы пытаетесь отрисовывать каждый кадр, требует слишком много работы от GPU, или
2.  Вы создаете новый пользовательский интерфейс во время анимации/взаимодействия (например, загружаете новый контент во время прокрутки).

### Слишком много работы на GPU

В первом случае вы увидите трассировку, в которой поток UI и/или поток Render Thread выглядят следующим образом:

![Перегруженный GPU](SystraceBadUI.png)

Обратите внимание на большое количество времени, проведенное в `DrawFrame`, которое пересекает границы кадра. Это время, потраченное на ожидание, пока GPU осушит свой буфер команд с предыдущего кадра.

Чтобы уменьшить это, вам следует:

-   изучите возможность использования `renderToHardwareTextureAndroid` для сложного, статичного содержимого, которое анимируется/трансформируется (например, анимация слайдов/альфа-анимации `Navigator`)
-   убедитесь, что вы **не** используете `needsOffscreenAlphaCompositing`, который отключен по умолчанию, поскольку в большинстве случаев он значительно увеличивает покадровую нагрузку на GPU.

### Создание новых представлений в потоке пользовательского интерфейса

Во втором сценарии вы увидите примерно следующее:

![Создание представлений](SystraceBadCreateUI.png)

Заметьте, что сначала поток JS немного думает, затем вы видите некоторую работу, выполненную в потоке нативных модулей, а затем дорогостоящий обход в потоке UI.

Не существует быстрого способа смягчить это, если только вы не сможете отложить создание нового пользовательского интерфейса до окончания взаимодействия или не сможете упростить создаваемый пользовательский интерфейс. Команда react native работает над решением этой проблемы на уровне инфраструктуры, которое позволит создавать и настраивать новый пользовательский интерфейс вне основного потока, позволяя взаимодействию продолжаться плавно.