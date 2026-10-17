# Лекционное задание № 1 — Expo tutorial

Источник: https://docs.expo.dev/tutorial/introduction/

Конспект глав 1–4: Introduction, Create your first app, Add navigation, Build a screen.

---

## 1. Introduction

**О чём туториал.** Строим приложение **StickerSmash** на Expo, которое работает на Android, iOS и в вебе из одной кодовой базы. Проходится за ~2 часа, разбит на 9 глав.

**Чему научимся:**

- создать приложение из шаблона по умолчанию с TypeScript;
- сделать нижнюю панель вкладок из двух экранов через Expo Router;
- разобрать макет экрана и сверстать его с помощью flexbox;
- выбирать изображение из медиатеки через системный UI платформы;
- сделать модальное окно со стикерами (`<Modal>` и `<FlatList>`);
- добавить жесты для взаимодействия со стикером;
- сделать скриншот и сохранить его на диск (сторонние библиотеки);
- учесть различия между Android, iOS и web;
- настроить status bar, splash screen и иконку приложения.

**Как пользоваться.** Принцип «learning by doing»: больше делаем, чем читаем. Изменённый код в примерах подсвечивается. Готовый код доступен на GitHub: https://github.com/expo/examples/tree/master/stickersmash

---

## 2. Create your first app

### Что нужно заранее

- **Expo Go** на реальном Android/iOS-устройстве (на iOS ещё нужен бесплатный Expo-аккаунт и вход в нём и в Expo CLI, и в Expo Go);
- **Node.js (LTS)**;
- редактор кода (например, VS Code);
- компьютер с терминалом (macOS, Linux, Windows с PowerShell или WSL2);
- базовое знание TypeScript и React.

### Создание проекта

```sh
npx create-expo-app@latest StickerSmash
# в списке выбрать SDK 57
cd StickerSmash
```

`create-expo-app` — консольная утилита, создающая новый React Native проект. Берётся шаблон **default**: в нём уже есть Expo Router, TypeScript, Expo CLI, пример навигации на вкладках, и он сразу настроен под Android, iOS и web.

### Ассеты

Скачать архив с картинками для туториала и заменить стандартные файлы в папке `assets/images`.

### Скрипт reset-project

```sh
npm run reset-project
```

Удаляет шаблонный код: в `src/app` остаются только `index.tsx` и `_layout.tsx`, а старые файлы (`components`, `constants`, `hooks`) переносятся в папку `example` — её можно удалить.

### Запуск

```sh
npx expo start
```

1. Запускается dev-сервер, в терминале появляется QR-код.
2. Android: Expo Go → **Scan QR code**. iOS: обычная камера.
3. Для веба нажать `w` в терминале.

### Редактирование экрана `src/app/index.tsx`

Это точка входа приложения. Используются базовые компоненты React Native: `<View>` (контейнер) и `<Text>` (текст). Стили задаются **JS-объектами** через `StyleSheet.create`, а не CSS, но свойства похожи на CSS. Цвета — как в вебе: hex, `rgba`, `hsl`, именованные.

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

После сохранения изменения сразу применяются на всех подключённых устройствах (Fast Refresh).

**Синяя шестерёнка** — кнопка Tools, открывает Developer menu (перезагрузка, Fast Refresh и т. д.). Показывается только при разработке, в продакшн-сборку не попадает.

**Итог главы:** создали проект, поработали с базовыми компонентами React Native и готовы разрабатывать StickerSmash.

---

## 3. Add navigation

### Основы Expo Router

Expo Router — файловая маршрутизация для React Native и веба: каждый файл в папке `app` становится экраном в приложении и страницей на вебе.

- **Папка `app`** (в шаблоне это `src/app`) — только маршруты и layout'ы.
- **Root layout** — файл `src/app/_layout.tsx`, общий UI (заголовки, tab bar) для всех маршрутов.
- **`index.tsx`** соответствует родительскому пути и не добавляет сегмент (`src/app/index.tsx` → маршрут `/`).
- Файл маршрута экспортирует React-компонент **по умолчанию** (`.js`, `.jsx`, `.ts`, `.tsx`).
- Навигация одинаковая на Android, iOS и web.

### Новый экран в стеке

Файл `src/app/about.tsx` (маршрут `/about`):

