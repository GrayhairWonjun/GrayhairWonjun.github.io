---
layout: single
title:  "Web App에 Push Notification 보내기"
comments: true
archive: true
date:   2019-09-01 21:31:00
categories: Web App
tags: Push Notifications Web App
---

Web App Push notifications 작동방식에 대해 간단히 설명하면 아래 그림과 같다. PUSH API가 W3C에 정식 표준으로 등록이 진행 중에 있다.
최신 W3C Push API 문서는 https://www.w3.org/TR/push-api/ 의 사이트에서 확인할 수 있다. 아래의 그림에서 보는 것 처럼 크게보면 사용자가 자신의 웹브라우저를 통해 push notification 등록을 진행하는 과정과 이 과정이 완료된 후 서버에서 클라이언트로 push notification 을 push service를 통해 전송하는 과정 둘로 나눌 수 있다.

![Web-App-Push-Notifications-diagram](/assets/images/posts/2019-09-01-WebAppPushNotifications.png)

### Push API 흐름
#### 1. Service Worker Registration
web page(javascript코드)에서 사용자의 web browser로 ServiceWorker 등록을 요청한다. ServiceWorker는 보통 sw.js 파일로 생성되며
push server로 부터 전송된 push message를 event handler를 통해 전달받아 원하는 형태로 display하거나 action을 정의하기 위해 사용되는 파일이다.
사용자의 웹 브라우저가 PUSH API를 지원하는지 확인한 후에 ServiceWorker 파일을 브라우저에 등록한다.
```javascript
$(function () {
    if ('serviceWorker' in navigator) {
        console.warn("register service worker");
        navigator.serviceWorker.register('sw.js').then(initialiseState);
    } else {
        console.warn('Service workers are not supported in this browser.');
    }
});
```
#### 2. Create push subscription
사용자의 웹 브라우저가 푸시서버로 등록 요청을 하는 과정. 각 푸시 서비스는 사용자의 웹 브라우저 종류에 따라 다르다. 만일 크롬 웹 브라우저라면
푸시 서비스는 구글에 의에 제공되고 파이어폭스를 사용한다면 파이어폭스 푸시 서버가 이를 제공한다. 푸시 서버에 등록이 완료되면 메시지 전송을 위한
고유한 end point url을 전달받게 된다.
```javascript
function initialiseState() {
    if (!('showNotification' in ServiceWorkerRegistration.prototype)) {
        console.warn('Notifications aren\'t supported.');
        return;
    }
    if (Notification.permission === 'denied') {
        console.warn('The user has blocked notifications.');
        return;
    }
    if (!('PushManager' in window)) {
        console.warn('Push messaging isn\'t supported.');
        return;
    }
    navigator.serviceWorker.ready.then(function (serviceWorkerRegistration) {
        serviceWorkerRegistration.pushManager.getSubscription().then(function (subscription) {
            if (!subscription) {
                console.warn("subscribe");
                subscribe();
                return;
            }
        }).catch(function(err) {
            console.warn('Error during getSubscription()', err);
        });
    });
}
function subscribe() {
    navigator.serviceWorker.ready.then(function (serviceWorkerRegistration) {
        const subscribeOptions = {
            userVisibleOnly: true,
            applicationServerKey: urlB64ToUint8Array(privateKey)
        }
        serviceWorkerRegistration.pushManager.subscribe(subscribeOptions).then(function (subscription) {
            return sendSubscriptionToServer(subscription);
        }).catch(function (e) {
            if (Notification.permission === 'denied') {
                console.warn('Permission for Notifications was denied');
            } else {
                console.error('Unable to subscribe to push.', e);
            }
        });
    });
}
```
#### 3. Distribute subscription
등록된 subscription 정보를 Application Server에 전달하면 이를 파일이나 DB에 저장하고 이후 사용자에게 메시지를 전송할 때 활용한다.
```javascript
function sendSubscriptionToServer(subscription) {
    var key = subscription.getKey ? subscription.getKey('p256dh') : '';
    var auth = subscription.getKey ? subscription.getKey('auth') : '';

    return fetch('/push/subscription', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            endpoint: subscription.endpoint,
            key: key ? btoa(String.fromCharCode.apply(null, new Uint8Array(key))) : '',
            auth: auth ? btoa(String.fromCharCode.apply(null, new Uint8Array(auth))) : ''
        })
    });
}
```
#### 4. Send push message
Application Server가 Push Server로 사용자에게 메시지를 전송할 것을 요청한다. 이 때 사용자의 subscription 정보를 이용하여 메시지를 전송한다.

