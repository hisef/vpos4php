## PHP için Sanal Pos Kütüphanesi ##

Dahius tarafından özgür yazılım dünyasına kazandırılmayı hedeflenen bir projedir.

VirtualPos kütüphanesi Türkiye'deki bankalar için adapter sağlar. İstenirse diğer ülkeler için de kolayca adapte edilebilir.

**Kurulum için;**
```
# pear channel-discover pear.dahius.org
# pear install Dahius/VirtualPos-beta
```

**Kullanım örneği;**

```
<?php
   session_start();

    require_once "Dahius/VirtualPos/Loader.php";

    // !!! GÜVENLİK UYARISI !!! 
    // CONFIG.YML dosyanızı web üzerinden erişilemez bir dizine kurunuz. 
    // Örnekte Linux config dizini önerilmiştir.

    $vpos = new Dahius_VirtualPos("/etc/vpos/config.yml");

    $request = new Dahius_VirtualPos_Request();
    $request->isThreeDSecure = true;
    $request->cardHolder = "Hasan Ozgan";
    $request->cardNumber = "4253-6789-2345-9876";
    $request->cvc = 454;
    $request->expireMonth = 1;
    $request->expireYear = 2011;
    $request->amount = 10.67;
    $request->currency = "TRL"; // TRL, USD, EUR
    $request->installment = 5;
    $request->orderId = md5(uniqid(rand(), true)); // Your order id

    var_dump($request->binNumber,       // 425367
             $request->secureNumber,    // 4253-68**-****-9876
             $request->cardType);       // visa

    $adapter = $vpos->factory("bonus");


    /**
     *  Adapter Features
     *  ---------------------------------- 
        $adapter->provision($request); // kredi kartından bloke işlemi
        $adapter->sale($request);      // kredi kartından para çekme
        $adapter->reversal($request);  // para çekilen işlem üzerinde kısmi düzeltme
        $adapter->disposal($request);  // bloke olan paranın çekilmesi.
        $adapter->refusal($request);   // işlem iptali.
    */

    $response = $adapter->provision($request);
    if (!$response->succeed) {
        throw new Exception($response->message);
    }

    if ($request->isThreeDSecure) {
        $_SESSION["__VirtualPOS__"] = serialize($request);
        die($response->message);
    }
    else {
        var_dump($response);
        echo "SUCCESS";
    }
```

**Ayar dosyası örneği;**

```

bonus:
    adapter: Dahius_VirtualPos_Adapter_CC5
    parameters:
        host:
            bank: https://ccpos.garanti.com.tr/servlet/cc5ApiServer
            authentication: https://ccpos.garanti.com.tr/servlet/gar3Dgate
            callback: https://example.com/callback/for/3DSecure
        username: garanti_user
        password: garanti_pass
        store_type: 3d
        working_mode: P
        client_id: 12345678
        store_key: 123456
        valid_md_status: [1, 2, 3, 4]

worldcard:
    adapter: Dahius_VirtualPos_Adapter_Posnet
    parameters:
        host:
            bank: https://www.posnet.ykb.com/PosnetWebService/XML
            authenticate: https://www.posnet.ykb.com/3DSWebService/YKBPaymentService
            callback: https://example.com/callback/for/3DSecure
        username: ykb_user
        password: ykb_pass
        tid: 12345678
        mid: 1234567890
        posnet_id: 987
        merchant_key: 310,28,192,97,13,90,45,167
        valid_md_status: [1, 2, 3, 4]
    
```

_[Dahius](http://dahius.com) tarafından geliştirilmiştir ve tüm hakları [Dahius](http://dahius.com)'a aittir._

![http://www.dahius.com/tr/wp-content/themes/several/skin/logo.png](http://www.dahius.com/tr/wp-content/themes/several/skin/logo.png)