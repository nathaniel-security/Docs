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
