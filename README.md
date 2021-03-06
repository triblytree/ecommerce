E-commerce
==============
This is an eCommerce module for Laravel.

Installation
------------
Add the following requirements to your `composer.json` file:
```javascript
"repositories":
[
	{
		"type": "vcs",
		"url": "https://github.com/triblytree/ecommerce"
	}
],
"require": {
	"tribly/ecommerce": "dev-master"
},
```

Issue a `composer update` to install the package.

Add the following service provider to your `providers` array in `app/config/app.php`:
```php
'Tribly\Ecommerce\EcommerceServiceProvider'
```

Issue the following commands:
```bash
php artisan migrate --package="tribly/ecommerce"   # Run the migrations
php artisan asset:publish                        # Publish the assets
php artisan config:publish tribly/ecommerce        # Publish the config
```

Open up your `app/config/packages/tribly/ecommerce/config.php` and set your credentials for your payment gateway. An example using Stripe:
```php
'stripe' => array(
	'test' => array(
		'secret'      => 'xxxxxxxxxxxxxx',
		'publishable' => 'xxxxxxxxxxxxxx'
	),
	'live' => array(
		'secret'      => 'xxxxxxxxxxxxxx',
		'publishable' => 'xxxxxxxxxxxxxx'
	)
)
```

Finally, make sure you've defined your from 'address' and 'name' in the app/config/mail.php file. If you don't, orders won't be submitted correctly and receipts won't be sent out.

```php
'from' => array(
	'address' => 'xxxx@xxxx.xxx',
	'name' => 'xxxx'
),
```

Cart Usage
----------
[The cart class](https://github.com/JVMartin/angel-products/blob/master/src/Angel/Products/Cart.php) stores variations of products, based on their selected options, in the session.

### Add Products
```php
$Product = App::make('Product');
$Cart    = App::make('Cart');

// Grab the user's desired product from the database.
$product = $Product::with('options')->findOrFail(Input::get('product_id'));

// Mark the selected option items by their IDs.
$product->markSelectedOptions(Input::get('options'));

// Add the product to the cart in the user's desired quantity, saving the unique key for accessing it later.
$key = $Cart->add($product, Input::get('quantity'));
```

### Add Products with Custom Options
```php
$Product = App::make('Product');
$Cart    = App::make('Cart');

// Grab the user's desired product from the database.
$product = $Product::findOrFail(Input::get('product_id'));

$product->addCustomOptions(array(
	'Size' => array(
		'name'  => 'Large',
		'price' => 4.50
	),
	'Color' => array(
		'name'  => 'Green',
		'price' => -2.50,
		'image' => 'assets/images/green-shirt.jpg'
	)
));

// Add the product to the cart in the user's desired quantity, saving the unique key for accessing it later.
$key = $Cart->add($product, Input::get('quantity'));
```

### Retrieve Key
If you need to get the key for a product (i.e. to remove that product from the cart) you can do so like this:
```php
// Retrieve the key.
$key = $Cart->key($product);

// Use the key however you wish.
$Cart->remove($key);
```

### Remove Products
```php
$Cart->remove($key);
```

### Adjust the Quantity of Products
```php
$Cart->quantity($key, 5);
```

### Retrieve Products
```php
$details = $Cart->get($key);

// $details then looks like this:
array(
	'product' => {String, JSON encoded product},
	'price'   => {Float, price per unit},
	'qty'     => {Int, quantity of units}
);
```

### Loop Through Products
```php
foreach (Session::get('cart') as $key=>$details) {
	$product = json_decode($details['product']);
	$price   = $details['price'];
	$qty     = $details['qty'];
	$total   = $Cart->totalForKey($key);
}
```

### Get Totals
```php
// The total for all products in the cart.
echo $Cart->total();

// The subtotal for all products in the cart (aka, not including shipping or discounts)
echo $Cart->subtotal();

// The total for a specific product variation by key.
echo $Cart->totalForKey($key);

// The total number of items in the cart.  (Variations x their quantity)
echo $Cart->count();
```
