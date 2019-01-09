# react-native-netaxept

React Native port of Nets Netaxept SDK

**NOTE** - This project is still under maintenance. DO NOT USE THIS YET.

## Getting started

`$ npm install react-native-netaxept --save` or `$ yarn add react-native-netaxept`

### Mostly automatic installation

`$ react-native link react-native-netaxept`

### Manual installation


#### iOS

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-netaxept` and add `RNNetaxept.xcodeproj`
3. In XCode, in the project navigator, select your project. Add `libRNNetaxept.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
4. Run your project (`Cmd+R`)<

#### Android

1. Open up `android/app/src/main/java/[...]/MainActivity.java`
  - Add `import com.calcey.RNNetaxeptPackage;` to the imports at the top of the file
  - Add `new RNNetaxeptPackage(getApplicationContext())` to the list returned by the `getPackages()` method
2. Append the following lines to `android/settings.gradle`:
  	```
  	include ':react-native-netaxept'
  	project(':react-native-netaxept').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-netaxept/android')
  	```
3. Insert the following lines inside the dependencies block in `android/app/build.gradle`:
  	```
      compile project(':react-native-netaxept')
  	```

## Usage
```javascript
import NetaxeptSDK, {
  NetaxeptConfig,
  NetaxeptPaymentMethods,
  NetaxeptCardPayment,
  NetaxeptCardToken,
  NetaxeptThemeParams,
} from 'react-native-netaxept';
```
## API

