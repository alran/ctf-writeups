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

```
Base64.decode64(string)
```

To convert the morse code, we wrote a simple script that split the data at ' ' and converted the data to an ascii character.

```
morse_code.split(' ').map { |letter| morse_dict[letter] }.join('').upcase
```

For the last two types, we used ruby's `pack` method to convert the binary and hex strings into human-readable format.

```
[binary_data].pack("B*")
...
[hex.tr(' ', '')].pack('H*')
```

Last, we created a simple loop that requested bytes from the socket, decoded the data based on the question number (we kept 
track of this as part of the script), and sent the answer back to the socket.

```
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
