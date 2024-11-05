# Gitlab Attacks

* It is open-source and originally written in Ruby, but the current technology stack includes Go, Ruby on Rails, and Vue.js
* a GitLab instance can be set up to allow anyone to register and then log in.

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>



### Foot printing & Discovery

* The only way to footprint the GitLab version number in use is by browsing to the `/help` page when logged in.
  * &#x20;If the GitLab instance allows us to register an account, we can log in and browse to this page to confirm the version.

### Enumeration

* The first thing we should try is browsing to `/explore` and see if there are any public projects that may contain something interesting.
* &#x20;If we try to register with an email that has already been taken, we will get the error&#x20;
  * `1 error prohibited this user from being saved: Email has already been taken`.
  * As of the time of writing, this username enumeration technique works with the latest version of GitLab.
  * Even if the `Sign-up enabled` checkbox is cleared within the settings page under `Sign-up restrictions`,
    * we can still browse to the `/users/sign_up` page and enumerate users but will not be able to register a user.

### Attacking GitLab

#### Username enumerations

* failtoban exist only works for 13.10.3

```shell-session
./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt
```
