# Знакомство с Expo

Expo - это фреймворк для React Native. С ним можно делать приложения под android, ios и web на одном коде. Expo CLI нужен чтобы создавать проект, запускать, собирать и т.д.

Сайт: https://docs.expo.dev/

## Create a project

Нужен Node.js (LTS). Проект создается командой:

```
npx create-expo-app@latest
```

- по умолчанию берется шаблон default, в нем уже есть примеры кода и Expo Router (навигация)
- другой шаблон можно выбрать через `--template`, например `blank` - пустой проект без навигации
- версия SDK может быть разная, если запускаю на телефоне через Expo Go, то SDK проекта должен подходить к версии Expo Go из магазина

## Set up your environment

Запускать приложение можно:

- на телефоне - ставим приложение Expo Go и сканируем QR-код, самый быстрый вариант
- на эмуляторе android - нужен Android Studio
- на симуляторе ios - нужен Xcode, работает только на macOS

Есть два способа запуска:

- Expo Go - готовая песочница, работают только те библиотеки которые в нее уже встроены
- development build - своя сборка приложения, можно подключать любые нативные библиотеки

```
npx expo run:android
npx expo run:ios
npx expo start --dev-client
```

## Start developing

Запуск сервера:

```
npx expo start
```

В терминале появится QR-код и кнопки:

- a - открыть на android
- i - открыть на ios
- w - открыть в браузере
- r - перезагрузить
- m - меню разработчика

Код лежит в папке `app`, каждый файл там это экран (file-based routing). Главный экран - `app/index.tsx`. Работает fast refresh, после сохранения файла изменения сразу видны в приложении.

Команда `npm run reset-project` переносит стартовый код в `app-example` и создает пустую папку `app`, чтобы начать с чистого проекта.

Структура проекта примерно такая:

```
app/          - экраны
assets/       - картинки, шрифты
components/   - свои компоненты
app.json      - настройки приложения
package.json  - зависимости
```

## Next steps

Что делать дальше:

- пройти Expo tutorial
- почитать про Expo Router
- посмотреть справочник Expo SDK (камера, файлы, уведомления и т.д.)
- EAS (Expo Application Services) - облачные сервисы для сборки, обновлений и публикации приложений
- если вопросы - сообщество в Discord и на GitHub

## Tools for development

- Expo CLI (`npx expo ...`) - запуск, сборка, установка библиотек
- Expo Go - тестирование на устройстве
- development builds - тестирование с любыми нативными библиотеками
- `npx expo install <пакет>` - ставит версию библиотеки, которая подходит к текущему SDK
- `npx expo-doctor` - проверяет проект на проблемы
- React Native DevTools - отладка
- EAS CLI - сборка и публикация
- VS Code - редактор

Основные команды:

```
npx expo start
npx expo install expo-image
npx expo prebuild
npx expo run:android
npx expo run:ios
```

## Navigation

Навигация в Expo сделана через Expo Router (основан на React Navigation). Маршруты определяются файлами в папке `app`:

- `app/index.tsx` - главный экран `/`
- `app/about.tsx` - экран `/about`
- `app/_layout.tsx` - общий layout для экранов в папке
- `app/[id].tsx` - динамический маршрут
- `app/(tabs)/` - группа, скобки в адрес не попадают
- `app/+not-found.tsx` - страница "не найдено"

Типы навигаторов: Stack (стопка экранов), Tabs (вкладки), Drawer (боковое меню).

Переход между экранами:

```tsx
import { Link, router } from 'expo-router';

<Link href="/about">About</Link>

router.push('/about');
```

Параметры динамического маршрута:

```tsx
import { useLocalSearchParams } from 'expo-router';

const { id } = useLocalSearchParams();
```

# Expo tutorial

Источник: https://docs.expo.dev/tutorial/introduction/

В туториале делается приложение StickerSmash: выбираем фото из галереи, добавляем на него стикер (эмодзи), двигаем его жестами и сохраняем результат картинкой. Работает на android, ios и web. Всего 9 глав, примерно 2 часа.

## Introduction

Вводная глава. Туториал построен на практике: больше делаем, меньше читаем теорию. Исходный код есть на GitHub (expo/examples, папка stickersmash).

## Create your first app

Создаем проект и запускаем:

```
npx create-expo-app@latest StickerSmash
cd StickerSmash
npx expo start
```

Потом `npm run reset-project`, чтобы убрать стартовый код, и правим `app/index.tsx`:

```tsx
import { Text, View, StyleSheet } from 'react-native';

export default function Index() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Home screen</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#25292e',
    alignItems: 'center',
    justifyContent: 'center',
  },
  text: {
    color: '#fff',
  },
});
```

View - контейнер, Text - текст, стили задаются через StyleSheet.create().