아래 위의 내용을 종합하여 만들어진 sample code이다. \<your public key\>  부분에 public key를 넣어주면 된다.

**Client sample code**
```javascript
const privateKey = '<your public key>';

/**
 * Step one: run a function on load (or whenever is appropriate for you)
 * Function run on load sets up the service worker if it is supported in the
 * browser. Requires a serviceworker in a `sw.js`. This file contains what will
 * happen when we receive a push notification.
 * If you are using webpack, see the section below.
 */
$(function () {
    if ('serviceWorker' in navigator) {
        console.warn("register service worker");
        navigator.serviceWorker.register('js/util/sw.js').then(initialiseState);
    } else {
        console.warn('Service workers are not supported in this browser.');
    }
});

/**
 * Step two: The serviceworker is registered (started) in the browser. Now we
 * need to check if push messages and notifications are supported in the browser
 */
function initialiseState() {
    // Check if desktop notifications are supported
    if (!('showNotification' in ServiceWorkerRegistration.prototype)) {
        console.warn('Notifications aren\'t supported.');
        return;
    }
    // Check if user has disabled notifications
    // If a user has manually disabled notifications in his/her browser for
    // your page previously, they will need to MANUALLY go in and turn the
    // permission back on. In this statement you could show some UI element
    // telling the user how to do so.
    if (Notification.permission === 'denied') {
        console.warn('The user has blocked notifications.');
        return;
    }
    // Check is push API is supported
    if (!('PushManager' in window)) {
        console.warn('Push messaging isn\'t supported.');
        return;
    }
    navigator.serviceWorker.ready.then(function (serviceWorkerRegistration) {
        // Get the push notification subscription object
        serviceWorkerRegistration.pushManager.getSubscription().then(function (subscription) {
            // If this is the user's first visit we need to set up
            // a subscription to push notifications
            if (!subscription) {
                console.warn("subscribe");
                subscribe();
                return;
            }
            // Update the server state with the new subscription
            // sendSubscriptionToServer(subscription);
        }).catch(function(err) {
            // Handle the error - show a notification in the GUI
            console.warn('Error during getSubscription()', err);
        });
    });
}

function urlB64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
        .replace(/\-/g, '+')
        .replace(/_/g, '/');

    const rawData = window.atob(base64);
    const outputArray = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
}

/**
 * Step three: Create a subscription. Contact the third party push server (for
 * example mozilla's push server) and generate a unique subscription for the
 * current browser.
 */
function subscribe() {
    navigator.serviceWorker.ready.then(function (serviceWorkerRegistration) {
        // Contact the third party push server. Which one is contacted by
        // pushManager is  configured internally in the browser, so we don't
        // need to worry about browser differences here.
        //
        // When .subscribe() is invoked, a notification will be shown in the
        // user's browser, asking the user to accept push notifications from
        // <moniassist.nhausa.com>. This is why it is async and requires a catch.
        const subscribeOptions = {
            userVisibleOnly: true,
            applicationServerKey: urlB64ToUint8Array(privateKey)
        }
        serviceWorkerRegistration.pushManager.subscribe(subscribeOptions).then(function (subscription) {
            // Update the server state with the new subscription
            return sendSubscriptionToServer(subscription);
        }).catch(function (e) {
            if (Notification.permission === 'denied') {
                console.warn('Permission for Notifications was denied');
            } else {
                console.error('Unable to subscribe to push.', e);
            }
        });
    });
}

/**
 * Step four: Send the generated subscription object to our server.
 */
function sendSubscriptionToServer(subscription) {
    // Get public key and user auth from the subscription object
    var key = subscription.getKey ? subscription.getKey('p256dh') : '';
    var auth = subscription.getKey ? subscription.getKey('auth') : '';

    // This example uses the new fetch API. This is not supported in all
    // browsers yet.
    return fetch('/push/subscription', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            endpoint: subscription.endpoint,
            // Take byte[] and turn it into a base64 encoded string suitable for
            // POSTing to a server over HTTP
            key: key ? btoa(String.fromCharCode.apply(null, new Uint8Array(key))) : '',
            auth: auth ? btoa(String.fromCharCode.apply(null, new Uint8Array(auth))) : ''
        })
    });
}
```
아래 코드는 serviceWorker 예제코드이다. 메시지를 받는 이벤트와 받은 메시지를 클릭했을때의 이벤트에 대한 핸들러를 등록하는 코드이다.

