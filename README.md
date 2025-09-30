# GeocoderString

**GeocoderString** is a ProcessWire module that allows you to geocode arbitrary strings (addresses, postcodes, place names) or reverse-geocode coordinates into human-readable addresses.

It uses the configuration, providers, and libraries from the [FieldtypeGeocoder](https://github.com/spoetnik/FieldtypeGeocoder) module, so all authentication keys and provider settings are reused automatically.

---

## Requirements

* [ProcessWire](https://processwire.com) 3.x
* [FieldtypeGeocoder](https://github.com/spoetnik/FieldtypeGeocoder) module (must be installed and configured)

---

## Installation

1. Install and configure **FieldtypeGeocoder** (choose a provider and enter API keys).
2. Copy `GeocoderString.module` into your site’s `site/modules/GeocoderString/` folder.
3. In the ProcessWire admin, go to **Modules → Refresh**, then install **GeocoderString**.

---

## Usage

Load the module from the `$modules` API:

```php
$geocoder = $modules->get('GeocoderString');
```

### 1. Basic Geocoding

```php
$result = $geocoder->geocode('1017 XE Amsterdam');

if ($result) {
    echo "Lat: " . $result['lat'] . "\n";
    echo "Lng: " . $result['lng'] . "\n";
    echo "Address: " . $result['formatted_address'] . "\n";
}
```

---

### 2. Geocoding with Country Restriction

```php
$result = $geocoder->geocode('Amsterdam', ['country' => 'NL']);
```

---

### 3. Reverse Geocoding

```php
$address = $geocoder->reverseGeocode(52.3702, 4.8952);

if ($address) {
    echo $address['formatted_address'];
}
```

---

## Caching Results with WireCache

Geocoding APIs often have **rate limits**. To avoid repeated requests, you can cache results using ProcessWire’s `WireCache`:

### Cache a Forward Geocode

```php
$cacheKey = 'geocode_1017XE';
$result = $cache->get($cacheKey);

if (!$result) {
    $result = $geocoder->geocode('1017 XE Amsterdam');
    if ($result) {
        // cache for 1 day
        $cache->save($cacheKey, $result, WireCache::expireDay);
    }
}

echo $result['lat'] . ', ' . $result['lng'];
```

### Cache a Reverse Geocode

```php
$lat = 52.3702;
$lng = 4.8952;
$cacheKey = "reverse_{$lat}_{$lng}";

$address = $cache->get($cacheKey);

if (!$address) {
    $address = $geocoder->reverseGeocode($lat, $lng);
    if ($address) {
        // cache for 1 week
        $cache->save($cacheKey, $address, 604800); // seconds
    }
}

echo $address['formatted_address'];
```

### Notes on Caching

* Use unique cache keys per query (e.g., include postcode or lat/lng).
* Adjust cache expiry to balance freshness and API quota usage.
* All cached results are stored in ProcessWire’s standard cache system.

---

## Error Handling

* If no result is found, methods return `null`.
* Errors are logged to the ProcessWire log: `site/assets/logs/GeocoderString.txt`.

---

## License

MIT License – see LICENSE file for details.
