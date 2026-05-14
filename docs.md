# Shawty Documentation

## System Architecture

The system is composed of 4 layers:

1. **Gateway Layer**
2. **Processing Layer**
3. **Cache Layer**
4. **Persistence Layer**

## Processing Layer

The processing layer handles all the business logic.

### Pseudocode/Contract

Assuming the processing layer as a function:

#### Write Function
```pseudocode
    fun write(long_url):
        // Prior checks to confirm the long_url is not in the DB
        if db.check_exists(long_url):
            // Return and exit if the URL exists
            return db.get_short(long_url)
            log("existent record")
        // If the long_url is new, pass it through the algorithm
        url_key = algorithm(long_url)
        // Checks to ensure the short URL is not present in DB
        while (db.check_exists(url_key)):
            // If it exists, regenerate the key until it is unique
            url_key = algorithm(url_key)
        db.save(long_url, url_key)  // url_key is the shortened URL
        log("success")
        // Done
```

#### Read Function
```pseudocode
    fun read(short_url):
        // Prior checks to ensure cache hit
        if (cache.check_exists(short_url)):
            // If it exists in cache, return it without consulting DB
            return cache.get_long(short_url)
        log("cache miss")
        // Forward request to DB if cache miss
        if (db.get_long(short_url)):
            return db.get_long(short_url)
        // Error: record non-existent
        log("non-existing record")
```