**sw.js 예제 소스코드**
```javascript
'use strict';

self.addEventListener('push', function(event) {
    console.log('[Service Worker] Push Received.');
    const title = 'Push Message Received';
    const options = {
        body: event.data.text(),
        icon: 'images/icon.png',
        badge: 'images/badge.png'
    };
    event.waitUntil(self.registration.showNotification(title, options));
});

self.addEventListener('notificationclick', function(event) {
    console.log('[Service Worker] Notification click Received.');
    event.notification.close();
    event.waitUntil(clients.openWindow(event.data.url));
});
```
아래 서버 예제는 web-push 라이브러리(https://github.com/web-push-libs/webpush-java/) 를 이용하여 push notification을 전송한다. 서버에서 메시지를 client에게 push notification을 전송하기 위해서는 private key와 public key를 `PushService` 객체에 등록해 주어야만 한다.

**Server 예제 코드**
```java
@Service
public class PushNotificationService {

    @Value("${push.notification.web.publickey}") String publicKey;
    @Value("${push.notification.web.privatekey}") String privateKey;

    public void sendPushMessage(PushSubscription sub, byte[] payload) throws GeneralSecurityException, InterruptedException, JoseException, ExecutionException, IOException {
        Notification notification;
        PushService pushService;
        // Create a notification with the endpoint, userPublicKey from the subscription and a custom payload
        notification = new Notification(
                sub.getEndpoint(),
                sub.getUserPublicKey(),
                sub.getAuthAsBytes(),
                payload
        );
        // Instantiate the push service, no need to use an API key for Push API
        pushService = new PushService();
        pushService.setPublicKey(publicKey);
        pushService.setPrivateKey(privateKey);
        // Send the notification
        pushService.send(notification);
    }
}
```
```java
@Data
@ToString
public class PushSubscription {
    private String endpoint;
    private String key;
    private String auth;

    public PushSubscription() {
        // Add BouncyCastle as an algorithm provider
        if (Security.getProvider(BouncyCastleProvider.PROVIDER_NAME) == null) {
            Security.addProvider(new BouncyCastleProvider());
        }
    }

    /**
     * Returns the base64 encoded auth string as a byte[]
     */
    public byte[] getAuthAsBytes() {
        return Base64.decode(getAuth());
    }
    /**
     * Returns the base64 encoded public key string as a byte[]
     */
    public byte[] getKeyAsBytes() {
        return Base64.decode(getKey());
    }

    /**
     * Returns the base64 encoded public key as a PublicKey object
     */
    public PublicKey getUserPublicKey() throws NoSuchAlgorithmException, InvalidKeySpecException, NoSuchProviderException {
        KeyFactory kf = KeyFactory.getInstance("ECDH", BouncyCastleProvider.PROVIDER_NAME);
        ECNamedCurveParameterSpec ecSpec = ECNamedCurveTable.getParameterSpec("secp256r1");
        ECPoint point = ecSpec.getCurve().decodePoint(getKeyAsBytes());
        ECPublicKeySpec pubSpec = new ECPublicKeySpec(point, ecSpec);

        return kf.generatePublic(pubSpec);
    }
}
```

Server 코드를 위해 아래의 dependency를 pom.xml 파일에 추가한다.

**pom.xml**
```xml
<dependency>
  <groupId>nl.martijndwars</groupId>
  <artifactId>web-push</artifactId>
  <version>5.0.2</version>
</dependency>
<dependency>
  <groupId>org.bouncycastle</groupId>
  <artifactId>bcprov-jdk15on</artifactId>
  <version>1.62</version>
</dependency>
```