```tsx
import { Text, View, StyleSheet } from 'react-native';

export default function AboutScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>About screen</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#25292e',
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    color: '#fff',
  },
});
```

В `_layout.tsx` регистрируем экраны через `<Stack.Screen />` и задаём заголовки через `options`:

```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="about" options={{ title: 'About' }} />
    </Stack>
  );
}
```

**Stack** — основа навигации между экранами: новый экран «накладывается» поверх текущего (на Android анимируется сверху, на iOS — справа).

### Переход между экранами — `Link`

`Link` из `expo-router` рендерит `<Text>` и принимает проп `href`. Принимает те же стили, что и `<Text>`.

```tsx
import { Link } from 'expo-router';

<Link href="/about" style={styles.button}>
  Go to About screen
</Link>
```

### Маршрут not-found

Файл `src/app/+not-found.tsx` показывается, если маршрут не существует (вместо падения приложения на мобильных или 404 на вебе). Внутри — `Stack.Screen` с заголовком и `Link` на `/`.

```tsx
import { View, StyleSheet } from 'react-native';
import { Link, Stack } from 'expo-router';

export default function NotFoundScreen() {
  return (
    <>
      <Stack.Screen options={{ title: 'Oops! Not Found' }} />
      <View style={styles.container}>
        <Link href="/" style={styles.button}>
          Go back to Home screen!
        </Link>
      </View>
    </>
  );
}
```

Проверить можно в браузере: `http://localhost:8081/123`.

### Нижняя панель вкладок (tabs)

1. В `src/app` создать папку **`(tabs)`** — круглые скобки означают группу маршрутов, которая отображается как нижний tab bar.
2. Создать `(tabs)/_layout.tsx` — layout вкладок (отдельный от root layout).
3. Перенести `index.tsx` и `about.tsx` внутрь `(tabs)`.

Структура:

```
src/app
├── _layout.tsx          Root layout
├── +not-found.tsx       любой несуществующий маршрут
└── (tabs)
    ├── _layout.tsx      Tab layout
    ├── index.tsx        маршрут '/'
    └── about.tsx        маршрут '/about'
```

Root layout (Stack остаётся, чтобы `+not-found` показывался поверх любых вложенных навигаторов):

```tsx
<Stack>
  <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
</Stack>
```

Tab layout:

```tsx
import { Tabs } from 'expo-router';

export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen name="index" options={{ title: 'Home' }} />
      <Tabs.Screen name="about" options={{ title: 'About' }} />
    </Tabs>
  );
}
```

### Иконки и оформление вкладок

Установка библиотеки иконок (сервер остановить через `Ctrl + C`):

```sh
npx expo install @expo/vector-icons
```

Затем снова `npx expo start`.

В `(tabs)/_layout.tsx`:

- `tabBarIcon` — функция с параметрами `focused` и `color`, возвращает иконку (`Ionicons`);
- `screenOptions.tabBarActiveTintColor` — цвет активной вкладки (`#ffd33d`);
- `headerStyle.backgroundColor`, `headerShadowVisible: false`, `headerTintColor` — оформление заголовка;
- `tabBarStyle.backgroundColor` — фон нижней панели.

```tsx
import { Tabs } from 'expo-router';
import Ionicons from '@expo/vector-icons/Ionicons';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#ffd33d',
        headerStyle: { backgroundColor: '#25292e' },
        headerShadowVisible: false,
        headerTintColor: '#fff',
        tabBarStyle: { backgroundColor: '#25292e' },
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color, focused }) => (
            <Ionicons name={focused ? 'home-sharp' : 'home-outline'} color={color} size={24} />
          ),
        }}
      />
      <Tabs.Screen
        name="about"
        options={{
          title: 'About',
          tabBarIcon: ({ color, focused }) => (
            <Ionicons
              name={focused ? 'information-circle' : 'information-circle-outline'}
              color={color}
              size={24}
            />
          ),
        }}
      />
    </Tabs>
  );
}
```

**Итог главы:** добавили stack-навигатор и нижние вкладки.

---

## 4. Build a screen

Строим первый экран StickerSmash: большое изображение по центру и две кнопки в нижней части. Первая кнопка — выбрать фото с устройства, вторая — продолжить с картинкой по умолчанию.

### Разбор экрана на элементы

