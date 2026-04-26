---
title: Translations
deprecated: false
hidden: false
metadata:
  robots: index
---
By using our translation features, you can deliver data in your users' preferred languages to create a more engaging and accessible experience. Let’s explore what you can translate and how the process works.

## What fields can you translate?

The API supports translations for entities containing text, such as names. The list below shows all entities available for translation.

- country
- league
- team

## Which languages are supported?

| Language Code | Language  |
| ------------- | --------- |
| `en`          | English   |
| `bg`          | Български |
| `es`          | Español   |

> Please note: if a translation is unavailable, the API defaults to English.

## How does it work?

Integrating translations is straightforward. Use the **language** parameter with your desired country's shortcode to retrieve data in that language. The API will automatically return the translated content.

For example, to display league names in Spanish from the leagues endpoint:

```curl
https://api.sstrader.com/v1/regions?language=es
```

<br />
