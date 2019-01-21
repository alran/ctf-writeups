## Challenge

This challenge was labelled "image processing" and included a folder filled with png images.

The clue:
"Instructions to disable C3 were mistaken for an advertisement for new housing on Charvis 9HD. 
They’ve been shredded in the final office tidy-up. Nobody bothered to empty the trash, so a 
bit of glue and you should be good?"

## Discovery

I downloaded the folder to my computer and looked through some of the images at random. They
were all the same shape (a long, narrow rectangle -- skinny width and tall height). They had
bits of black and white, though some images were entirely white.

Since these images were supposed to represent a shredded document, I knew I would need to figure
out a way to order them properly and combine them into something intelligible.

Rather than do this by hand, I decided to go the extra mile and write a script that would do much
of the heavy lifting for me. I researched image-related libraries in Node and settled on a package
called `pngjs`.

My strategy was to loop through all of the image files and mark the places where it was black.
Then, I'd check all of the other images and try to match it with the one that had black in most or
all of the same places. I would figure out a potential ordering for the pngs based on these
comparisions.

I started by creating a list of all of the images in the downloaded folder.

```javascript
const fs = require('fs');

const files = fs.readdirSync(folder);
```

Next, I created a hash that stored the indexes of the black areas on each image. I used the [pngjs](https://www.npmjs.com/package/pngjs)
library to parse the png. This library gave me access to the width and height of the image, 
allowing me to iterate down the height of the image to check the color. The width of these particular
images were 10, but since the black area on each image stretched the full width, I only needed to 
look at the first column of the image (note the for loop for the y values and the un-changing value
of x, below).

```javascript
const PNG = require('pngjs').PNG;

const colorByFile = {};
const color = 0; // black
const promises = [];

for (let fileIdx = 0; fileIdx < files.length; fileIdx++) {
  const file = files[fileIdx];
  const promise = new Promise((resolve, reject) => {
    fs.createReadStream(`${ folder }/${ file }`)
      .pipe(new PNG())
      .on('parsed', function() {
        colorByFile[file] = new Set();
        let x = 0;

        for (let y = 0; y < this.height; y++) {
          let idx = (this.width * y + x) << 2;
          if (this.data[idx] === color) {
            colorByFile[file].add(idx);
          }
        }

        resolve();
      });
  })
  promises.push(promise);
}
```

I created promises for each png parsing session so that when all were complete, I could do the work
of figuring out which images were best suited to be next to each other. I iterated through each of the
png files one more time, comparing them against all of the other pngs. I found the number places where
the pngs had black coloring at the same index. I wrote these intersections to a file, for manual
review and gut checking.

```javascript
Promise.all(promises).then(() => {
  const intersections = {};

  for (let fileIdx = 0; fileIdx < files.length; fileIdx++) {
    const file1 = files[fileIdx];
    intersections[file1] = {};

    for (let fileIdx2 = 0; fileIdx2 < files.length; fileIdx2++) {
      const file2 = files[fileIdx2];
      if (file2 === file1) { continue; }
      let intersectCount = [...colorByFile[file1]].filter(x => colorByFile[file2].has(x)).length;
      if (intersectCount === 0) { continue; }

      intersections[file1][file2] = intersectCount;
    }
  }

  fs.writeFile('intersections.json', JSON.stringify(intersections), err => {
    if (err) { return console.log('Error writing to file'); }
    console.log('The file was saved');
  });
})
```

I used the [merge-img](https://www.npmjs.com/package/merge-img) package to play with combining different
combinations of the image slices into a final image. This package does the work of taking an array of image 
files and turning them into one image of all of them placed side by side, in the given order.

```javascript
mergeImg(orderedImages).then(img => {
  img.write('unshredded.png', () => console.log('done'));
})
```

## Solution

I realized (not so) quickly that this challenge was perhaps better suited for manual work (there were only 26 images
in the folder to start with and 6 of these were completely white) than taking the time to create the perfect de-shredding 
algorithm. I used the intersections hash I had generated to get a basic idea of what should be near another, but at the
end of the day, I simply had to fit the pieces together like a puzzle until the solution was spelled out.

___

Square CTF - 2018
