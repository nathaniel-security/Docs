# Insecure Direct Object References (IDOR)

* IDOR vulnerabilities occur when a web application exposes a direct reference to an object, like a file or a database resource, which the end-user can directly control to obtain access to other similar objects
* download.php?file\_id=123

## Identifying IDORs

### URL Parameters & APIs

* `?uid=1` or `?filename=file_1.pdf`

### AJAX Calls

```javascript
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```

### Understand Hashing/Encoding

* download.php?filename=c81e728d9d4c2f636f067f89cc14862c
* At a first glance, we may think that this is a secure object reference, as it is not using any clear text or easy encoding.
  * However, if we look at the source code, we may see what is being hashed before the API call is made

```javascript
$.ajax({
    url:"download.php",
    type: "post",
    dataType: "json",
    data: {filename: CryptoJS.MD5('file_1.pdf').toString()},
    success:function(result){
        //
    }
});
```

### Compare User Roles

* If we want to perform more advanced IDOR attacks, we may need to register multiple users and compare their HTTP requests and object reference
* if we had access to two different users, one of which can view their salary after making the following API call:

```json
{
  "attributes" : 
    {
      "type" : "salary",
      "url" : "/services/data/salaries/users/1"
    },
  "Id" : "1",
  "Name" : "User1"

}
```
