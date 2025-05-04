# 🛡️ Twitch Ad Skipper + Turbo Overlay Remover

Браузерный UserScript, который блокирует рекламу на Twitch и автоматически удаляет оверлей с предложением Twitch Turbo.

## 🚀 Возможности

- 🔕 Блокирует рекламные запросы `fetch` и `XMLHttpRequest` к `ads.twitch.tv`
- 🧼 Удаляет HTML-баннеры и оверлеи с предложением Twitch Turbo
- 👀 Следит за DOM и убирает элементы сразу после появления
- 🧩 Работает без вмешательства в логику React-приложения Twitch

## 🧰 Установка

1. Установите расширение [Tampermonkey](https://www.tampermonkey.net/) в ваш браузер.
2. Нажмите **"Create a new script"** и вставьте содержимое скрипта из [`twitch-skipper.user.js`](./twitch-skipper.user.js)
3. Сохраните скрипт.
4. Перейдите на [https://www.twitch.tv](https://www.twitch.tv) и наслаждайтесь просмотром без рекламы и баннеров.



```js
// ==UserScript==
// @name         Twitch Skipper
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Скрывает рекламные оверлеи и предложения Twitch Turbo на Twitch
// @author       Teddy903
// @match        https://www.twitch.tv/*
// @grant        none
// @updateURL    https://github.com/Teddy903/Twitch-Skipper/raw/main/twitch-skipper.user.js
// @downloadURL  https://github.com/Teddy903/Twitch-Skipper/raw/main/twitch-skipper.user.js
// ==/UserScript==


(function () {
  'use strict';

  const blockedHosts = ['ads.twitch.tv'];

  // 1. Блокировка fetch
  const originalFetch = window.fetch;
  window.fetch = function (...args) {
    if (args[0] && blockedHosts.some(host => args[0].toString().includes(host))) {
      console.log('[Twitch Skipper] Блокирован fetch:', args[0]);
      return new Promise(() => {}); // Promise never resolves
    }
    return originalFetch.apply(this, args);
  };

  // 2. Блокировка XMLHttpRequest
  const originalOpen = XMLHttpRequest.prototype.open;
  XMLHttpRequest.prototype.open = function (method, url) {
    if (url && blockedHosts.some(host => url.includes(host))) {
      console.log('[Twitch Skipper] Блокирован XHR:', url);
      this.abort(); // прерываем запрос
      return;
    }
    return originalOpen.apply(this, arguments);
  };

  // 3. Удаление оверлея с предложением Twitch Turbo
  const hideOverlay = () => {
    const selector = 'div.player-overlay-background, div[data-a-target="tw-core-button-label-text"]';
    document.querySelectorAll(selector).forEach(el => {
      if (el.textContent?.includes('Turbo') || el.textContent?.includes('блокировщик рекламы')) {
        console.log('[Twitch Skipper] Удален оверлей:', el);
        el.closest('div')?.remove();
      }
    });
  };

  // 4. MutationObserver следит за появлением оверлея
  const observer = new MutationObserver(hideOverlay);
  observer.observe(document.body, { childList: true, subtree: true });

  // 5. И периодическая проверка (на всякий случай)
  setInterval(hideOverlay, 2000);

  console.log('[Twitch Skipper] Скрипт запущен.');
})();
