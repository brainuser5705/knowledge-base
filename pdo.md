# PDO

## Setup

1. Initialize server-related variables 
   *(this example is with local server)*

   ```php
   $servername = "localhost";
   $username = "root";
   $password = "";
   $db = "database";
   ```

2. Set up the connection

   ```php
   try{
       // set connection
       $conn = new PDO("mysql:host=$servername;dbname=$db", $username, $password);
       // set PDO error mode to exception
       $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
     
       // SQL statement code goes here
       
   }catch (PDOException e){
       echo "Error: " . $e->getMessage();
   }
   ```

## Statements

<https://www.php.net/manual/en/book.pdo.php>

- Regular queries

  ```php
  /* 1 */
  $sql = 'INSERT INTO dog VALUES ("ginger", "m", 4);';
  $conn->exec($sql); // returns number of affected rows
  
  /* 2 */
  $sql = 'SELECT * FROM dog WHERE age = 4;';
  $arr = $conn->query($sql); // default fetch mode is associative array
  $value = $arr->fetchAll(); // gets the array of rows
  ```

- Prepared statements

  - Multiple ways of doing it, check out the doc:  https://www.php.net/manual/en/pdo.prepared-statements.php
  - You can use `:var_name` or `?` to substitute for values

  

