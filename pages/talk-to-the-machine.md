## Challenge
The cryptic clue for this challenge was `Gotta be fast <IP ADDRESS>:<PORT>`

## Discovery
When we hit the ip address and port with netcat, we got a base64 encoded string back. If we waited too long, the connection
would close. However, if we quickly decoded the string and pasted it in the connection, we would get a prompt saying that
we were correct, followed by a morse code string.

```
nc <IP ADDRESS> <PORT>
```

## Solution

We realized that there were only four types of string the connection sent back: base64, morse code, binary, and hex, in that 
order. 

Base64 decoding is fairly straightforward in ruby.

```ruby
Base64.decode64(string)
```

To convert the morse code, we wrote a simple script that split the data at ' ' and converted the data to an ascii character.

```ruby
morse_code.split(' ').map { |letter| morse_dict[letter] }.join('').upcase
```

For the last two types, we used ruby's `pack` method to convert the binary and hex strings into human-readable format.

```ruby
[binary_data].pack("B*")
...
[hex.tr(' ', '')].pack('H*')
```

Last, we created a simple loop that requested bytes from the socket, decoded the data based on the question number (we kept 
track of this as part of the script), and sent the answer back to the socket.

```ruby
loop do
  response = socket.recv(1000)
  question = response.split("\n")[0]
  answer = send("question_#{ question_number % 4 }", question)
  question_number += 1
  socket.write(answer + "\n")
end
```

In our actual implementation, we used "puts" to print the response from the socket once the question number was greater 
than 50. This way, we could keep our program relatively fast (we wanted to finish before the connection closed) but still
get the flag once we reached the end.


## Full code:

```ruby
require 'socket'
require 'base64'

socket = TCPSocket.new('172.31.2.59', 51966)

socket.recv(1000)

question_number = 0

def question_0(question)
  Base64.decode64(question)
end

def question_1(morse_code)
  morse_dict = {".-"=>"a", "-..."=>"b", "-.-."=>"c", "-.."=>"d", "."=>"e", "..-."=>"f", "--."=>"g", "...."=>"h", ".."=>"i", ".---"=>"j", "-.-"=>"k", ".-.."=>"l", "--"=>"m", "-."=>"n", "---"=>"o", ".--."=>"p", "--.-"=>"q", ".-."=>"r", "..."=>"s", "-"=>"t", "..-"=>"u", "...-"=>"v", ".--"=>"w", "-..-"=>"x", "-.--"=>"y", "--.."=>"z", " "=>" ", ".----"=>"1", "..---"=>"2", "...--"=>"3", "....-"=>"4", "....."=>"5", "-...."=>"6", "--..."=>"7", "---.."=>"8", "----."=>"9", "-----"=>"0"}

  letters = morse_code.split(' ')
  letters = letters.map do |letter|
    morse_dict[letter]
  end

  letters.join('').upcase
end

def question_2(question)
  [question].pack("B*")
end

def question_3(hex)
  [hex.tr(' ', '')].pack('H*')
end

loop do
  response = socket.recv(1000)
  question = response.split("\n")[0]
  answer = send("question_#{ question_number % 4 }", question)
  question_number += 1
  socket.write(answer + "\n")
end
```

___

Def Con 2018 - OpenCTF - Aug 12, 2018
