# 🛠 PHP Code Review

### PHPInfo()

xdebug, file\_uploads (for race condition with lfi), disabled\_functions, etc.

### PHP Sandbox

Test your PHP code with this code tester (ideal when working with older PHP code).

{% embed url="https://sandbox.onlinephpfunctions.com" %}

### Single quotes

PHP interprets double quotes differently from single quotes:

```php
<?php
    $name = "Pepe";
    print("<h3>$name</h3>" . PHP_EOL); // <h3>Pepe</h3>
    print('<h3>$name</h3>' . PHP_EOL); // <h3>$name</h3>
?>
```

{% hint style="danger" %}
It is good to implement a regular expression to identify this, although it is very difficult due to the amount of false positives we can generate.
{% endhint %}

### SQL Injection

{% tabs %}
{% tab title="Vulnerable" %}
```php
<?php

require "database.php";
$error = null;

if ($_SERVER["REQUEST_METHOD"] == "POST") {
  if (empty($_POST["name"]) || empty($_POST["phone_number"])) {
    $error = "Please fill all the fields.";
  } else {
    $name = $_POST["name"];
    $phoneNumber = $_POST["phone_number"];

    $statement = $conn->prepare(
    "INSERT INTO contacts (name, phone_number) VALUES ('$name', '$phoneNumber')"
    );
    $statement->execute();

    header("Location: index.php"); // Redirect
  }
}
?>
```
{% endtab %}

{% tab title="Protection" %}
Using **PDO** (for any supported database driver):

```php
<?php

require "database.php";
$error = null;

if ($_SERVER["REQUEST_METHOD"] == "POST") {
  if (empty($_POST["name"]) || empty($_POST["phone_number"])) {
    $error = "Please fill all the fields.";
  } else {

    $statement = $conn->prepare(
    "INSERT INTO contacts (name, phone_number) VALUES (:name, :phoneNumber)"
    );
    $statement->bindParam(":name", $_POST["name"]);
    $statement->bindParam(":phoneNumber", $_POST["phone_number"]);
    // $statement->execute(":name", $_POST["name"]); // Without bindParam
    $statement->execute();

    header("Location: index.php");
  }
}
?>
```

Using **MySQLi** (for MySQL):

```php
<?php

require "database.php";
$error = null;

if ($_SERVER["REQUEST_METHOD"] == "POST") {
  if (empty($_POST["name"]) || empty($_POST["phone_number"])) {
    $error = "Please fill all the fields.";
  } else {

    $statement = $conn->prepare(
    "INSERT INTO contacts (name, phone_number) VALUES (?, ?)"
    );
    // 's' specifies the variable type => 'string':
    $statement->bind_param('s', $_POST["name"]);
    $statement->bind_param('s', $_POST["phone_number"]);
    $statement->execute();

    header("Location: index.php");
  }
}
?>
```
{% endtab %}
{% endtabs %}

