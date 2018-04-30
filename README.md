# PHP WHOIS
PHP WHOIS client implementation. Provides parsed domain info and raw-text lookup responses. Sends queries to native WHOIS services via port 43

[![Build Status](https://travis-ci.org/io-developer/php-whois.svg?branch=master)](https://travis-ci.org/io-developer/php-whois)
[![PHP version](https://img.shields.io/badge/php-%3E%3D5.4-8892BF.svg)](https://secure.php.net/)
[![Packagist](https://img.shields.io/packagist/v/io-developer/php-whois.svg)](https://packagist.org/packages/io-developer/php-whois)

## Requirements
- PHP >= __5.4__ (compatible with __7.*__ up to __nightly__)
- mbstring
- intl


## Installation
Via __Composer__ cli command
````
composer require io-developer/php-whois
````
Or via __composer.json__
````
"require": {
    "io-developer/php-whois": "^3.0.0"
}
````


## Usage

#### Whois client creation

```php
<?php

require_once '../vendor/autoload.php';

use Iodev\Whois\Whois;

$whois = Whois::create();
```

#### Domain availability

```php
<?php

use Iodev\Whois\Whois;

if (Whois::create()->isDomainAvailable("google.com")) {
    echo "Bingo! Domain is available! :)";
}
```

#### Domain lookup

```php
<?php

use Iodev\Whois\Whois;

$response = Whois::create()->lookupDomain("google.com");
echo $response->getText();
```

#### Parsed domain info

```php
<?php

use Iodev\Whois\Whois;

$info = Whois::create()->loadDomainInfo("google.com");
echo "Domain created: " . date("Y-m-d", $info->getCreationDate());
echo "Domain expires: " . date("Y-m-d", $info->getExpirationDate());
echo "Domain owner: " . $info->getOwner();
```


## Advanced usage

#### Сommon client powered by memcached
```php
<?php

use Iodev\Whois\Whois;
use Iodev\Whois\Loaders\SocketLoader;
use Iodev\Whois\Loaders\MemcachedLoader;

$m = new Memcached();
$m->addServer('127.0.0.1', 11211);
$loader = new MemcachedLoader(new SocketLoader(), $m);

$whois = Whois::create($loader);
```


#### Complete domain info loading

```php
<?php

use Iodev\Whois\Whois;
use Iodev\Whois\Exceptions\ConnectionException;
use Iodev\Whois\Exceptions\ServerMismatchException;

$whois = Whois::create();
try {
    $info = $whois->loadDomainInfo("google.com");
    if (!$info) {
        echo "Null if domain available";
        exit;
    }
    echo $info->getDomainName() . " expires at: " . date("d.m.Y H:i:s", $info->getExpirationDate());
} catch (ConnectionException $e) {
    echo "Disconnect or connection timeout";
} catch (ServerMismatchException $e) {
    echo "TLD server (.com for google.com) not found in current server hosts";
}
```


#### Original WHOIS answer via lookup

```php
<?php

use Iodev\Whois\Whois;

$response = Whois::create()->lookupDomain("google.com");
echo $response->getText();
```

#### Original WHOIS answer via domain info


```php
<?php

use Iodev\Whois\Whois;

$info = Whois::create()->loadDomainInfo("google.com");
$resp = $info->getResponse();

echo "WHOIS response for '{$resp->getDomain()}':\n{$resp->getText()}";
```


#### Сustom whois hosts

```php
<?php

use Iodev\Whois\Whois;
use Iodev\Whois\Modules\Tld\Server;
use Iodev\Whois\Modules\Tld\Parser;

$whois = Whois::create();

// Define custom whois host
$customServer = new Server(".co", "whois.nic.co", false, Parser::create());

// Or define the same via assoc way
$customServer = Server::fromData([
    "zone" => ".co",
    "host" => "whois.nic.co",
]);

// Append to existing provider
$whois->getTldModule()->addServers([$customServer]);

// Now you can load info
$info = $whois->loadDomainInfo("google.co");

var_dump($info);
```
