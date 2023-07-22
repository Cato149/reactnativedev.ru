---
description: Панель вкладок в нижней части экрана, выполненная в стиле Material-Design, позволяет переключаться между различными маршрутами с помощью анимации
---

# Навигатор по нижним вкладкам Material

Панель вкладок в нижней части экрана, выполненная в стиле Material-Design, позволяет переключаться между различными маршрутами с помощью анимации. Маршруты инициализируются лениво - их экранные компоненты не устанавливаются до тех пор, пока они не будут впервые сфокусированы.

Здесь используется компонент [`BottomNavigation`](https://callstack.github.io/react-native-paper/bottom-navigation.html) из [`react-native-paper`](https://reactnativepaper.com). Если вы [настроите плагин Babel](https://callstack.github.io/react-native-paper/getting-started.html), то он не будет включать всю библиотеку `react-native-paper` в ваш бандл.

![Навигатор по нижним вкладкам Material](material-bottom-tabs.gif)

## Установка

Чтобы использовать этот навигатор, убедитесь, что у вас есть [`@react-navigation/native` и его зависимости (следуйте этому руководству)](getting-started.md), затем установите [`@react-navigation/material-bottom-tabs`](https://github.com/react-navigation/react-navigation/tree/main/packages/material-bottom-tabs):

```bash
npm install @react-navigation/material-bottom-tabs react-native-paper react-native-vector-icons
```

Этот API также требует установки `react-native-vector-icons`! Если вы используете Expo managed workflow, то он будет работать без дополнительных шагов. В противном случае [следуйте этим инструкциям по установке](https://github.com/oblador/react-native-vector-icons#installation).

Чтобы использовать этот навигатор вкладок, импортируйте его из `@react-navigation/material-bottom-tabs`.

## API Определение

!!!note ""

    💡 Если при использовании `createMaterialBottomTabNavigator` вы столкнулись с какими-либо ошибками, пожалуйста, открывайте проблемы на [`react-native-paper`](https://github.com/callstack/react-native-paper), а не в репозитории `react-navigation`!

Чтобы использовать этот навигатор вкладок, импортируйте его из `@react-navigation/material-bottom-tabs`:

```js
import { createMaterialBottomTabNavigator } from '@react-navigation/material-bottom-tabs';

const Tab = createMaterialBottomTabNavigator();

function MyTabs() {
    return (
        <Tab.Navigator>
            <Tab.Screen
                name="Home"
                component={HomeScreen}
            />
            <Tab.Screen
                name="Settings"
                component={SettingsScreen}
            />
        </Tab.Navigator>
    );
}
```

!!!note ""

    Полное руководство по использованию приведено на сайте [Tab Navigation](tab-based-navigation.md)

## RouteConfigs

Объект route configs представляет собой отображение имени маршрута на конфигурацию маршрута.

### Пропсы {#props}

Компонент `Tab.Navigator` принимает следующие параметры:

#### `id`

Необязательный уникальный идентификатор навигатора. Он может быть использован с помощью [`navigation.getParent`](navigation-prop.md#getparent) для ссылки на этот навигатор в дочернем навигаторе.

#### `initialRouteName`

Имя маршрута, которое должно отображаться при первой загрузке навигатора.

#### `screenOptions`

Параметры по умолчанию, используемые для экранов в навигаторе.

#### `backBehavior`

Этот параметр управляет тем, что происходит при вызове `goBack` в навигаторе. Это включает в себя нажатие кнопки "назад" на устройстве или жест "назад" на Android.

Поддерживаются следующие значения:

-   `firstRoute` - возврат на первый экран, заданный в навигаторе (по умолчанию)
-   `initialRoute` - возврат к начальному экрану, переданному в параметре `initialRouteName`, если значение не передано, то по умолчанию возвращается к первому экрану
-   `order` - возврат к экрану, определенному перед сфокусированным экраном
-   `history` - возврат к последнему посещенному экрану в навигаторе; если один и тот же экран посещается несколько раз, то старые записи удаляются из истории
-   `none` - не обрабатывать кнопку "Назад"

#### `shifting`

Если используется стиль сдвига, то значок активной вкладки сдвигается вверх, чтобы показать ярлык, а неактивные вкладки не будут иметь ярлыка.

По умолчанию это значение `true`, если у вас более 3 вкладок. Передайте `shifting={false}`, чтобы явно отключить эту анимацию, или `shifting={true}`, чтобы всегда использовать эту анимацию.

#### `labeled`

Показывать ли ярлыки на вкладках. Если `false`, то будут отображаться только значки.

#### `activeColor`

Пользовательский цвет для значка и ярлыка на активной вкладке.

#### `inactiveColor`

Пользовательский цвет для значка и метки на неактивной вкладке.

#### `barStyle`

Стиль для нижней панели навигации. Здесь можно передать пользовательский цвет фона:

```js
<Tab.Navigator
    initialRouteName="Home"
    activeColor="#f0edf6"
    inactiveColor="#3e2465"
    barStyle={{ backgroundColor: '#694fad' }}
>
    {/* ... */}
</Tab.Navigator>
```

Если на Android используется полупрозрачная панель навигации, то здесь также можно задать нижнюю подложку:

```js
<Tab.Navigator
    initialRouteName="Home"
    activeColor="#f0edf6"
    inactiveColor="#3e2465"
    barStyle={{ paddingBottom: 48 }}
>
    {/* ... */}
</Tab.Navigator>
```

### Опции {#options}

Для настройки экранов в навигаторе можно использовать следующие [options](screen-options.md):

#### `title`

Общий заголовок, который может использоваться в качестве запасного варианта для `headerTitle` и `tabBarLabel`.

#### `tabBarIcon`

Функция, которая при задании `{ focused: boolean, color: string }` возвращает узел React.Node для отображения в панели вкладок.

#### `tabBarColor`

Цвет для панели вкладок, когда активна вкладка, соответствующая экрану. Используется для эффекта пульсации. Поддерживается только в том случае, если `shifting` имеет значение `true`.

#### `tabBarLabel`

Строка заголовка вкладки, отображаемая на панели вкладок. При неопределенности используется сцена `title`. Для скрытия см. опцию `labeled` в предыдущем разделе.

#### `tabBarBadge`

Значок, который будет отображаться на значке вкладки, может быть `true` для отображения точки, `string` или `number` для отображения текста.

#### `tabBarAccessibilityLabel`

Метка доступности для кнопки вкладки. Она считывается программой чтения с экрана, когда пользователь нажимает кнопку вкладки. Рекомендуется установить это значение, если у вас нет метки для вкладки.

#### `tabBarTestID`

ID для размещения этой кнопки вкладки в тестах.

### События {#events}

Навигатор может [выдавать события](navigation-events.md) на определенные действия. Поддерживаются следующие события:

#### `tabPress`

Это событие возникает, когда пользователь нажимает кнопку вкладки для текущего экрана на панели вкладок. По умолчанию нажатие кнопки вкладки выполняет несколько действий:

-   Если вкладка не сфокусирована, то нажатие кнопки вкладки фокусирует эту вкладку.
-   Если вкладка уже сфокусирована:
    -   Если экран для вкладки отображается в виде прокрутки, то можно использовать [`useScrollToTop`](use-scroll-to-top.md) для прокрутки в верхнее положение.
    -   Если на экране вкладки отображается стековый навигатор, то для стека выполняется действие `popToTop`.

Чтобы предотвратить поведение по умолчанию, можно вызвать `event.preventDefault`:

```js
React.useEffect(() => {
    const unsubscribe = navigation.addListener(
        'tabPress',
        (e) => {
            // Prevent default behavior

            e.preventDefault();
            // Do something manually
            // ...
        }
    );

    return unsubscribe;
}, [navigation]);
```

### Хелперы {#helpers}

Навигатор вкладок добавляет в реквизит навигации следующие методы:

#### `jumpTo`

Осуществляет переход к существующему экрану в навигаторе вкладок. Метод принимает следующие аргументы:

-   `name` - _string_ - Имя маршрута, на который необходимо перейти.
-   `params` - _object_ - Параметры экрана для передачи маршруту назначения.

```js
navigation.jumpTo('Profile', { name: 'Michaś' });
```

## Пример {#example}

```js
import { createMaterialBottomTabNavigator } from '@react-navigation/material-bottom-tabs';
import MaterialCommunityIcons from 'react-native-vector-icons/MaterialCommunityIcons';

const Tab = createMaterialBottomTabNavigator();

function MyTabs() {
    return (
        <Tab.Navigator
            initialRouteName="Feed"
            activeColor="#e91e63"
            barStyle={{ backgroundColor: 'tomato' }}
        >
            <Tab.Screen
                name="Feed"
                component={Feed}
                options={{
                    tabBarLabel: 'Home',
                    tabBarIcon: ({ color }) => (
                        <MaterialCommunityIcons
                            name="home"
                            color={color}
                            size={26}
                        />
                    ),
                }}
            />
            <Tab.Screen
                name="Notifications"
                component={Notifications}
                options={{
                    tabBarLabel: 'Updates',
                    tabBarIcon: ({ color }) => (
                        <MaterialCommunityIcons
                            name="bell"
                            color={color}
                            size={26}
                        />
                    ),
                }}
            />
            <Tab.Screen
                name="Profile"
                component={Profile}
                options={{
                    tabBarLabel: 'Profile',
                    tabBarIcon: ({ color }) => (
                        <MaterialCommunityIcons
                            name="account"
                            color={color}
                            size={26}
                        />
                    ),
                }}
            />
        </Tab.Navigator>
    );
}
```

## Использование с `react-native-paper` (опционально)

Вы можете использовать поддержку тематики в `react-native-paper` для настройки нижней навигации материала, обернув свое приложение в [`Provider` из `react-native-paper`](https://callstack.github.io/react-native-paper/getting-started.html). Частым случаем использования этой функции может быть настройка цвета фона для экранов, когда ваше приложение имеет темную тему. Следуйте [инструкциям в документации по `react-native-paper`](https://callstack.github.io/react-native-paper/theming.html), чтобы узнать, как настроить тему.

## Ссылки

-   [Material Bottom Tabs Navigator](https://reactnavigation.org/docs/material-bottom-tab-navigator/)
