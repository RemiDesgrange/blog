---
title: "How to Handle Browser Clipboard in JS"
date: 2022-12-15T08:13:56+01:00
draft: false
tags: ["Dev", "js", "Browser"]
category: ["Dev", "frontend", "js"]
image: ""
description: "Handling the clipboard in JS is more tedious than might think."
---

The other day I wanted to be able to copy/paste files to an HTML `textarea`. 
like in Github or GitLab. Since I'm not a frontend expert I quickly browsed to
[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Element/paste_event).

```html
<textarea id="mytextarea"></textarea>
```

```js
const myTextArea = document.querySelector("#mytextarea")
myTextArea.addEventListener("paste", (e) => {
  let paste = (event.clipboardData || window.clipboardData).files;
  console.log(paste)
})
```

Even if you didn't read the doc, you might think that if you paste some text, the `files`
element would be empty. Think twice.

The `clipboardData` object contains 2 `DataTransferItems` objects that are iterator, 
`files` and `items`. An iterator means, once you have consulted it, data is gone.

In fact it depends on what you are copying... If you copy raw text, then `files` will be empty
But if you paste rich text, like from Words or other visual editor, then the clipboard will have 4 elements.


Yes 4 elements.

1. rich text
2. rtf
3. raw text
4. an image of the rich text, in case you could not handle it properly but wants to diplay
it anyway

I did not found any solution to this on the internet. Even ChatGPT was wrong.

So here is how I handled it after maybe one hour of try-n-die:

```js
function getFilesFromPasteEvent(event) {
    if ((event.clipboardData || event.orginalEvent.clipboardData).getData('text') != '') {
        return [];
    }
    return Array.from((event.clipboardData || event.originalEvent.clipboardData).files);
}

function uploadAttachment(files) {
    //do what you have to do
}

const textAreaElem = document.querySelector("textarea")
textAreaElem.addEventListener("paste", (event) => {
    const files = getFilesFromPasteEvent(event);
    if (files.length > 0) {
        event.preventDefault()
        uploadAttachment(files)
    }
})
```

In the end the code is not _that_ complex but it took me a while to understand what to.