## Add navigation

Делаем две вкладки: Home и About (через Expo Router).

- создаем файлы `app/(tabs)/index.tsx`, `app/(tabs)/about.tsx` и `app/+not-found.tsx`
- `app/_layout.tsx` - корневой layout со Stack
- `app/(tabs)/_layout.tsx` - layout с компонентом Tabs, для каждой вкладки задаем title и иконку (Ionicons из @expo/vector-icons)
- в `+not-found.tsx` выводим сообщение и ссылку Link на главный экран

```tsx
import { Tabs } from 'expo-router';

export default function TabLayout() {
  return (
    <Tabs screenOptions={{ tabBarActiveTintColor: '#ffd33d' }}>
      <Tabs.Screen name="index" options={{ title: 'Home' }} />
      <Tabs.Screen name="about" options={{ title: 'About' }} />
    </Tabs>
  );
}
```

## Build a screen

Делаем главный экран. Разбиваем макет на компоненты и верстаем через flexbox.

- ImageViewer - показывает картинку, используется Image из `expo-image`
- Button - своя кнопка (Pressable + Text)
- картинки лежат в `assets/images`, подключаются через `require()`
- на экране картинка сверху (flex: 1), кнопки снизу

```
npx expo install expo-image
```

## Use an image picker

Выбор фото из галереи делается через библиотеку `expo-image-picker`.

```
npx expo install expo-image-picker
```

Функция `launchImageLibraryAsync()` открывает системный выбор картинки:

```tsx
import * as ImagePicker from 'expo-image-picker';

const pickImageAsync = async () => {
  const result = await ImagePicker.launchImageLibraryAsync({
    allowsEditing: true,
    quality: 1,
  });

  if (!result.canceled) {
    setSelectedImage(result.assets[0].uri);
  } else {
    alert('Вы не выбрали изображение');
  }
};
```

Адрес выбранной картинки хранится в состоянии (useState) и передается в ImageViewer.

## Create a modal

Делаем окно выбора стикера.

- CircleButton - круглая кнопка "+", IconButton - кнопка с иконкой и подписью (Reset, Save)
- EmojiPicker - модальное окно на компоненте Modal (animationType="slide", transparent, управляется через isVisible)
- EmojiList - список эмодзи на FlatList (data, renderItem, horizontal)
- EmojiSticker - показывает выбранный стикер поверх фото

В родительском компоненте храним `isModalVisible` и функции `onAddSticker`, `onModalClose`, `onReset`.

## Add gestures

Добавляем жесты, чтобы стикер можно было двигать и увеличивать. Нужны библиотеки react-native-gesture-handler и react-native-reanimated.

```
npx expo install react-native-gesture-handler react-native-reanimated
```

- все приложение оборачиваем в GestureHandlerRootView
- двойной тап (увеличить/уменьшить стикер) - `Gesture.Tap().numberOfTaps(2)`
- перетаскивание - `Gesture.Pan()`, значения translateX и translateY через useSharedValue
- анимированные стили - useAnimatedStyle, плавность через withSpring
- компоненты для анимации - Animated.View и Animated.Image

## Take a screenshot

Сохраняем готовую картинку (фото + стикер) в галерею.

```
npx expo install react-native-view-shot expo-media-library
```

- react-native-view-shot - функция captureRef() делает снимок нужного компонента
- expo-media-library - saveToLibraryAsync(uri) сохраняет файл в галерею, перед этим надо запросить разрешение (usePermissions и requestPermission)
- на web captureRef не работает, там используется библиотека dom-to-image

## Handle platform differences

На android, ios и web приложение ведет себя по-разному, это нужно учитывать.

- платформу можно узнать через `Platform.OS` ('android', 'ios', 'web')
- скриншот: на мобильных captureRef, на web dom-to-image
- на web другие размеры окна и нет жестов касания как на телефоне, поэтому для web правим стили отдельно

```tsx
import { Platform } from 'react-native';

if (Platform.OS !== 'web') {
  // android и ios
} else {
  // web
}
```

## Configure status bar, splash screen and app icon

Последние настройки внешнего вида.

- status bar - библиотека expo-status-bar, компонент `<StatusBar style="light" />`
- splash screen (экран загрузки) - настраивается в app.json через плагин expo-splash-screen (картинка, цвет фона)
- иконка приложения - путь указывается в app.json в поле icon, для android еще adaptiveIcon

app.json - главный файл настроек приложения: название, версия, иконки, плагины.

```json
{
  "expo": {
    "name": "StickerSmash",
    "icon": "./assets/images/icon.png"
  }
}
```

## Learning resources

Что смотреть дальше: документация Expo, документация React Native, туториалы по EAS, сообщество Expo (Discord, GitHub).
