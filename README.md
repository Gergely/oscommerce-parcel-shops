# osCommerce 2 Parcel Shops edition

oscommerce-parcel-shops


[oscommerce official web site](https://www.oscommerce.com/)

[osCommerce 2 on github](https://github.com/osCommerce/oscommerce2)


This code proposal enable to use Parcel Shops shipping address in checkout.
Many shipping company offer Parcel automats, Shops or PickUp Points so its time to extend our favourite shopping osCommerce cart with this option.


This mod is very simple to use only need some little code changes.

### What changes need?

1. Extend shipping class

2. Edit checkout steps

3. Email editions

4. Save shipping quotes into orders table

### What are effects doing?

Shipping selection page ensure that be able to use google map `iframe` apps, `jQuery autocomplete` or `Bootstrap typeahead` input field selectors.
Session safety data transfer allow to use parcel identity number during the whole checkout.

This is mean that shipping address is keeping everywhere and no need extra manipulation after checkout.
Parcel address is saved in orders table as shipping address.

Session safety data ensure preselections for page *checkout_shipping.php* in shipping modules per session users.
After success checkout `shipping` is altered from session.

Shipping quote methods save into orders so administration could use it in order manipulations.


### Cheet codes and logics

#### Get `{myparcel_id}` from session everywhere you need:

```php
$_SESSION['shipping']['data']['{myparcel_id}'];
```

#### Save data into the $_SESSION['shipping'] with shipping module function
Use `before_payment` class method in {yourparcelshopmodule} class.

Code Example:
```php
  function before_payment() {
    global $shipping, $HTTP_POST_VARS;
      $error = true;
      if (isset($HTTP_POST_VARS['{myparcel_id}']) && !empty($HTTP_POST_VARS['{myparcel_id}'])) {
        $pre_data = tep_db_prepare_input($HTTP_POST_VARS['{myparcel_id}']);

        if (tep_not_null($pre_data[$shipping['id']])) {
          $shipping['data']['{myparcel_id}'] = $pre_data[$shipping['id']];
          $error = false;
        }
      }
      if ($error) {
        $shipping['error'] = CHECKOUT_SHIPPING_PARCELSHOP_ERROR;
        tep_redirect(tep_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL'));
      }
  }
```

#### Prepaire the Parcel Shop address for *checkout_confirmation.php* page
Use `before_process` class method in {yourparcelshopmodule} class.

Code example:
```php
  function before_process() {
    global $order, $shipping;
    if ( isset($shipping['data']['{myparcel_id}']) && tep_not_null($shipping['data']['{myparcel_id}'])) {
      $parcelshop_data = $this->parcelshop_data($shipping['data']['{myparcel_id}']);
      $order->delivery['company'] = $parcelshop_data['company'];
      $order->delivery['name'] = $order->delivery['lastname'] . ' ' . $order->delivery['firstname'];
      $order->delivery['street_address'] = $parcelshop_data['street'];
      $order->delivery['suburb'] = '';
      $order->delivery['city'] = $parcelshop_data['city'];
      $order->delivery['postcode'] = $parcelshop_data['pcode'];
      $order->delivery['state'] = '';
    }
  }
```

#### Display on *checkout_confirmation.php* page the Parcel Shop address
Use `cc_process` class method in {yourparcelshopmodule} class.

```php
  function cc_process() {
     return {your email parcel shop address html};
  }
```

#### Parcel Shop address rendering into emails
Use `after_process` class method in {yourparcelshopmodule} class.

```php
  function after_process() {
     return {your email parcel shop address text/html};
  }
```



I have tried to follow the original oscommerce coding logic as payment modules work.
Modifications are found in this github repo branches.
