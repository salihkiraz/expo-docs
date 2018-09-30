---
title: Localization
---

You can use this module to Localize your app, and access the locale data on the native device.
Using the popular library [`i18n-js`](https://github.com/fnando/i18n-js) with `expo-localization` will enable you to create a very accessible experience for users.

## Usage

```javascript
import React from 'react';
import { Text } from 'react-native';
import { Localization } from 'expo-localization';
import i18n from 'i18n-js';
const en = {
  foo: 'Foo',
  bar: 'Bar {{someValue}}',
};
const fr = {
  foo: 'como telle fous',
  bar: 'chatouiller {{someValue}}',
};

i18n.fallbacks = true;
i18n.translations = { fr, en };
i18n.locale = Localization.locale;
export default class LitView extends React.Component {
  componentWillMount() {
    this._subscription = Localization.addListener(({ locale }) => {
      i18n.locale = locale;
    });
  }
  componentWillUnmount() {
    if (!!this._subscription) {
      this._subscription.remove();
    }
  }
  render() {
    return (
      <Text>
        {i18n.t('foo')} {i18n.t('bar', { someValue: Date.now() })}
      </Text>
    );
  }
}
```

## API

### Constants

This API is mostly synchronous and driven by constants.

#### `Localization.locale: string`

Native device language, returned in standard format. ex: `en-US`, `es-US`.

#### `Localization.locales: Array<string>`

List of all the native languages provided by the user settings. These are returned in the order the user defines in their native settings.

#### `Localization.country: ?string`

Country code for your device.

#### `Localization.isoCurrencyCodes: ?Array<string>`

A list of all the supported ISO codes.

#### `Localization.timezone: string`

The current time zone in display format. ex: `America/Los_Angeles`

#### `Localization.isRTL: boolean`

This will return `true` if the current language is Right-to-Left.

### Methods

> Callbacks are Android only, changing the native locale on iOS will cause all the apps to reset.

#### `Localization.addListener(listener: Listener): ?Subscription`

Observe when a language is added or moved in the Android settings.

#### `Localization.removeAllListeners(): void`

Clear all language observers.

#### `Localization.removeSubscription(subscription: Subscription): void`

Stop observing when the native languages are edited.
