# What is VRZNO?

(/vərəˈzɑːnoʊ/ vər-ə-ZAH-noh)

VRZNO is a bridge between Javascript & PHP in an extremely nontraditional sense.

* VRZNO lets you call javascript code from PHP.

* VRZNO is a statically compiled PHP module.

* VRZNO runs in-browser.

## How is this possible?

VRZNO is the first PHP extension built for PIB/php-wasm. Once its compiled with PHP to WASM, it can be served to any browser and executed client side.

The PIB/php-wasm packages allow you to call PHP from javascript, the only way to communicate from PHP back to JS was to print output.

This changes all that.

VRZNO allows you to control the page directly from PHP.

## Usage

At the moment, the only exposed functions are vrzno_run($funcName, $args) & vrzno_eval($js).

`vrzno_run` will execute a global function. It takes a function name as the first param, and an array of arguments as its second:

```php
<?php vrzno_run('alert', ['Hello, world!']);
```

`vrzno_eval` will run any javascript code provided in its first param in the context of the page, and return its response as a string to PHP:

```php
<?php vrzno_eval('alert(`Hello, world!`)');
```

There are plans in the works for `vrzno_set($varname, $value)`, `vrzno_get($varname)`, and `vrzno_select($namespace)`.

## Compiling with PHP 7.4

```bash

# Enter your home dir (or wherever you'd like to work on the project)
cd ~

# Clone the PHP Repo
git clone https://github.com/php/php-src.git

# Enter the extension directory
pushd php-src/ext

# Clone the VRZNO repo into the PHP source tree
git clone https://github.com/seanmorris/vrzno.git

# Return to the project root
popd

# Remove the configure script if any exists
rm configure

# Rebuild the configuration script
./buildconf --force

# Run the new configuration script via emscripten.
emconfigure \
  ./configure \
  --disable-all \
  --disable-cgi \
  --disable-cli \
  --disable-rpath \
  --disable-phpdbg \
  --with-valgrind=no \
  --without-pear \
  --without-pcre-jit \
  --with-layout=GNU \
  --enable-embed=static \
  --enable-bcmath \
  --enable-json \
  --enable-ctype \
  --enable-mbstring \
  --disable-mbregex \
  --enable-tokenizer \
  --enable-vrzno

# Run the make scripts with emscripten.
emmake make -j8

# Compile with EMCC
pushd php-src

emmake make -j8

emcc -O3 \
	-I .              \
	-I Zend           \
	-I main           \
	-I TSRM/          \
	../source/pib_eval.c \
	-o vrzno-php.o

emcc -O3 \
  --llvm-lto 2                   \
  -s ENVIRONMENT=web             \
  -s EXPORTED_FUNCTIONS='["_pib_init", "_pib_destroy", "_pib_eval" "_pib_refresh", "_main", "_php_embed_init", "_php_embed_shutdown", "_php_embed_shutdown", "_zend_eval_string"]' \
  -s EXTRA_EXPORTED_RUNTIME_METHODS='["ccall", "UTF8ToString", "lengthBytesUTF8"]' \
  -s MODULARIZE=1                 \
  -s EXPORT_NAME="'PHP'"          \
  -s TOTAL_MEMORY=134217728       \
  -s ASSERTIONS=0                 \
  -s INVOKE_RUN=0                 \
  -s ERROR_ON_UNDEFINED_SYMBOLS=0 \
    libs/libphp7.a vrzno-php.o -o out/php.js

# Check that the files were built:

ls -al out/

```
