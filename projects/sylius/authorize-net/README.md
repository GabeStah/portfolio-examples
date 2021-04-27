**Note: This README is extracted from the original source project that I created. Please see [WARNING & COPYRIGHT NOTICE](../../../README.md#warning--copyright-notice) for more info.**

# SyliusAuthorizeNetPlugin

The SyliusAuthorizeNetPlugin adds [Authorize.net](https://www.authorize.net/) payment gateway support to the [Sylius](https://sylius.com/) platform.

This plugin was created out of a need for an Authorize.net + Sylius gateway that is compatible with the modern Authorize.net API. Unfortunately, both the existing [Authorize.net Payum](https://github.com/Payum/Payum/tree/master/src/Payum/AuthorizeNet/Aim) integration as well as [Ben Borla's SyliusAuthorizeNet](https://github.com/benborla/sylius-authorize-net) plugin rely on the deprecated [Authorize.net AIM](https://developer.authorize.net/api/upgrade_guide.html#aim) API. A new Payum gateway and integrated Sylius plugin was the only way to create a functional Authorize.net solution.

## Features

- Sylius plugin for single-step installation
- Internal [Payum](https://github.com/Payum/Payum) integration to rely on existing Sylius state machines
- One-step payment information collection with combined authorization and capture logic
- Theme-aware payment form template
- Authorize.net API response and error handling

## Installation

Install in the target project:

```bash
composer install solarix/sylius-authorize-net-plugin
```

## Configuration

1. Add relevant override definitions to `_sylius.yaml` config:

```yaml
sylius_payum:
  template:
    obtain_credit_card: '@SolarixSyliusAuthorizeNetPlugin/obtainCreditCard.html.twig'
```

2. Add a new Sylius Payment Method and select `Authorize.net`
3. Add credentials to `API ID` and `Transaction ID` fields.
4. Select `Yes` for Auto-Capture if transaction should perform authorization and capture in the same step.
5. Update the appropriate `sylius_gateway_config` DB record and add any relevant template overrides:

```json
{
  "api_id": "123456",
  "transaction_key": "123456",
  "auto_capture": 1,
  "use_authorize": null,
  "payum.template.obtain_credit_card": "@SolarixSyliusAuthorizeNetPlugin/obtainCreditCard.html.twig"
}
```
