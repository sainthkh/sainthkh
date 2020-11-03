# Ghost Koenig Editor Footnote Spec.

## Why?

There IS a way to add footnotes in Koenig Editor. But it's too manual. And when something changes, we need to change the numbers and the order manually.

And many users want an easier and more automatic way of doing this. [It got 41 votes in the Ghost forum.](https://forum.ghost.org/t/footnote-support-in-editor/1521)

## Feature Overview

There are 2 types of users:

* Reader: readers of Ghost blogs. They click superscript numbers to scroll to or read footnotes.
* Writer: writers of the Ghost blogs. It means anyone who can use the editor of the blog. In other words, it means [all roles in this doc](https://ghost.org/faq/managing-your-team/). They create/read/update/delete footnotes.

## Non-goals

* Don't allow nested footnote (i.e. footnotes inside a footnote).
* Don't allow footnotes used multiple times in the content.
* Don't implement WISIWYG toolbar.

## UI

### Terms

* Footnote link: links in superscript format inside the content. 
* Backlink: link(s) in front of footnotes except markdown editors.

![terms](https://github.com/sainthkh/sainthkh/raw/master/docs/ghost-footnote/images/terms.png)

### Blog UI For Readers

It is designed to replicate the UX of Wikipedia except the footnotes referenced multiple times.

#### Default behavior

* when a user clicks a footnote link, the page scrolls to the bottom to show the footnote content. 
* When a backlink is clicked, the page scrolls back to the referenced footnote link. 

![default user interface](https://github.com/sainthkh/sainthkh/raw/master/docs/ghost-footnote/images/default-user-interface.png)

#### Customizations

Users can customize the design of footnotes.

* Footnote links are numbers. They are wrapped with brackets(`[]`) which can be removed with CSS. You cannot turn it on/off inside a post or settings because it should be consistent site-wise.
* You can show the footnote contents with popup like Wikipedia with JS/CSS. (Maybe, it should be implemented in Casper as an example.)
* You can hide the contents of the notes at the end of the post like [this post](https://what-if.xkcd.com/153/) with CSS.


### Admin UI For Writers

It is designed to closely replicate the UX of Google Docs and MS Word. 

#### Create a footnote

There are 2 ways to do this:

* Click Footnote menu. The footnote is added at the end of the selection.
* Press Ctrl + Alt + F. We're following the convention of MS Word and Google Docs.

![menu](https://github.com/sainthkh/sainthkh/raw/master/docs/ghost-footnote/images/footnote-menu.png)

Then, footnote link is added at the end of the selection or the current caret position. Then, the menu is changed to footnote input like link input. 

![footnote editor](https://github.com/sainthkh/sainthkh/raw/master/docs/ghost-footnote/images/new-footnote-editor.png)

Press `enter` to save the footnote. Then, `[note]` atom is created at the position. And footnote numbers are only calculated when rendering. 

Press `esc` or click outside to cancel creating a footnote or editing footnote content. 

#### Edit Footnote

##### Footnote Contents

You can use markdown inside footnotes. You can bold/italicize texts and add links, images with markdown.

##### Hover to show the footnote content

It's just like link input.

##### Copy/Cut & Paste

They all work. It duplicates the footnote link and content. 

##### Move Caret

It's like any other atoms. It's treated like an inline block.

#### Delete Footnote

It's like any other atom. It's deleted as a whole block. 

#### Markdown Editor

Footnotes inside the markdown editors are handled together with paragraph editors. 

The numbers are continued throughout the post. And all footnotes are at the end of the post content. 


## Implementation

This part briefly explains what code should be written where. (As we all know, there are things we cannot guess before really writing code for them.)

### Create footnote

In this phase, we focus on adding plain text footnote. (Don't support markdown.)

#### Click Menu item

* koenig-toolbar.hbs
  - Add button: cover it with `#unless this.basicOnly`.
* koenig-tooblar.js
  - Add action `editFootnote`. Like `editLink`, it'll call koenig-editor's `editLink`.

#### Show footnote-input

* create koenig-footnote-input.hbs and koenig-footnote-input.js
  - maybe many lines of code should be copied from `koenig-link-input.hbs/js`. 
  - maybe we need to create a base component for `koenig-link-input`, `koenig-snippet-input` and `koenig-footnote-input`.
* add action for keydown. When enter is typed, update atom.

#### koenig-editor todos.

* koenig-editor.js
  - add action `editFootnote`. Use `createAtom` and `insertMarkers` inside it.
* koenig-editor.hbs
  - setup footnote-input.

#### Press ctrl+alt+f

* key-commands.js
  - add ctrl+alt+f. And `send('editFootnote')`.

### Update footnote

#### Hover to show footnote content

* Copy or refactor `koenig-link-toolbar.hbs/js`. We use the same design.
  - The difference might the actions, especially `remove()`. We might need to use `deleteRange()`.
  - we need to change the `<a>` tag part. Because we're not showing urls. 
  - `_showToolbar()` logic should be changed too.

### Server side

#### Compile mobiledoc to html

As the mobiledoc doesn't support footnote by default. We need a bit complicated process.

* kg-default-atoms/lib/footnote.js
  - Create `sup` element. 
  - Add `a` element inside it. `href` should be `#fn-{number}`
  - Add footnote number as anchor text. 
  - cache footnote contents to an array.
  - create a function that generates footnote area in html.
* server/lib/mobiledoc.js
  - create a function that returns the compiled footnotes section in kg-default-atoms/lib/footnote.js. Maybe name it to `createFootnote`.
* server/models/post.js
  - render it first and append the compiled result at line 438.
  - maybe it's better to create a render function inside mobiledoc.js and call it inside post.js

### footnote-input phase 2: support markdown

* In kg-default-atoms/lib/footnote.js, we render the footnote content with markdown-it and concatenate them inside footnotes section compiler function.

### Support markdown card

* We need to manipulate `markdown-it` AST. 
