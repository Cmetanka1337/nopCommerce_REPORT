# nopCommerce_REPORT | ISSUE #8120: Auto-cancel unpaid orders

## Project Overview

This report documents the implementation of the nopCommerce feature requested in [issue #8120](https://github.com/nopSolutions/nopCommerce/issues/8120), completed in the fork [Cmetanka1337/nopCommerce_burtyk](https://github.com/Cmetanka1337/nopCommerce_burtyk) on the branch [issue/8120_auto-cancel-unpaid-orders](https://github.com/Cmetanka1337/nopCommerce_burtyk/tree/issue/8120_auto-cancel-unpaid-orders).

The goal of the issue was to add an automatic order cancellation flow for unpaid orders, with store-scoped configuration, support for ignored payment methods, optional cart restoration, and safe scheduling behavior for both fresh installs and existing upgrades.

## Issue Summary

The feature adds a scheduled task that automatically cancels orders with `PaymentStatus.Pending` when they remain unpaid longer than a configurable delay.

Key behavior:

- the feature is disabled by default;
- the default delay is 2880 minutes;
- ignored payment methods are respected;
- customer email is not sent on auto-cancel;
- the cart can be restored after cancellation if the setting is enabled;
- the behavior works per store in multi-store setups.

## Implemented Changes

### 1. Order settings and admin UI

The following settings were added to `OrderSettings`:

- `EnableAutoCancelUnpaidOrders`
- `AutoCancelUnpaidOrdersDelayInMinutes`
- `AutoCancelUnpaidOrdersIgnoredPaymentMethodSystemNames`
- `RestoreCartAfterAutoCancellation`

The admin settings page was updated to support:

- store-scoped overrides;
- a multi-select list of ignored payment methods;
- round-trip saving and reloading of the new settings.

### 2. Scheduled task

A new scheduled task was implemented to process unpaid orders automatically.

Behavior of the task:

- iterates through all stores;
- loads store-specific order settings;
- skips stores where the feature is disabled;
- skips invalid delay values safely;
- filters out ignored payment methods;
- cancels eligible pending orders without notifying the customer;
- restores the cart when the corresponding setting is enabled;
- logs errors per order without stopping the whole batch.

### 3. Install, upgrade, and localization

Support for the feature was added for both new and existing installations:

- a new scheduled task record is created during fresh install;
- a separate upgrade migration adds the task for existing installations;
- localization resources were added for the new admin labels and hints;
- the admin UI now renders friendly labels instead of raw localization keys.

### 4. Tests

The implementation includes tests for:

- default `OrderSettings` values;
- order settings save/load round-trip;
- normalization and parsing of ignored payment methods;
- scheduled task behavior for eligible orders, ignored methods, and store-specific settings.

## Validation

The feature was verified with local builds and manual testing.

Validation steps included:

- opening the order settings page in the admin area;
- saving the new settings and reloading the page;
- creating test pending orders;
- running the new scheduled task manually;
- confirming that eligible orders were cancelled;
- confirming that the cart restoration flow worked correctly;
- lowering the task interval to verify automatic execution.

## [Demo](https://youtu.be/q5vnebpOV7k)

## Conclusion

Issue #8120 is implemented in the current branch and the feature behaves as expected across settings, scheduled task execution, install/upgrade flow, and tests.
