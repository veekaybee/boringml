---
title: is_set

weight: 2
bookToc: false
---

# Is set versus array_key_exists

I dug into the performance of isset versus array_key_exists in PHP. 

Both look at an array and determine if it has a specific key, but [their behavior is different.](https://coderwall.com/p/q9erfw/isset-vs-array_key_exists)

isset will return false if the value of that key is null. array_key_exists will only look at the key itself. 

Using an id as an example: 

```php
$a = array('1' => '`12345678`', 'key2' => null);

isset($a['key1']); //true
isset($a['key2']); //false

array_key_exists('key1', $a); // true
array_key_exists('key2', $a); //true
```

So if you want your function to be null-safe, isset is always the best best. It's also marginally better for performance because it's a PHP language construct rather than a function, like [array_key_exists]. 

Looking at a small sample of data, we can see the marginal performance improvements at scale: (note, it's better not to do all three performance comparisons in the same script because, using the same array for inserts means there's a small cache warm-up benefit for array_key_exists if it comes after isset. 

```php
<?php

// Small Loops
$small_array = array();

for($i = 0;$i < 100;$i++) {
    array_push($small_array,$i);
}

$start_small = hrtime(true);
for($i = 0;$i < 1000;$i++) {
    isset($small_array['key1']);
}
$end_small = hrtime(true);

$small_isset_time = ($end_small - $start_small) / 1000 ;
echo "Small isset loop is $small_isset_time ns \n ";

$start_small = hrtime(true);
for($i = 0;$i < 1000;$i++) {
    array_key_exists('key1', $small_array);
}
$end_small = hrtime(true);

$small_isset_time = ($end_small - $start_small) / 1000 ;
echo "Small array_key_exists loop is $small_isset_time ns \n";
```

// Medium Loops

```php
$medium_array = array();
for($i = 0;$i < 1000;$i++) {
    array_push($medium_array, $i);
}


$start_medium = hrtime(true);
for($i = 0;$i < 1000;$i++) {
    isset($medium_array['key1']);
}
$end_medium = hrtime(true);

$medium_isset_time = ($end_medium - $start_medium) / 1000 ;
echo "Medium isset loop is $small_isset_time ns \n ";

$start_medium = hrtime(true);
for($i = 0;$i < 1000;$i++) {
    array_key_exists('key1', $medium_array);
}
$end_medium = hrtime(true);

$medium_isset_time = ($end_medium - $start_medium) / 1000 ;
echo "medium array_key_exists loop is $medium_isset_time ns \n";
```

// Large Loops
```php
$large_array = array();
for($i = 0;$i < 100000;$i++) {
    array_push($large_array, $i);
}

$start_large = hrtime(true);
for($i = 0;$i < 1000;$i++) {
    isset($large_array['key1']);
}
$end_large = hrtime(true);

$large_isset_time = ($end_large - $start_large) / 1000 ;
echo "Large isset loop is $small_isset_time ns \n ";

$start_large = hrtime(true);
for($i = 0;$i < 1000;$i++) {
    array_key_exists('key1', $large_array);
}
$end_large = hrtime(true);

$large_isset_time = ($end_large - $start_large) / 1000 ;
echo "Large array_key_exists loop is $large_isset_time ns \n";
```


# Results: 

```bash
Small isset loop is 18.179 ns 
Small array_key_exists loop is 30.321 n

medium isset loop is 16.552 ns 
medium array_key_exists loop is 20.224 ns 

large isset loop is 16.669 ns 
large array_key_exists loop is 19.662 ns 
```



# Usage

Does using one or the other matter? It depends and probably doesn't matter much in my specific use case (where the calls to Lucene/ES are of a larger concern), but, given the null safety guarantees and marginal performance improvement, isset would be better to use in general.