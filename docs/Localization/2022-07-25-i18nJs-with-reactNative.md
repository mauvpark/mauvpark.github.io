---
layout: post
title: How to manage i18n-js with React Native
date: 2022-07-25 23:56:00 +0900
parent: Localization
categories: localization, i18n, react-native, expo, missing, translation
nav_order: 1
---

*Written at 2022-07-25 23:56*
*Modified at 2022-07-26 16:00*

# How to manage i18n-js with React Native
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Introduction

[i18n-js](https://github.com/fnando/i18n-js) is a simple tool that can manage localization in your app. It could be used when you want to publish your app globally by applying each country's language.

Expo introduces how to use this library [as follows](https://docs.expo.dev/versions/latest/sdk/localization/).

```js
import * as React from 'react';
import { View, StyleSheet, Text } from 'react-native';
import * as Localization from 'expo-localization';
import i18n from 'i18n-js';

// Set the key-value pairs for the different languages you want to support.
i18n.translations = {
  en: { welcome: 'Hello', name: 'Charlie' },
  ja: { welcome: 'こんにちは' },
};
// Set the locale once at the beginning of your app.
i18n.locale = Localization.locale;
// When a value is missing from a language it'll fallback to another language with the key present.
i18n.fallbacks = true;

export default App => {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>
        {i18n.t('welcome')} {i18n.t('name')}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({ ... }); 
```

## Missing translation

I read Expo's doc, and I coded it as follows.

```js
// Home.js
i18n.locale = Localization.locale;
i18n.fallbacks = true;

export const Home = ({navigation}) => {
    i18n.translations = {
      en: { welcome: 'Hello', name: 'Charlie' },
      ko: { welcome: '안녕하세요' },
    };

    return (
        <View>
            <Text>{i18n.t("welcome")}</Text>
            <Button title="Go to Settings" onPress={() => navigation.navigate("Settings")} />
        </View>
    );
}
```

```js
// Settings.js
export const Settings = ({navigation}) => {
    i18n.translations = {
      en: { settings: 'Settings' },
      ko: { settings: '설정' },
    };

    return (
        <View>
            <Text>{i18n.t("settings")}</Text>
            <Button title="Go to Home" onPress={() => navigation.navigate("Home")} />
        </View>
    );
}
```

There are two different translation scopes as you could see. Would it work well? The answer is **_not really_**. When you go back to the `Home` screen, you could lose your scope `i18n.t("welcome")`.

> It means that you should update your scope whenever your page is changed.

Okay, why this is happening? Because the **i18n-js** library works like a singleton, if you set your translation scope on a page one time, then it keeps tracking that scope. Likewise, if you move to `Settings` from `Home`, i18n's scope will be changed to the `Settings` screen's translation. if you go back, if it does not render again, we can not sure whether its scope changed or not.

> So `missing en.something translation` means your scope is not focusing properly.

## Solution 1: Use one scope

Simply, you could declare only one scope when an app starts.

```js
// App.js
i18n.locale = Localization.locale;
i18n.fallbacks = true;
i18n.translations = {
    en: { welcome: 'Hello', name: 'Charlie', settings: 'Settings' },
    ko: { welcome: '안녕하세요', settings: '설정' },
};

export function App () {
    return (...);
}
```

```js
// Home.js
export const Home = ({navigation}) => {
    return (
        <View>
            <Text>{i18n.t("welcome")}</Text>
            <Button title="Go to Settings" onPress={() => navigation.navigate("Settings")} />
        </View>
    );
}
```

```js
// Settings.js
export const Settings = ({navigation}) => {
    return (
        <View>
            <Text>{i18n.t("settings")}</Text>
            <Button title="Go to Home" onPress={() => navigation.navigate("Home")} />
        </View>
    );
}
```

`i18n.translations` has only one scope on the `App` screen. i18n's scope is **global**, so you do not need to worry its scope will be lost.

> If you want to keep tracking your scope easily, you could go with this solution.

## Solution 2: Update your scope when your view is changed

**Solution 1** is good if your localization texts are not too many. But if your localization texts are too many, you would prefer to divide texts per page. This approach enhances the initial load time and makes your code clean.

```js
// Home.js
export const Home = ({navigation}) => {
    useEffect(() => {
        // ! Can not track translation.
        i18n.translations = {
          en: { welcome: 'Hello', name: 'Charlie' },
          ko: { welcome: '안녕하세요' },
        };
    }, [])

    return (
        <View>
            /* ! Missing translation error */
            <Text>{i18n.t("welcome")}</Text>
            <Button title="Go to Settings" onPress={() => navigation.navigate("Settings")} />
        </View>
    );
}
```

We can use the `useEffect` hook to initialize scope but the problem is you could not track your scope after `i18n.translation` is changed. Because `useEffect` is executed after rendering.

> `i18n.t()` must be re-rendered after new translation scope is applied.

You need to use an asynchronous hook to update the value without missing translation. Let's suspense rendering your components until completing to load target translation.

```js
// Home.js
export const Home = ({navigation}) => {
    const [isLoaded, setIsLoaded] = useState(false);
    
    useEffect(() => {
        const unsubscribe = navigation.addListener('focus', () => {
            i18n.translations = {
              en: { welcome: 'Hello', name: 'Charlie' },
              ko: { welcome: '안녕하세요' },
            };
            setIsLoaded(true);
        });

        return () => unsubscribe()
    }, [])

    return (
        <View>
            {!isLoaded ? 
                <Loader /> : 
                <>
                    <Text>{i18n.t("welcome")}</Text>
                    <Button title="Go to Settings" onPress={() => navigation.navigate("Settings")} />
                </>
            }
        </View>
    );
}
```

> `navigation.addListener`'s focus event fired when the screen is focused every time. `isLoaded` will update the state when translation is changed.

## Conclusion

Okay, now you could manage your i18n texts. Let's summarize. 
1. `Missing translation` means your scope is not focused and possibly your scope is focusing on a previous scope or nothing is scoped.
2. **Solution 1** can be complicated if your app is big. Maybe you would be suffered from creating each unique name. So I recommend declaring your translation scope per page, but not each component. It will make your code complicated.
3. If you applied scope to your app, try restarting your app-building(Only if you can not find any other problem after applying scope and when `Missing translation` error happens).
4. ❗ `i18n.t()` method should be used after `i18n.translations` is assigned, or you would get `Missing translation` error. So, if you declared `i18n.t()` in a static file that is being built during building time such as **error texts** you will get the `Missing translation`. To prevent it, you can make them as methods and inject `i18n`. Thus, it will update your recent `i18n.translations`. (i18n-js version 3.9.2)

```js
export const example = (i18n) => {
    return i18n.t("tv")
} 
```


How about your unique solution? Or do you have a better solution? Share it with me. 👋

<script src="https://utteranc.es/client.js"
        repo="mauvpark/mauvpark.github.io" 
        issue-term="pathname"
        theme="github-light"
        label="comment"
        crossorigin="anonymous"
        async>
</script>
