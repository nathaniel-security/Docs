# Local File Inclusion (LFI)

* &#x20;example,
  * the page may have a `?language` GET parameter,
  * if a user changes the language from a drop-down menu,
    * then the same page would be returned but with a different `language` parameter (e.g. `?language=es`).
  * In such cases, changing the language may change the directory the web application is loading the pages from (e.g. `/en/` or `/es/`).&#x20;
    *

        <figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

        <figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

### Read vs Execute

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `require()`/`require_once()` |         ✅        |      ✅      |        ❌       |
| `file_get_contents()`        |         ✅        |      ❌      |        ✅       |
| `fopen()`/`file()`           |         ✅        |      ❌      |        ❌       |
| **NodeJS**                   |                  |             |                |
| `fs.readFile()`              |         ✅        |      ❌      |        ❌       |
| `fs.sendFile()`              |         ✅        |      ❌      |        ❌       |
| `res.render()`               |         ✅        |      ✅      |        ❌       |
| **Java**                     |                  |             |                |
| `include`                    |         ✅        |      ❌      |        ❌       |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `@Html.Partial()`            |         ✅        |      ❌      |        ❌       |
| `@Html.RemotePartial()`      |         ✅        |      ❌      |        ✅       |
| `Response.WriteFile()`       |         ✅        |      ❌      |        ❌       |
| `include`                    |         ✅        |      ✅      |        ✅       |

### Filename Prefix

&#x20;\- On some occasions, our input may be appended after a different string. For example, it may be used with a prefix to get the full filename, like the following example:

```php
include("lang_" . $_GET['language']);
```

* In this case, if we try to traverse the directory with `../../../etc/passwd`, the final string would be `lang_../../../etc/passwd`, which is invalid:
*

    <figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>
* As expected, the error tells us that this file does not exist. so, instead of directly using path traversal, we can prefix a `/` before our payload, and this should consider the prefix as a directory, and then we should bypass the filename and be able to traverse directories:
*

    <figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### Appended Extensions

* This is quite common, as in this case, we would not have to write the extension every time we need to change the language.
* This may also be safer as it may restrict us to only including PHP files. In this case, if we try to read `/etc/passwd`, then the file included would be `/etc/passwd.php`, which does not exist:
*

    <figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

### Second-Order Attacks

* a web application may allow us to download our avatar through a URL like (`/profile/$username/avatar.png`).
* If we craft a malicious LFI username (e.g. `../../../etc/passwd`), then it may be possible to change the file being pulled to another local file on the server and grab it instead of our avatar.
  * we would be poisoning a database entry with a malicious LFI payload in our username.
  * Then, another web application functionality would utilize this poisoned entry to perform our attack (i.e. download our avatar based on username value).
  * This is why this attack is called a `Second-Order` attack.

### Basic Bypasses

#### Non-Recursive Path Traversal Filters