| Method                                            | Return Type                       |
| ------------------------------------------------- | --------------------------------- |
| [init(...)](#init)                                | `Promise<void>`                   |
| [setMerchantId(...)](#setMerchantId)              | `Promise<void>`                   |
| [setCurrencyCode(...)](#setCurrencyCode)          | `Promise<void>`                   |
| [getPaymentMethods(...)](#getPaymentMethods)      | `Promise<NetaxeptPaymentMethods>` |
| [addCard(...)](#addCard)                          | `Promise<void>`                   |
| [checkout(...)](#checkout)                        | `Promise<void>`                   |
| [customizeTheme(...)](#customizeTheme)            | `Promise<void>`                   |
---

## init

Initialize the PIA SDK. `merchantId` is not mandatory on initialization. You can choose to set the merchant ID using [setMerchantId(...)](#setMerchantId) at any time. The `merchantId` **MUST** be set before any actions related to payment though.

**Example**

```javascript
const config: NetaxeptConfig = {merchantBackendBaseURL: 'https://merchant.url/api'};

// You can await .init promise to resolve if
// you want to make sure the values are set before moving on to the next line
await NetaxeptSDK.init(config);
```

##### NetaxeptConfig

| Key                      | Value      | Default |
| ------------------------ | ---------- | ------- |
| merchantBackendBaseURL   | `string`   | `null`  |
| merchantId               | `string?`  | `null`  |
| currencyCode             | `string?`  | `EUR`   |
| testMode                 | `boolean?` | `false` |
| cvvRequired              | `boolean?` | `true`  |
---

## setMerchantId

Sets the merchant ID. A merchant ID is required to create a charge. In most cases, you'll only be working with a single merchantID, in that case you can set the merchantID on [initialization](#init). This function is for those who want to work with multiple merchantIDs.

**Example**

```javascript
// You can await .setMerchantId promise to resolve if
// you want to make sure the values are set before moving on to the next line
await NetaxeptSDK.setMerchantId('MERCHANT ID');
```
---

## setCurrencyCode

Sets the currency code. In most cases, you'll never need this function as you'll be working with a single currency code; so you can get away with setting it on [initialization](#init). But this is here in any case.

**Example**

```javascript
// You can await .setCurrencyCode promise to resolve if
// you want to make sure the values are set before moving on to the next line
await NetaxeptSDK.setCurrencyCode('SEK');
```
---
## getPaymentMethods

Get payment methods available for the consumer.

```javascript
const consumerId: string = 'consumer_id';
const paymentMethods: NetaxeptPaymentMethods = await NetaxeptSDK.getPaymentMethods(consumerId);
```

##### NetaxeptPaymentMethods

| Key       | Value Type                     |
| --------- | ------------------------------ |
| methods   | `Array<NetaxeptCardPayment>` |
| tokens    | `Array<NetaxeptCardToken>`   |

##### NetaxeptCardPayment

| Key         | Value Type |
| ----------- | ---------- |
| id          | `string`   |
| displayName | `string`   |
| fee         | `string`   |
| imageUri    | `string`   |
| type        | `string`   |

##### NetaxeptCardToken

| Key         | Value Type |
| ----------- | ---------- |
| id          | `string`   |
| issuer      | `string`   |
| expiryDate  | `string`   |
| imageUri    | `string`   |
| type        | `string`   |
---

## addCard

Opens the PIA terminal to add and save a card to the consumer. These saved cards can be used when making a payment.

**Example**

```javascript
try {
  const consumerId: string = 'consumer_id';
  await NetaxeptSDK.addCard(consumerId);
  console.log('Success'); // If execution comes to this line, it means card is succesfully saved
} catch (err) {
  console.warn('Whoops, something went wrong', err); // Couldn't save the card.
}
```

---

## checkout

Initiate the PIA payment flow. This will open the PIA payment modal and take over the payment process.

**Example**

```javascript
try {
  const consumerId: string = 'consumer_id';
  const paymentMethods: NetaxeptPaymentMethods = await NetaxeptSDK.getPaymentMethods(consumerId);
  // You can select either a pre-saved card token, or a different payment method as the actual `paymentMethod`
  const paymentMethod: NetaxeptCardPayment | NetaxeptCardToken = paymentMethods.tokens[0] || paymentMethods.methods[0];

  const orderNumber: string = 'NetaxeptSDK-ORDER001';
  const amount: number = 100.00;
  const vatAmount: number = 3.0;
  
  // You will get a `type` in all tokens and methods.
  // We need it back in the checkout function to work properly
  const paymentType: string = paymentMethod.type;

  await NetaxeptSDK.checkout(consumerId, orderNumber, amount, vatAmount, paymentType, paymentMethod);
  console.log('Success'); // If execution comes to this line, it means checkout is successful
} catch (err) {
  console.warn('Whoops, something went wrong', err); // Couldn't complete the payment
}
```

---

## customizeTheme

Customize the theme of PIA terminal.

```javascript
const theme: NetaxeptThemeParams = {
  systemFont: 'Montserrat-Bold',
  
  // Be careful when setting bar color
  // The status bar color changes to match this color and it doesn't change back when the card form is closed
  barColor: 'rgba(0, 0, 0, 0.8)',
  barTitleColor: '#FFFFFF',
};

// You can await .customizeTheme promise to resolve if
// you want to make sure the values are set before moving on to the next line
await NetaxeptSDK.customizeTheme(theme);
```

##### NetaxeptThemeParams

All keys are optional. Only specify the elements you want to change.

| Key                                 | Type                |  iOS | Android |
| ----------------------------------- | ------------------- | :--: | :-----: |
| `systemFont`                        | `string`            |  ✅  |   ✅  |   
| `buttonFont`                        | `string`            |  ✅  |   ✅  |   
| `fieldFont`                         | `string`            |  ✅  |   ✅  |   
| `labelFont`                         | `string`            |  ✅  |   ✅  |   
| `labelTextColor`                    | `string`            |  ✅  |   ✅  |   
| `fieldTextColor`                    | `string`            |  ✅  |   ✅  |   
| `buttonTextColor`                   | `string`            |  ✅  |   ✅  |   
| `switchThumbColor`                  | `string`            |  ✅  |   ✅  |   
| `toolbarActionButtonTextColor`      | `string`            |  ✅  |   ✅  |   
| `errorFieldBorderColor`             | `string`            |  ✅  |   ✅  |   
| `validFieldBorderColor`             | `string`            |  ✅  |   ✅  |   
| `toolbarBackgroundColor`            | `string`            |  ✅  |   ✅  |   
| `toolbarTitleColor`                 | `string`            |  ✅  |   ✅  |   
| `bodyBackgroundColor`               | `string`            |  ✅  |   ✅  |   
| `tokenCardVCLayoutBackgroundColor`  | `string`            |  ✅  |   ✅  |   
| `cardIOBackroundColor`              | `string`            |  ✅  |   ✅  |   
| `cardIOButtonTextColor`             | `string`            |  ✅  |   ✅  |   
| `cardIOButtonTextFont`              | `string`            |  ✅  |   ✅  |   
| `cardIOTextColor`                   | `string`            |  ✅  |   ✅  |   
| `cardIOPreviewFrameColor`           | `string`            |  ✅  |   ✅  |   
| `cardIOTextFont`                    | `string`            |  ✅  |   ✅  |   

**NOTE** - Use `systemFont` to change all fonts across the PIA payment process. This will be overridden if you also add elemental font types to the theme config (`buttonFont`, `fieldFont`, etc.).

**NOTE** - For fonts, give font names that are available in the app. In Android, the font must be available in `app/src/main/assets/fonts` as a `ttf` file.

**NOTE** - We support following formats of colors.
* Hex - `#FFFFFF`
* Hex with alpha - `#FFFFFFFF`
* RGB - `rgb(12, 32, 54)`
* RGBA - `rgba(12, 32, 54, 1.0)`
