`PrestaShop\PrestaShop\Core\Domain\Currency\Command\ToggleExchangeRateAutomatizationCommand`

{{% notice warning %}}
**Deprecated:** This command is tied to the CronJobs module, which is being phased out and will no longer be available after December 2025. To automatically update currency exchange rates, configure a standard cron job instead. See the [Pricing FAQ]({{< relref "/faq/pricing" >}}) for details on setting up currency rate updates.
{{% /notice %}}

_Class ToggleExchangeRateAutomatizationCommand is responsible for turning on or off the automatic exchange rate update setting._

| Command details            |    |
| -------------------------- | -- |
| **Constructor parameters** | <ul> <li>`$bool $exchangeRateStatus`</li> </ul> |
| **Handler class**          | `PrestaShop\PrestaShop\Adapter\Currency\CommandHandler\ToggleExchangeRateAutomatizationHandler`  <p> Implements: </p> <ul>  <li>`PrestaShop\PrestaShop\Core\Domain\Currency\CommandHandler\ToggleExchangeRateAutomatizationHandlerInterface`</li>  |
| **Return type** |  `void`  |
