## Challenge
In this challenge, we were given a Python script and a `<host>:<port>`

## Discovery
We started by reviewing the Python script. The script expected some user input,
but had restrictions on what that input could be. Input length, for example,
could not be greater than 35 characters. We also could not use the words `system`,
`exec`, `fork`, or `print`, or use back ticks.

```python
if len(user_input) > max_length:
    sys.exit(3)

prohibited_funcs = ['system', 'exec', 'fork', 'print']

for prohibited_func in prohibited_funcs:
  if prohibited_func in user_input:
      sys.exit(1)

if '`' in user_input:
  sys.exit(2)
```

After these checks, the program set an environment variable with the flag and ran
the user input as a perl command.

```python
env = os.environ
env['flag'] = flag

p = subprocess.run(['perl', '-e', user_input], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, env=env)
```

We spent some time reading up on perl commands, but kept coming back to the
`subprocess.run` command, which sent stderr to STDOUT.

## Solution
With stderr going to stdout, we realized that if we could force an error
related to the environment variable, we could print the flag. Perl has a `die`
function that raises an exception. We ran the command below and got the flag.

```perl
die %ENV
```

A follow up challenge to this one had a seven character limit for the command.
Did you know that Perl does not need spaces to run properly? We got the flag.

```perl
die%ENV
```

___

OpenCTF - August 2019