[https://academy.hackthebox.com/module/23/section/1491](https://academy.hackthebox.com/module/23/section/1491) [#oscp-checkout](app://obsidian.md/index.html#oscp-checkout)

```php
$language = str_replace('../', '', $_GET['language']);
```

* We see that all `../` substrings were removed, which resulted in a final path being `./languages/etc/passwd`.
* However, this filter is very insecure, as it is not `recursively removing` the `../` substring, as it runs a single time on the input string and does not apply the filter on the output string. - For example, if we use `....//` as our payload, then the filter would remove `../` and the output string would be `../`, which means we may still perform path traversal.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

* The `....//` substring is not the only bypass we can use, as we may use `..././` or `....\/` and several other recursive LFI payloads.
* Furthermore, in some cases, escaping the forward slash character may also work to avoid path traversal filters (e.g. `....\/`), or adding extra forward slashes (e.g. `....////`)



#### Encoding

* some of these filters may be bypassed by URL encoding our input, such that it would no longer include these bad characters
  * but would still be decoded back to our path traversal string once it reaches the vulnerable function.
* If the target web application did not allow `.` and `/` in our input, we can URL encode `../` into `%2e%2e%2f`

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>



#### Approved Paths

* Some web applications may also use Regular Expressions to ensure that the file being included is under a specific path
* &#x20;For example, the web application we have been dealing with may only accept paths that are under the `./languages` directory, as follows:

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```

* To find the approved path, we can examine the requests sent by the existing forms, and see what path they use for the normal web functionality.
* Furthermore, we can fuzz web directories under the same path, and try different ones until we get a match.
* To bypass this, we may use path traversal and start our payload with the approved path, and then use `../` to go back to the root directory and read the file we specify, as follows:

```
<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd
```

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

#### Appended Extension

* some web applications append an extension to our input string (e.g. `.php`), to ensure that the file we include is in the expected extension
* There are a couple of other techniques we may use, but they are `obsolete with modern versions of PHP and only work with PHP versions before 5.3/5.4`.

**Path Truncation**

* In earlier versions of PHP, defined strings have a maximum length of 4096 characters, likely due to the limitation of 32-bit systems
* &#x20;If a longer string is passed, it will simply be `truncated`, and any characters after the maximum length will be ignored.
* Furthermore, PHP also used to remove trailing slashes and single dots in path names, so if we call (`/etc/passwd/.`) then the `/.` would also be truncated, and PHP would call (`/etc/passwd`). PHP, and Linux systems in general, also disregard multiple slashes in the path (e.g. `////etc/passwd` is the same as `/etc/passwd`).
* Similarly, a current directory shortcut (`.`) in the middle of the path would also be disregarded (e.g. `/etc/./passwd`).
* If we combine both of these PHP limitations together, we can create very long strings that evaluate to a correct path.
* Whenever we reach the 4096 character limitation, the appended extension (`.php`) would be truncated, and we would have a path without an appended extension.
* Finally, it is also important to note that we would also need to `start the path with a non-existing directory` for this technique to work.
* An example of such payload would be the following:

```url
?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]
```

* Of course, we don't have to manually type `./` 2048 times (total of 4096 characters), but we can automate the creation of this string with the following command:

```shell-session
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
non_existing_directory/../../../etc/passwd/./././<SNIP>././././
```

* We may also increase the count of `../`, as adding more would still land us in the root directory, as explained in the previous section.
* However, if we use this method, we should calculate the full length of the string to ensure only `.php` gets truncated and not our requested file at the end of the string (`/etc/passwd`). This is why it would be easier to use the first method.

**Null Bytes**

* PHP versions before 5.5 were vulnerable to `null byte injection`,
  * which means that adding a null byte (`%00`) at the end of the string would terminate the string and not consider anything after it.
* This is due to how strings are stored in low-level memory, where strings in memory must use a null byte to indicate the end of the string, as seen in Assembly, C, or C++ languages.
* To exploit this vulnerability, we can end our payload with a null byte
* e.g

```
/etc/passwd%00
```

* such that the final path passed to `include()` would be (`/etc/passwd%00.php`)
* This way, even though `.php` is appended to our string, anything after the null byte would be truncated, and so the path used would actually be `/etc/passwd`, leading us to bypass the appended extension.

### PHP Filters

### Input Filters

* [PHP Filters](https://www.php.net/manual/en/filters.php) are a type of PHP wrappers, where we can pass different types of input and have it filtered by the filter we specify.
  * To use PHP wrapper streams, we can use the `php://` scheme in our string, and we can access the PHP filter wrapper with `php://filter/`
* The `filter` wrapper has several parameters, but the main ones we require for our attack are `resource` and `read`.
  * The `resource` parameter is required for filter wrappers, and with it we can specify the stream we would like to apply the filter on (e.g. a local file), while the `read` parameter can apply different filters on the input resource, so we can use it to specify which filter we want to apply on our resource.
* There are four different types of filters available for use, which are
  * [String Filters](https://www.php.net/manual/en/filters.string.php)
  * [Conversion Filters](https://www.php.net/manual/en/filters.convert.php)
  * [Compression Filters](https://www.php.net/manual/en/filters.compression.php)
  * [Encryption Filters](https://www.php.net/manual/en/filters.encryption.php)

### Standard PHP Inclusion

```
http://<SERVER_IP>:<PORT>/index.php?language=config
```

* This may be useful in certain cases, like accessing local PHP pages we do not have access over (i.e. SSRF), but in most cases, we would be more interested in reading the PHP source code through LFI, as source codes tend to reveal important information about the web application
* This is where the `base64` php filter gets useful, as we can use it to base64 encode the php file, and then we would get the encoded source code instead of having it being executed and rendered
* This is especially useful for cases where we are dealing with LFI with appended PHP extensions, because we may be restricted to including PHP files only

### Source Code Disclosure

* Once we have a list of potential PHP files we want to read, we can start disclosing their sources with the `base64` PHP filter.
* to try to read the source code of `config.php` using the base64 filter, by specifying `convert.base64-encode` for the `read` parameter and `config` for the `resource` parameter, as follows:

```url
php://filter/read=convert.base64-encode/resource=config
```

```
http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config
```

* **Note:** We intentionally left the resource file at the end of our string, as the `.php` extension is automatically appended to the end of our input string, which would make the resource we specified be `config.php`.
* then decode the string

### LFI wordlists

*   ```
    ffuf -w /opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
    ```

    <br>

### Fuzzing Server Webroot

* linux wordlist
  * [https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)
* Windows wordlist
  * [https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt)
* Others
  * can also use
    * [https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)
      * Though they are not part of `seclists`, so we need to download them first

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```





### Tools

* [LFISuite](https://github.com/D35m0nd142/LFISuite)
* [LFiFreak](https://github.com/OsandaMalith/LFiFreak)
* [liffy](https://github.com/mzfr/liffy)

\
\
![](/broken/files/m0JTaztzKPvH3gfZXNdG)

