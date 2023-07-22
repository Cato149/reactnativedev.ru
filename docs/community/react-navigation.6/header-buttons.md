---
description: В статье описывается работа и настройки хедера и кнопок
---

# Кнопки заголовков

Теперь, когда мы знаем, как настроить внешний вид наших заголовков, давайте сделаем их разумными! На самом деле, возможно, это амбициозно, давайте просто сделаем их способными реагировать на наши прикосновения очень четко определенными способами.

## Добавление кнопки в заголовок

Наиболее распространенным способом взаимодействия с заголовком является нажатие на кнопку слева или справа от заголовка. Давайте добавим кнопку в правую часть заголовка (одно из самых труднодоступных мест на всем экране, в зависимости от размера пальца и телефона, но также и обычное место для размещения кнопок).

```js
function StackScreen() {
    return (
        <Stack.Navigator>
            <Stack.Screen
                name="Home"
                component={HomeScreen}
                options={{
                    headerTitle: (props) => (
                        <LogoTitle {...props} />
                    ),
                    headerRight: () => (
                        <Button
                            onPress={() =>
                                alert('This is a button!')
                            }
                            title="Info"
                            color="#fff"
                        />
                    ),
                }}
            />
        </Stack.Navigator>
    );
}
```

Когда мы определяем кнопку таким образом, переменная `this` в `options` не является экземпляром `HomeScreen`, поэтому вы не можете вызвать `setState` или какие-либо методы экземпляра. Это очень важно, поскольку очень часто требуется, чтобы кнопки в заголовке взаимодействовали с экраном, которому принадлежит заголовок. Поэтому далее мы рассмотрим, как это сделать.

> 💡 Обратите внимание, что существует разработанная сообществом библиотека для отрисовки кнопок в заголовке с правильным стилем: [react-navigation-header-buttons](https://github.com/vonovak/react-navigation-header-buttons).

## Взаимодействие заголовка с его экранным компонентом

В некоторых случаях компоненты в заголовке должны взаимодействовать с компонентом экрана. Для такого случая нам необходимо использовать `navigation.setOptions` для обновления опций. Используя `navigation.setOptions` внутри компонента экрана, мы получаем доступ к его реквизитам, состоянию, контексту и т.д.

```js
function StackScreen() {
    return (
        <Stack.Navigator>
            <Stack.Screen
                name="Home"
                component={HomeScreen}
                options={({ navigation, route }) => ({
                    headerTitle: (props) => (
                        <LogoTitle {...props} />
                    ),
                    // Add a placeholder button without the `onPress` to avoid flicker
                    headerRight: () => (
                        <Button title="Update count" />
                    ),
                })}
            />
        </Stack.Navigator>
    );
}

function HomeScreen({ navigation }) {
    const [count, setCount] = React.useState(0);

    React.useEffect(() => {
        // Use `setOptions` to update the button that we previously specified
        // Now the button includes an `onPress` handler to update the count
        navigation.setOptions({
            headerRight: () => (
                <Button
                    onPress={() => setCount((c) => c + 1)}
                    title="Update count"
                />
            ),
        });
    }, [navigation]);

    return <Text>Count: {count}</Text>;
}
```

Здесь мы обновляем `headerRight` с помощью кнопки с обработчиком `onPress`, который имеет доступ к состоянию компонента и может его обновлять.

## Настройка кнопки "Назад

Функция `createNativeStackNavigator` предоставляет умолчания для кнопки "Назад" в зависимости от платформы. На iOS это включает в себя метку рядом с кнопкой, которая показывает заголовок предыдущего экрана, если он помещается в свободное пространство, в противном случае на ней написано "Назад".

Вы можете изменить поведение метки с помощью команды `headerBackTitle` и придать ей стиль с помощью команды `headerBackTitleStyle` ([подробнее](native-stack-navigator.md#headerbacktitle)).

Для настройки изображения кнопки "Назад" можно использовать `headerBackImageSource` ([подробнее](native-stack-navigator.md#headerbackimagesource)).

## Переопределение кнопки "Назад

Кнопка "Назад" будет автоматически отображаться в стековом навигаторе при любой возможности возврата пользователя с текущего экрана &mdash; другими словами, кнопка "Назад" будет отображаться при наличии в стеке более одного экрана.

В общем случае это то, что нужно. Но возможно, что в некоторых обстоятельствах вы захотите настроить кнопку "Назад" больше, чем это можно сделать с помощью вышеупомянутых опций. В этом случае вы можете установить опцию `headerLeft` для React-элемента, который будет отрисовываться, как мы это сделали с `headerRight`. В качестве альтернативы опция `headerLeft` также принимает React-компонент, который может быть использован, например, для переопределения поведения onPress кнопки "Назад". Подробнее об этом можно прочитать в [api reference](native-stack-navigator.md#headerleft).

## Резюме

-   Вы можете настроить кнопки в заголовке через свойства `headerLeft` и `headerRight` в `options`.
-   Кнопка "Назад" полностью настраивается с помощью `headerLeft`, но если вы хотите изменить только заголовок или изображение, то для этого есть другие `опции` &mdash; `headerBackTitle`, `headerBackTitleStyle` и `headerBackImageSource`.
-   Для доступа к объектам `navigation` и `route` можно использовать обратный вызов реквизита options.

## Ссылки

-   [Header buttons](https://reactnavigation.org/docs/header-buttons)
