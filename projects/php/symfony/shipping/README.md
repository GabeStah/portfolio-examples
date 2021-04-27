**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](https://github.com/GabeStah/portfolio-examples/#warning--copyright-notice) for more info.**

# Solarix Shipping (PHP)

The Solarix Shipping (PHP) package handles basic shipping tasks with a provider-based plugin system.

## Features

- Shippable units (groups of items)
- Shippable items
- Rate requests/responses
- Origin/destination address handling
- Provider-specific overrides and extensibility (i.e. FedEx, UPS, DHL, etc)

## Architecture

Solarix Shipping makes heavy use of strategy, factory, builder, and chain of responsibility design principles.

A provider is selected via code or environment variable which determines provider-specific logic, classes, and overrides to be invoked during execution. Strict interface adherence encourages strategy patterns, allowing the provider to override a given class/method/behavior without the client's knowledge.

## Usage

Instantiate a [provider](src/Solarix/Shipping/Provider) such as `FedExProvider`. In the example below the [FedExProvider](src/Solarix/Shipping/Provider/FedExProvider.php) is being autowired via dependency injection within a Symfony app.

```yaml
services:
  solarix.shipping.provider.fedex:
    class: Solarix\Shipping\Provider\FedExProvider

  app.shipping_calculator.fedex:
    class: App\Shipping\Calculator\FedExRateCalculator
    arguments:
      ['@app.shipping.fedex_service', '@solarix.shipping.provider.fedex']
    tags:
      - {
          name: sylius.shipping_calculator,
          calculator: fedex,
          label: 'FedEx',
          form_type: App\Form\Type\FedExShippingCalculatorType,
        }
```

```php
/**
 * @param FedExProvider $shippingProvider
 */
public function __construct(
  FedExProvider $shippingProvider
) {
  $this->shippingProvider = $shippingProvider;
}
```

However, the client may opt to manually instantiate the `Provider` instance:

```php
$this->shippingProvider = new FedExProvider();
```

### Factory Object Instantiation

Create object instances via [factories](src/Solarix/Shipping/Factory) obtained through the provider's `getXXXFactory` methods. While the client _can_ directly instantiate objects, it is advisable to use the provider's factory methods to ensure the generated instances are compatible with the provider.

A typical first step is to create a `Shipment`, which acts as a parent object to all shipment-related entities:

```php
// Create a Shipment
$shipment = $this->shippingProvider->getShipmentFactory()->create();
```

### Shippable Objects

A `Shippable` object represents a single shippable entity, such as a physical product, a digital document, etc. A `ShippableUnit` is a _collection_ of one or more `Shippable` objects.

Start adding shippable entities to the `Shipment`. In most cases, all `Shippables` within a `Shipment` will be added to the same `ShipmentUnit`:

```php
// Create a ShippableUnit
$shippableUnit = $this->shippingProvider
  ->getShippableUnitFactory()
  ->create();

// Create a Shippable
$shippable = $this->shippingProvider
  ->getShippableFactory()
  ->create()
  ->setDepth(10.0)
  ->setHeight(15.0)
  ->setId('foobar')
  ->setQuantity(3)
  ->setWeight(5.55)
  ->setWidth(10.0)

// Add Shippable to ShippableUnit
$shippableUnit->addShippable($shippable);

// Add ShippableUnit to Shipment
$shipment->addShippableUnit($shippableUnit);
```

However, in some scenarios it may be useful to divide `Shippables` into separate `ShippableUnit` groupings. For example, one `ShippableUnit` for physical goods, one for digital goods, etc.

### Addressing

If the `Provider` requires physical addresses to calculate rates the next step is to use the `AddressFactory` to create addresses:

```php
$originAddress = $this->shippingProvider
  ->getAddressFactory()
  ->create()
  ->setCity('Portland')
  ->setCountryCode('US')
  ->setPostalCode(97214)
  ->setStateOrProvinceCode('OR')
  ->setStreetLines([
    '1945 SE Water Ave'
  ]);

$destinationAddress = $this->shippingProvider
  ->getAddressFactory()
  ->create()
  ->setCity('New York')
  ->setCountryCode('US')
  ->setPostalCode(10011)
  ->setStateOrProvinceCode('NY')
  ->setStreetLines([
    '20 W 34th St'
  ]);
```

Solarix Shipping also supports generating default origin/destination addresses via environment variables:

```
SOLARIX_SHIPPING_DEFAULT_DESTINATION_ADDRESS_STREET="1945 SE Water Ave"
SOLARIX_SHIPPING_DEFAULT_DESTINATION_ADDRESS_CITY="Portland"
SOLARIX_SHIPPING_DEFAULT_DESTINATION_ADDRESS_STATE="OR"
SOLARIX_SHIPPING_DEFAULT_DESTINATION_ADDRESS_POSTAL_CODE="97214"
SOLARIX_SHIPPING_DEFAULT_DESTINATION_ADDRESS_COUNTRY="US"

SOLARIX_SHIPPING_DEFAULT_ORIGIN_ADDRESS_STREET="20 W 34th St"
SOLARIX_SHIPPING_DEFAULT_ORIGIN_ADDRESS_CITY="New York"
SOLARIX_SHIPPING_DEFAULT_ORIGIN_ADDRESS_STATE="NY"
SOLARIX_SHIPPING_DEFAULT_ORIGIN_ADDRESS_POSTAL_CODE="10011"
SOLARIX_SHIPPING_DEFAULT_ORIGIN_ADDRESS_COUNTRY="US"
```

To generate a default address just call the `AddressFactory->default()` method, optionally specifying the default address type:

```php
$originAddress = $this->shippingProvider
  ->getAddressFactory()
  ->default(AddressFactory::ORIGIN_TYPE);

$destinationAddress = $this->shippingProvider
  ->getAddressFactory()
  ->default(AddressFactory::DESTINATION_TYPE);
```

Once addresses are generated add them to the `Shipment`:

```php
$shipment->setOrigin($originAddress);
$shipment->setDestination($destinationAddress);
```

### Obtaining Provider Rates

To calculate shipping provider rates start by instantiating a `RateRequest` via the provider factory, passing in the `Shipment` instance which contains all relevant data:

```php
$rateRequest = $this->shippingProvider
  ->getRateRequestFactory()
  ->create($shipment);
```

The `RateRequest` can now be invoked at any time via the `makeRequest()` method, which will execute the necessary backend API calls of the provider and return a `RateResponse` instance:

```php
$rateResponse = $rateRequest->makeRequest();
```

The `RateResponse` instance may contain a collection of `ResponseStatus` objects which store messages, errors, and codes provided by the underlying provider.

Checking the `RateResponse` for errors can be accomplished via the `hasError()` method:

```php
if (!$rateResponse->hasError()) {
  return $rateResponse->getRates();
}
```

The `RateResponse` contains a collection of [Rate](src/Solarix/Shipping/Model/Rate/Rate.php) instances, each of which represent a singular provider rate with an id, base charge, fes, taxes, etc. Most providers generate multiple rates, so the `RateResponse->getRates()` method returns a collection of one or more `Rates`.

Generated `Rates` can now be used by the client. For example, obtaining the `Rate` instance based on the provider-generated `id` property:

```php
/** @var RateInterface $rate */
foreach ($rateResponse->getRates() as $rate) {
  if ('FEDEX_GROUND' == $rate->getId()) {
    return $rate;
  }
}
```
