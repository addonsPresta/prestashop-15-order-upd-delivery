# prestashop-15-order-upd-delivery
PrestaShop 1.5x modification allow to update delivery method on existing order

1. Edit /admin/themes/default/template/controllers/orders/_shipping.tpl, add:
```
<p>&nbsp;</p>
<form method="post" action="{$link->getAdminLink('AdminOrders')|escape:'htmlall':'UTF-8'}&vieworder&id_order={$order->id|escape:'htmlall':'UTF-8'}">
    <span>
        <select name="id_carrier">
            {foreach from=$carriers item=c key=key name=name}
                <option value="{$c.id_carrier}">{$c.name}</option>
            {/foreach}
        </select>
        <input type="submit" class="button" name="submitIdCarrier" value="{l s='Update'}" />
    </span>
</form>
```

2. Edit /controllers/admin/AdminOrdersController.php, add in renderView():
```
'carriers' => Carrier::getCarriers((int)Context::getContext()->language->id),
```

3.  Edit /controllers/admin/AdminOrdersController.php, add in postProcess():

```
if (Tools::isSubmit('submitIdCarrier') && isset($order))
{
    $carrier = new Carrier(Tools::getValue('id_carrier'));

    if ($carrier->active && !$carrier->deleted) {
        $order->id_carrier = $carrier->id_reference;
        $order->update();

        Db::getInstance()->Execute('
        UPDATE  `'._DB_PREFIX_.'order_carrier`
        SET `id_carrier` = '.(int)$carrier->id_reference.'
        WHERE `id_order` = '.(int)$order->id);
    }
}
```
