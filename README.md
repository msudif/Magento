# Magento

Snippets of Code I find useful for various magento tasks

#Change subscribers into a customer group (find the customer group in URL headers)
```
UPDATE
    customer_entity,
    newsletter_subscriber
SET
    customer_entity.`group_id` = 5
WHERE
    customer_entity.`entity_id` = newsletter_subscriber.`customer_id`
AND
    newsletter_subscriber.`subscriber_status` = 1;
```

# Generate test products

```

<?php
require 'app/Mage.php';

Mage::app()->setCurrentStore(Mage_Core_Model_App::ADMIN_STORE_ID);

for ($i = 0; $i < 100; $i++) {

    Mage::getModel('catalog/product')

        ->setWebsiteIds(array(1))

        ->setAttributeSetId(4)

        ->setTypeId('simple')

        ->setSku($i)

        ->setStatus(1)

        ->setTaxClassId(2)

        ->setVisibility(4)

        ->setName("Product Number $i")

        ->setDescription("This widget will give you years of trouble-free widgeting.")

        ->setShortDescription("High-end widget.")

        ->setPrice(99.99)

        ->setCategoryIds(array(152))

        ->setWeight(1.0)

        ->setStockData(array(

            'use_config_manage_stock' => 0,

            'manage_stock' => 1,

            'min_sale_qty' => 1,

            'max_sale_qty' => 2,

            'is_in_stock' => 1,

            'qty' => 999

        ))

        ->save();

};
```


# Get configurable products with disabled child products
```
<?php
$store_id = 0;
$action = "view";
require_once('app/Mage.php'); 
umask(0);
Mage::app()->setCurrentStore($store_id); 
echo 'loading store id: ' . $store_id . '<br />';

// grab ALL configurable products in the store
$configurables = Mage::getModel('catalog/product')->getCollection()
        ->addAttributeToFilter('type_id', 'configurable')
        ->addAttributeToSelect('sku');
$mismatch = array();
foreach($configurables as $configurable) {
	$i++;
    // grab ALL simple product ids associated with the current configurable product
    foreach($configurable->getTypeInstance()->getUsedProductIds() as $id) {
        $simple = Mage::getModel('catalog/product')->load($id);
        if ($simple->getStatus() ==Mage_Catalog_Model_Product_Status::STATUS_DISABLED) {
            if ($action == "view") {        
                $cfgSku = $configurable->getSku();
            
                if (!isset($mismatch[$cfgSku])) {
                    $mismatch[$cfgSku] = array(
                        'sku'         => $configurable->getSku(),
                        'associated'    => array()
                    );
                }
            
                $mismatch[$cfgSku]['associated'][] = array(
                    'sku'   => $simple->getSku()
					);
				echo 'Parent: '.$configurable->getSku(). ' child: '.$simple->getSku().'//'.$i.'<br>';
            } else {
                // update simple product's to match parent configurable

            }
        }
    }
}
```

#Find unsed attribute
```
SELECT attribute_id, attribute_code, frontend_label
FROM eav_attribute
WHERE entity_type_id = 4
AND attribute_id NOT IN(SELECT attribute_id FROM eav_entity_attribute);
```

#Find Products with no image
```
SELECT
    e.sku, COUNT(m.value) as cnt
FROM
    catalog_product_entity e 
    LEFT JOIN catalog_product_entity_media_gallery m
        ON e.entity_id = m.entity_id
GROUP BY
    e.entity_id
HAVING
    cnt = 0
```

#Create new admin user

```
INSERT INTO admin_user SELECT
	NULL `user_id`,
	"mohamed" `firstname`,
	"hasan" `lastname`,
	"admin@mydomain.com" `email`,
	"mohamed" `username`,
	"e99a18c428cb38d5f260853678922e03" `password`,
	NOW() `created`,
	NULL `modified`,
	NULL `logdate`,
	0 `lognum`,
	0 `reload_acl_flag`,
	1 `is_active`,
	NULL `extra`,
	NULL `rp_token`,
	NOW() `rp_token_created_at`
;

INSERT INTO admin_role SELECT
	NULL `role_id`,
	(SELECT `role_id` FROM admin_role WHERE `role_name` = 'Administrators') `parent_id`,
	2 `tree_level`,
	0 `sort_order`,
	'U' `role_type`,
	(SELECT `user_id` FROM admin_user WHERE `username` = 'mohamed') `user_id`,
	'mohamed' `role_name`;
```
