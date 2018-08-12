## Context:
In this challenge, we were given an ip address. With netcat, we discovered that it was outputting a TCP stream of text,
formatted to look like QR codes.

`nc <IP ADDRESS>`

Here is an example section of a QR code:

```
█████████████████████████████████████████
█████████████████████████████████████████
████ ▄▄▄▄▄ █▀█ ▄ ▀ ▄▀  ▀▀▀▀█▀█ ▄▄▄▄▄ ██
```

## Discovery:
We decoded QR codes at random using a phone camera and discovered that they were timestamps showing the current time.
The TCP stream output a new QR code every second, with an updated timestamp. We noticed that some of these timestamps had a character at the end that had nothing to do with the time. This happened every 30 or so seconds

```
[Sat Aug 11 17:39:51 PDT 2018 k]
```

## Solution:
We looked up ruby libraries for decoding QR codes and found that most required an image file to work properly. 

Step 1: Turn the text strings formatted to look like a QR code into an image file
Step 2: Use a library to decode the QR code image

For step one, we found a ruby library called chunky_png that would allow us to manually create an image, pixel by pixel,
using x and y coordinates.

Each line of text was actually two lines of the QR code. We split each line of text into a top and bottom half, then
iterated through each character and assined the x and y coordinate for that character to either be black or white. The QR code
was made up of four possible unicode characters, a full black square, a blank space, a top half or a bottom half:

```
█ ▄ ▀
```

We mapped this to the image file we created.

```
image = ChunkyPNG::Image.new(PIXEL_SIZE, PIXEL_SIZE, ChunkyPNG::Color::WHITE)
...
image[x, y] = color
```

As the name suggests, chunky_png only generates png files. We found another gem called mini_magick that could convert the 
file easily to jpg (image_magick also does this, but it has dependencies we didn't want to deal with).

```
converted_image = MiniMagick::Image.open(img_name)
converted_image.format 'jpeg'
converted_image.write 'qrstuff.jpg'
```

Our last step was to bring in a ruby library called 'zbar' to decode the QR shown in our newly created images. We played
with multiple libraries as part of this process, but desperately wanted to reduce our number of dependencies.


```
qr_read = ZBar::Image.from_jpeg(File.binread('qrstuff.jpg')).process
qr_read.first.data
```

In our final script, we connected via a ruby TCPSocket

```
require 'socket'
...
socket = TCPSocket.new('kajer.openctf.com', 37)
```

We ran our script and let it sit for a few minutes while it decoded all the QR codes. On our terminal, we watched the flag
name spell itself out :)

## Full code:
```
require 'zbar'
require 'chunky_png'
require 'socket'
require 'mini_magick'

QR_REGEX = /((.+?\n){21})/
CLEAR_CHR = "\e[2J"
COLOR_1 = "\e[0m"
COLOR_2 = "\e[40;37;1m"
PIXEL_SIZE = 10

$img_counter = 0

# returns the first QR and the rest of the text
def chop_first_qr(text)
  return [nil, nil] if text.nil?
  clear_index = text.index(CLEAR_CHR)
  return [nil, text] if clear_index == nil
  qr_match = text[(clear_index + CLEAR_CHR.length)..-1].match(QR_REGEX)
  return [nil, text] if !qr_match
  qr = qr_match[0]
  rest = text.sub(QR_REGEX, '')
  [qr, rest]
end

def filter_colors(text)
  text = text.gsub(COLOR_1, '')
  text = text.gsub(COLOR_2, '')
  text
end

# takes one row of text and returns two rows of pixels
def read_two_rows(text_row)
  top_row = []
  bottom_row = []

  text_row.force_encoding(Encoding::UTF_8)

  text_row.each_char do |i|
    case i
    when "\xE2\x96\x88"
      top_row << 1
      bottom_row << 1
    when ' '
      top_row << 0
      bottom_row << 0
    when "\xE2\x96\x80"
      top_row << 1
      bottom_row << 0
    when "\xE2\x96\x84"
      top_row << 0
      bottom_row << 1
    else
      puts "WRONG CHARACTER: #{ i }"
    end
  end

  [top_row, bottom_row]
end

def convert_img(img_name)
  converted_image = MiniMagick::Image.open(img_name)
  converted_image.format 'jpeg'
  converted_image.write 'qrstuff.jpg'
end

def read_qr_code
  qr_read = ZBar::Image.from_jpeg(File.binread('qrstuff.jpg')).process
  puts qr_read.first&.data if qr_read.any?
end

def do_pixel(image, x, y, color)
  ((x * PIXEL_SIZE)..(x * PIXEL_SIZE + 9)).each do |x|
    ((y * PIXEL_SIZE)..(y * PIXEL_SIZE + 9)).each do |y|
      image[x, y] = color
    end
  end
end

def make_img(qr)
  image = ChunkyPNG::Image.new(42 * PIXEL_SIZE, 42 * PIXEL_SIZE, ChunkyPNG::Color::WHITE)
  y_coord = 0
  qr.split("\n").each do |line|
    top_row, bottom_row = read_two_rows(line)
    top_row.each_with_index do |color, x_coord|
      c = color == 1 ? ChunkyPNG::Color::WHITE : ChunkyPNG::Color.from_hex('#000000')
      do_pixel(image, x_coord, y_coord, c)
    end

    y_coord += 1

    bottom_row.each_with_index do |color, x_coord|
      c = color == 1 ? ChunkyPNG::Color::WHITE : ChunkyPNG::Color.from_hex('#000000')
      do_pixel(image, x_coord, y_coord, c)
    end

    y_coord += 1
  end

  img_name = "qr#{ $img_counter }.png"
  image.save(img_name)

  convert_img(img_name)
  read_qr_code

  $img_counter += 1
end

socket = TCPSocket.new('kajer.openctf.com', 37)

sleep 1

res = socket.recv(10000)
res = filter_colors(res)

qr, rest = chop_first_qr(res)

loop do
  while qr do
    make_img(qr)
    qr, rest = chop_first_qr(rest)
  end

  rest += socket.recv(10000)
  rest = filter_colors(rest)
  qr, rest = chop_first_qr(rest)
end
```
