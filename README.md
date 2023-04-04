# Purpose

This repository contains example(s) of real-world solution(s) to demonstrate how I identify and solve problems as a software engineer. The included source is a demonstration of style and design using an functional, scalable, and idiomatic approach to a common software problem.

Disclaimer: This code is still an alpha stage work in progress. It is not designed or intended to be used by anyone but the author. This code was chosen because it demonstrates coding style utilizing asynchronous, functional, and metaprogramming behaviors rather than an end-to-end solution.

License: I do not authorize use of any part of this code. This is simply a demonstration of my code design, style, and how I might approach a problem. This serves no purpose other than evaluting the engineer's work. 

Attention: Please contact the author at daniel.guffey@gmail.com with comments, requests, and feedback.

The repository will be initially populated by a Python module I've designed and implemented called cache that defines a function decorator that automatically caches a function's results to reduce API costs without needing to modify any existing logic.

## cache

The purpose of this decorator is to automatically cache processing work at the function level to reduce system load for repeated function calls while not requiring any new or special changes to existing logic. Because the arguments are hashed, cached results for an add(x,y) function will dynamically pull the correct cache file for add(1,2) or add(2,2) and store the results seperately for each call.

This cache decorator can be configured to handle different use cases using the Cache dataclass. At its core this logic will automatically hash the target function's parameters and store the response using a configurable serializer.

### Description

This cache module provides a simple way to cache the results of asynchronous functions. It's designed with simplicity in mind and can be used in various applications where caching results is needed.

The cached decorator function takes an Optional configuration object defined as a dataclass called Cache.

The default configuration and options for Cache are defined below:

    serializer: Optional[Callable] = field(default=json.dumps)
    deserializer: Optional[Callable] = field(default=json.loads)
    cache_folder: Optional[Path] = field(default_factory=lambda: Path(".cache"))
    archive_folder: Optional[Path] = field(
        default_factory=lambda: Path(".cache/archive")
    )
    cache_expiry: Optional[Union[timedelta, datetime]] = field(default=None)
    logger: Optional[logging.Logger] = field(
        default_factory=lambda: logging.getLogger(__name__)

### Features

* Cache the results of async functions to the file system
* Load the cache based on function name, arguments, and keyword arguments
* Define cache expiration policies (timedelta or datetime)
* Cleanup expired cache files by moving them to an archive folder
* Customizable serializer and deserializer (default: JSON)

### Usage

    from decorators.cache import cached, Cache
    import openai

    @cached(cache=Cache(cache_expiry=timedelta(days=30)))
    async def list_engines() -> Dict:
        engine_list = await openai.Engine.alist()
        return engine_list

    @cached
    async def expensive_function(a: int, b: int) -> int:
        await asyncio.sleep(1)
        return a + b

    async def main():
        result = await expensive_function(1, 2)
        print(result)
        engines = await list_engines()
        print(engines)

    if __name__ == "__main__":
        asyncio.run(main())


#### Usage Description

In this example a coroutine is decorated by the cached decorator and when called the first time the decorator will store the results in a file. Then for the next 30 days based on the cache file's creation timestamp the cache file will be returned rather than calling out to the openai API. 

In functions with parameters like the second example, the args and kwargs are serialized and hashed as part of the cache file to support a keyed function to multiple cache file mapping.