- большая картинка по центру;
- две кнопки в нижней половине экрана;
- первая кнопка составная: родитель с жёлтой рамкой, внутри в ряд иконка и текст.

### Отображение картинки (`expo-image`)

Компонент `<Image>` из `expo-image` (уже есть в шаблоне) работает на всех платформах. В `source` передаётся либо статический ассет (`require(...)`), либо URL (`{ uri: ... }`).

```tsx
import { Image } from 'expo-image';

const PlaceholderImage = require('@/assets/images/background-image.png');
```

### Разделение на компоненты

Кастомные компоненты лежат в `src/components`, а не в `src/app`, потому что каждый файл в `src/app` — это layout или маршрут.

`src/components/image-viewer.tsx`:

```tsx
import { ImageSourcePropType, StyleSheet } from 'react-native';
import { Image } from 'expo-image';

type Props = {
  imgSource: ImageSourcePropType;
};

export default function ImageViewer({ imgSource }: Props) {
  return <Image source={imgSource} style={styles.image} />;
}

const styles = StyleSheet.create({
  image: {
    width: 320,
    height: 440,
    borderRadius: 18,
  },
});
```

**Что такое `@` в импорте:** псевдоним пути (path alias), чтобы не писать относительные пути вроде `../../`. Expo CLI настраивает его автоматически в `tsconfig.json`.

### Кнопки на `Pressable`

`<Pressable>` — рекомендуемый компонент для обработки нажатий: умеет одиночный тап, долгое нажатие, отдельные события нажатия и отпускания и т. д.

`src/components/button.tsx` (базовая версия):

```tsx
import { StyleSheet, View, Pressable, Text } from 'react-native';

type Props = {
  label: string;
};

export default function Button({ label }: Props) {
  return (
    <View style={styles.buttonContainer}>
      <Pressable style={styles.button} onPress={() => alert('You pressed a button.')}>
        <Text style={styles.buttonLabel}>{label}</Text>
      </Pressable>
    </View>
  );
}
```

При нажатии вызывается `alert()` через проп `onPress`.

### Тема `primary` и иконка

Добавляем необязательный проп `theme?: 'primary'`. Для primary-кнопки:

- **inline-стили** (объект прямо в `style`) переопределяют стили из `StyleSheet.create()`;
- белый фон (`backgroundColor: '#fff'`) задаётся inline, чтобы не менять вторую кнопку;
- жёлтая рамка `#ffd33d`, `borderWidth: 4`, `borderRadius: 18`;
- иконка `FontAwesome` (`picture-o`) из `@expo/vector-icons` перед текстом.

```tsx
import FontAwesome from '@expo/vector-icons/FontAwesome';

type Props = {
  label: string;
  theme?: 'primary';
};

export default function Button({ label, theme }: Props) {
  if (theme === 'primary') {
    return (
      <View
        style={[
          styles.buttonContainer,
          { borderWidth: 4, borderColor: '#ffd33d', borderRadius: 18 },
        ]}>
        <Pressable
          style={[styles.button, { backgroundColor: '#fff' }]}
          onPress={() => alert('You pressed a button.')}>
          <FontAwesome name="picture-o" size={18} color="#25292e" style={styles.buttonIcon} />
          <Text style={[styles.buttonLabel, { color: '#25292e' }]}>{label}</Text>
        </Pressable>
      </View>
    );
  }

  // обычная кнопка (как в базовой версии)
}
```

### Сборка экрана `src/app/(tabs)/index.tsx`

```tsx
import { View, StyleSheet } from 'react-native';

import Button from '@/components/button';
import ImageViewer from '@/components/image-viewer';

const PlaceholderImage = require('@/assets/images/background-image.png');

export default function Index() {
  return (
    <View style={styles.container}>
      <View style={styles.imageContainer}>
        <ImageViewer imgSource={PlaceholderImage} />
      </View>
      <View style={styles.footerContainer}>
        <Button theme="primary" label="Choose a photo" />
        <Button label="Use this photo" />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#25292e',
    alignItems: 'center',
  },
  imageContainer: {
    flex: 1,
  },
  footerContainer: {
    flex: 1 / 3,
    alignItems: 'center',
  },
});
```

**Итог главы:** сверстали первый экран приложения; в следующей главе добавим выбор изображения из медиатеки.
