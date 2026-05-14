# Shawty Documentation

## System Architecture

The system is composed of 4 layers:

1. **Gateway Layer**: validation, routing, read ratios
2. **Processing Layer**: write ratios
3. **Cache Layer**: durable tmp storage
4. **Persistence Layer**: primary storage

## Processing Layer

The processing layer handles all the business logic.

### Pseudocode/Contract

Assuming the processing layer as a function:

#### Write Function
```pseudocode
    private fun write(long_url):
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
        
        cache.set(new_short_url, long_url)
        log(success)
        return new_short_url

```

## Gateway Layer

The layer handling incoming/outgoing traffic

### Pseudocode/Contract

Assuming that the incoming/outgoing requests are denoted by req,res respectively and supposing the backend server S1 is live:

defining the routing methods to be used as a function to be passed
after validation checks have been passed

```pseudocode 
    private fun read(short_url):
        // Gateway owns cache and database fallback for reads
        cached = cache.get_long_url(short_url)
        if (cached):
            log(cache hit)
            return cached

        log(cache miss)
        long_url = db.get_long_url(short_url)
        if long_url:
            log(success)
            cache.set(short_url, long_url)
            log(cache warm with new url)
            return long_url
        
        log("non-existing record")
        return error

    private fun process(req):
        if req:
            if req.method == get: //placeholder for reads
                return read(req.short_url) //Gateway handles reads directly
            else if req.method == post: //placeholder for writes
                return S1.forward(req) //Backend S1 only receives writes
            else:
                log(unknown method)
                return error
```

now for the gateway layer's main function

```pseudocode 
    public fun process_layer(req):
        try:
          // all of these single checks must throw errors if
          there is something incorrect in the req and the check they are performing
          req.auth_check()
          req.format_check()
          req.type_check()
          req.sanitize()
          return process(req)
        catch(e):
            log(an error occured)
            return sanitizedError(e)
```

