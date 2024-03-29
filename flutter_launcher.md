# Пишем лаунчер для Android на Flutter

В один из дней мой коллега написал примерно такое сообщение: \
\- Меня тут спросили, можно ли лаунчер для android на flutter написать? Вроде можно.

Этот вопрос не давал мне покоя и я решил разобраться.

---

Вообще, идея написать свой лаунчер стара как мир и существует много туториалов (далее первое, что нашел и более менее приличное):
1. https://stackoverflow.com/questions/4841686/how-to-make-a-launcher
1. https://habr.com/ru/post/265823/
1. http://arnab.ch/blog/2013/08/how-to-write-custom-launcher-app-in-android/

Из интересного можно заметить, что эти все гайды написаны примерно лет десять назад.

Но из гайдов я понял, что **любое** приложение можно заставить считать лаунчером. Осталось проверить. Работает ли метод, которым пользовались лет 10 назад?

И этот метод до сих пор **работает**. Достаточно добавить следующие две строчки в `android/app/src/main/AndroidManifest.xml`

```xml
<application
    ...
    <activity
        ...
        <intent-filter>
            ...
            <category android:name="android.intent.category.HOME" />
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
```

Окей, что мы получим на данном этапе? Херню. Сделав только это получится херня. Почему? Этот манифест не сделает из вашего приложения лаунчер, да, андроид будет считать, что ваше приложение лаунчер, но это явно не все шаги, которые нужно выполнить.

Очевидно, что mvp лаунчера должно  иметь следующие фичи:
1. может показать список всех установленных приложений
1. по нажатию на некое визуальное отображение приложения "запустится" приложение.

Давайте приступим.

## Показываем список установленных приложений

Для того чтобы получить список установленных приложений в android есть класс [PackageManager](https://developer.android.com/reference/android/content/pm/PackageManager), ну чтож - пожалуй напишу плагин для того чтоб вызывать нативные методы. 

Почему я решил писать свой плагин, а не взять готовый?
1. тренировка, я знаю в теории как писать плагины и правил несколько готовых, но не создавал их с нуля;
1. легкий поиск навел меня на два плагина, но мне они не понравились (достаточно сырые и не поддерживаются, в таких случаях логично делать свое решение);
1. свое решение проще кастомизировать и подстраивать под свои хотелки (например, сделать EventChannel в дополнении к MethodChannel).

Кстати, за 10 лет появился [ньюанс](https://developer.android.com/training/package-visibility) поэтому нужно будет добавить в манифест следующие строчки:
```xml
<application
    ...
    <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
    <queries>
        <intent>
            <action android:name="android.intent.action.MAIN" />
        </intent>
    </queries>
```

А теперь...

![Мем как нарисовать сову в два действия](./img/how_paint_owl_mem.jpg)
*Как написать плагин*

Самое неудобное это...
Как же я привык к hot-restart/reload, написание нативных плагинов не про это