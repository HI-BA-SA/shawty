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
        if db.check_exists(long_url):
            log(duplicate record)
            return db.get_short(long_url)

        new_short_url = algorithm(long_url)
        while db.check_short_exists(new_short_url):
            new_short_url = algorithm(new_short_url)

        try:
            db.save(long_url, new_short_url)
            log(success)
        catch(e):
            log(db write failure)
            return error
        
        cache.set(new_short_url)
        log(success)
        return new_short_url
```

#### Read Function
```pseudocode
    fun read(short_url):
        cached = cache.get_long_url(short_url)
        if (cached):
            log(success)
            return cached

        log(cache miss)
        long_url = db.get_long_url(short_url)
        if long_url:
            return long_url
        
        log("non-existing record")
        return error
```
