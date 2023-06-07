# BackHandler

API Backhandler обнаруживает нажатия аппаратных кнопок для навигации назад, позволяет регистрировать слушателей событий для действия системы "Назад" и контролировать реакцию вашего приложения. Он предназначен только для Android.

Подписки на события вызываются в обратном порядке (т.е. последняя зарегистрированная подписка вызывается первой).

-   **Если одна подписка возвращает `true`,** то подписки, зарегистрированные ранее, не будут вызваны.
-   **Если ни одна подписка не возвращает `true` или ни одна из них не зарегистрирована,** то для выхода из приложения программно вызывается стандартная функциональность кнопки "Назад".

!!!warning "Предупреждение для пользователей модалов"

    Если ваше приложение показывает открытый `Modal`, `BackHandler` не будет публиковать никаких событий ([см. `Modal` docs](modal#onrequestclose)).

## Паттерн

```tsx
BackHandler.addEventListener(
    'hardwareBackPress',
    function () {
        /**
         * this.onMainScreen and this.goBack are just examples,
         * you need to use your own implementation here.
         *
         * Typically you would use the navigator here to go to the last state.
         */

        if (!this.onMainScreen()) {
            this.goBack();
            /**
             * When true is returned the event will not be bubbled up
             * & no other back action will execute
             */
            return true;
        }
        /**
         * Returning false will let the event to bubble up & let other event listeners
         * or the system's default back action to be executed.
         */
        return false;
    }
);
```

## Пример

В следующем примере реализован сценарий, в котором вы подтверждаете, хочет ли пользователь выйти из приложения:

<div data-snack-id="@bndby/backhandler" data-snack-platform="web" data-snack-preview="true" data-snack-theme="light" style="overflow:hidden;background:#F9F9F9;border:1px solid var(--color-border);border-radius:4px;height:505px;width:100%"></div>

`BackHandler.addEventListener` создает слушателя событий и возвращает объект `NativeEventSubscription`, который должен быть очищен с помощью метода `NativeEventSubscription.remove`.

## Использование с React Navigation

Если вы используете React Navigation для навигации по разным экранам, вы можете следовать их руководству [Custom Android back button behavior](https://reactnavigation.org/docs/custom-android-back-button-handling/).

## Хук для Backhandler

[React Native Hooks](https://github.com/react-native-community/hooks#usebackhandler) имеет хороший хук `useBackHandler`, который упростит процесс настройки слушателей событий.

## Методы

### `addEventListener()`

```tsx
static addEventListener(
  eventName: BackPressEventName,
  handler: () => boolean | null | undefined,
): NativeEventSubscription;
```

### `exitApp()`

```tsx
static exitApp();
```

### `removeEventListener()`

```tsx
static removeEventListener(
  eventName: BackPressEventName,
  handler: () => boolean | null | undefined,
);
```