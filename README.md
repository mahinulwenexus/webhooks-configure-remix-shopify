# webhooks-configure-remix-shopify
Configuring Webhooks in the easiest way in shopify using SHOPIFY REMIX APPLICATION.

<br>

#### All the other method of `configuring webhooks` in SHOPIFY APP doces are lengthy. I can easily configure the webhooks using SHOPIFY PREBUILT FUNCTIONALITIES.

<br>
<br>

## Steps
* There are some steps including the webhooks in Shopify Remix App.
* Let's first install a very basic application.
* `shopify app init`. Run the command to create a boilerplate of `shopify remix application`. 

<details close>
<summary><b>Careful:</b></summary>
<br>
Please make sure you have <b><i>shopify CLI</i></b> installed. Otherwise the <b><i>shopify app init</i></b> won't work.
</details>

<br>

* ![image](https://github.com/mahinulintern/webhooks-configure-remix-shopify/assets/167665561/5086c7b9-710b-4dc9-8470-a5b9e1edd8c4)
* The boilerplate looks something like this.
* There should be Two Major changes.
* One will be `app/shopify.server.js` and another changes will be `app/routes/webhook.js`.

<br>
<br>

# shopify.server.js
* This is where I will configure the webhooks most easily.
* The file looks like this.
<br>

```javascript
import "@shopify/shopify-app-remix/adapters/node";
import {
  ApiVersion,
  AppDistribution,
  DeliveryMethod,
  shopifyApp,
} from "@shopify/shopify-app-remix/server";
import { PrismaSessionStorage } from "@shopify/shopify-app-session-storage-prisma";
import { restResources } from "@shopify/shopify-api/rest/admin/2024-04";
import prisma from "./db.server";

const shopify = shopifyApp({
  apiKey: process.env.SHOPIFY_API_KEY,
  apiSecretKey: process.env.SHOPIFY_API_SECRET || "",
  apiVersion: ApiVersion.April24,
  scopes: process.env.SCOPES?.split(","),
  appUrl: process.env.SHOPIFY_APP_URL || "",
  authPathPrefix: "/auth",
  sessionStorage: new PrismaSessionStorage(prisma),
  distribution: AppDistribution.AppStore,
  restResources,
  webhooks: {
    APP_UNINSTALLED: {
      deliveryMethod: DeliveryMethod.Http,
      callbackUrl: "/webhooks",
    },
  },
  hooks: {
    afterAuth: async ({ session }) => {
      shopify.registerWebhooks({ session });
    },
  },
  future: {
    v3_webhookAdminContext: true,
    v3_authenticatePublic: true,
    v3_lineItemBilling: true,
    unstable_newEmbeddedAuthStrategy: true,
  },
  ...(process.env.SHOP_CUSTOM_DOMAIN
    ? { customShopDomains: [process.env.SHOP_CUSTOM_DOMAIN] }
    : {}),
});

export default shopify;
export const apiVersion = ApiVersion.April24;
export const addDocumentResponseHeaders = shopify.addDocumentResponseHeaders;
export const authenticate = shopify.authenticate;
export const unauthenticated = shopify.unauthenticated;
export const login = shopify.login;
export const registerWebhooks = shopify.registerWebhooks;
export const sessionStorage = shopify.sessionStorage;
```

<br>

* From that file, What I'm gonna change is this section:
```javascript
  webhooks: {
    APP_UNINSTALLED: {
      deliveryMethod: DeliveryMethod.Http,
      callbackUrl: "/webhooks",
    },
  },
```
![image](https://github.com/mahinulintern/webhooks-configure-remix-shopify/assets/167665561/2dabded6-a31c-4681-96eb-64d720a9cfa1)

<br>

* The `APP_UNINSTALLED` is called `webhook subscription topic`. Google it for further information.
* What is `callbackUrl: "/webhooks"`? Webhook subscription basically sends `post` request to `callbackUrl`'s link. Which can be handled by remix's `action` hook. It is simply saying that "send webhook data to this URL". <b>PROOF:</b> There is a `webhook.jsx` file in the `app/route` and also in the `webhook.jsx` file there will be a `action`. This is where Webhook is being handled.

<br>

# How to add more webhooks?
* For example I want to add `PROFILES_CREATE` webhook to my app.
![image](https://github.com/mahinulintern/webhooks-configure-remix-shopify/assets/167665561/0c31e577-bb8f-4dfd-9fd6-84caf871776a)

* Go to `app/shopify.server.js` and put code something like this
```javascript
distribution: AppDistribution.AppStore,
restResources,
webhooks: {
  APP_UNINSTALLED: {
    deliveryMethod: DeliveryMethod.Http,
    callbackUrl: "/webhooks",
  },
  PROFILES_CREATE: {
    deliveryMethod: DeliveryMethod.Http,
    callbackUrl: "/webhooks",
  },
}
```

* Whenever someone creates a product, shopify will send a `post` request to the `/webhook`.
* Now `/webhook` is simply a route of my `shopify app`.

<br>

# How does it work behind the scene
* After adding the `webhook topic` in `app/shopify.server.js` it <b>won't</b>  automatically work.
* `added webhook topic to file` -> `perform register webhook` -> `webhook will work`.
* Every shopify app[created by remix] will have a app url with which it will be connected to the main shopify app. <b>Funny thing is</b> Everytime I restart the server, it will be changed. People call it `application url`.
* Now shopify `webhooks` are <b>registed with application url</b>. So each time I restart the server, `application url` gets changed. Therefore the webhook doesn't work anymore.
* By default, `register webhooks` runs only once in the file `shopify.server.js` when I first install the run the `shopify app`.
* If I close the shopify app server and restart the server again the `webhook` which might be working, will not work again.

#### So I have to perform `webhook register manually` everytime I restart the server. How can I register webhook manually?

<br>

# Manually register webhook
* First, create a new route. Can be any name. Let's name it `webhooktest.jsx`
* <b>Goal:</b> Is to create a new route. When I visit that route, `webhook` will be registered.
* In that file: Let's put these code: (Don't want note whole details how everything works)
```javascript
import { authenticate } from "../shopify.server";
import { registerWebhooks } from "../shopify.server";

export async function loader ({request}) {
  const { session , admin} = authenticate.admin(request);
  registerWebhooks({session});
  return null;
}

export default function Page() {
  return(
    <>
      Webhook registered successfully!
    </>
  )
}

```
* Now when I visit `/webhooktest` route, webhook will be registered and will work untill I restart `shopify app server` again.
* And `shopify app server` means this
![image](https://github.com/mahinulintern/webhooks-configure-remix-shopify/assets/167665561/d09f202e-f7a7-48d9-b4ab-f52f4ed617d7)







