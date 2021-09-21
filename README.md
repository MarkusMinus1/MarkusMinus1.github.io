# hypertabs

create a simple tabbed interface

## Example

``` js
var Tabs = require('hypertabs')

var tabs = Tabs()

tabs.add(h('h1', 'foofoo'))
tabs.add(h('h1', 'baz'))

tabs.select(1) //change to the "baz" tab.

document.body.appendChild(tabs)

setTimeout(
  function () { tabs.select(0) },
  2000
)
```

When you call add, this creates a new tab, and it creates a page which contains the element you've provided.
By default hypertabs assumes that the page size will be fixed and any scrolling will be done on the element you've provided (this is important if you care about preserving scroll position jumping between tabs).

## API

### `Tabs(opts)`

Instantiates a tabs setup. `opts` is an optional _object_ which can contain any of the following keys:

- `onSelect` - a callback function that is called when a tab is selected (called with ...)
- `onClose` - a callback function that is called when a tab is closed (called with the page element being closed)
- `prepend` - an html element which is prepended before your tabs in the 'tab nav'
- `append` - an html element which is appended after your tabs in the 'tab nav'

### `tab#add(page)`

Adds a new page and makes an associated tab for it


### `tab#remove`

### `tab#has`

### `tab#get`

### `tab#select`

### `tab#selectRelative`

### `tab#fullscreen`

### `tab#isFullscreen`


## Notifications

Hypertabs wraps content you give it in a `div.page`.
It watches for whether there is a `-notify` class on this element, and keeps this class in sync with the appropriate tab.
In this way, you can signal updates to a page that is not currently selected.

```js
var welcomeTab = tabs.add(h('h1', 'Welcome!'))
var welcomePage = welcomeTab.page

welcomePage.classList.add('-notify')

welcomeTab.classList.contains('-notify')
// -> true
```

## Adding more to yor nav bar

Hypertabs takes an optional second argument which allows you to easily prepend or append node to the tabs nav-bar.

```js
var tabs = Tabs(onSelected, { prepend: status, append: aBurger })
```


## Styling

Hypertabs follows a class pattern that is compatible with [micro-css](https://github.com/mmckegg/micro-css) where styling is super tightly specified using the direct child only `>` and non-standard class prefixes to stop you from writing bad styles.

Your style schema for mcss is like:

```css
Hypertabs {
  nav {
    section.tabs {
      div.tab {
        -selected {
        }

        -notify{
        }

        a.link {
        }

        a.close {
        }
      }
    }
  }

  section.content {
    div.page {
      *  // this is the element whos scroll position will be preserved
    }
  }
}
```

In classic css, use a the following schema as a template:

```css
.Hypertabs {  }

.Hypertabs > nav {  }
.Hypertabs > nav > section.tabs {  }
.Hypertabs > nav > section.tabs > div.tab {  }
.Hypertabs > nav > section.tabs > div.tab.-selected {  }
.Hypertabs > nav > section.tabs > div.tab.-notify {  }
.Hypertabs > nav > section.tabs > div.tab > a.link {  }
.Hypertabs > nav > section.tabs > div.tab > a.close {  }

.Hypertabs > section.content {  }
.Hypertabs > section.content > div.page {  }
```

Getting scrolling of pages working can be a bit challenging with styling. The setup from an app which implements hypertabs can be found for both hortizontal and vertical formats in `./example`.

## License

MIT



//
// Easy testing: 
// $ npm install electro -g
// $ electro example/index.js
//

var h = require('hyperscript')
var fs = require('fs')
var microCss = require('micro-css')
var tabs = TABS = require('../')()

// document.head.appendChild(build_style_tag('styles.mcss'))
document.head.appendChild(build_style_tag('styles.vertical.mcss'))

var article = fs.readFileSync(__filename.replace('index.js', 'article.txt'), 'utf8')

var logEvents = {
  onfocus: function (ev) {
    console.log('focus', ev)
  },
  onblur: function (ev) {
    console.log('blur', ev)
  }
}

document.body.appendChild(tabs)

var opts = Object.assign({}, logEvents, { title: 'A trifle' })
tabs.add(h('h1', opts, 'foo bar baz'))

tabs.add(
  h('ul', logEvents, [
    h('li', 'foo'),
    h('li', 'bar'),
    h('li', 'baz')
  ])
)

tabs.add(
  h('form', logEvents, [
    h('input', {value: 'foo bar baz'}),
    h('submit')
  ])
)

var opts2 = Object.assign({}, logEvents, { title: 'Hyperspace (science fiction)' })
tabs.add(
  h('div', opts2, [
    article.split('\n').map(para => h('p', para))
  ])
)

var clockPage = tabs.add(FocusClock(), false)
// clockPage.content === the clock we asked to be added

function FocusClock () {
  var blurStart = Date.now()
  var focusStart, focused = false
  var focusTime, blurTime

  var clock = h('div', {
    onfocus: function () {
      focusStart = Date.now()
      focused = true
      blurTime = Date.now() - blurStart
      update()
      clock.parentNode.classList.remove('-notify')
    },
    onblur: function () {
      blurStart = Date.now()
      focusTime = Date.now() - focusStart
      focused = false
    }
  },
  'CLOCK'
  )

  function update () {
    var page = clock.parentNode

    var t = focused
      ? 'focus(focused)'
      : 'focus(blurred)'
    if(page.title != t) page.title = t

    if(Date.now() - blurStart > 1000 && !focused && !page.classList.contains('-notify')) {
      page.classList.add('-notify')
    }

    if(!focused) return
    clock.innerHTML = ''
    clock.appendChild(
      h('div', [
        h('p', 'current focus:'+(Date.now() - focusStart)),
        h('p', 'blur time:'+blurTime),
        h('p', 'prev focus:'+focusTime)
      ])
    )
  }

  setInterval(update, 100)

  return clock
}


window.onkeydown = function (ev) {
  console.log(ev.keyCode, tabs.isFullscreen())
  if(ev.keyCode === 70) {
    tabs.fullscreen(!tabs.isFullscreen())
  }
}

function build_style_tag (filename) {
  var mcss = fs.readFileSync(__filename.replace('index.js', filename), 'utf8')
  var css = microCss(mcss)

  // side effect - write a css copy for friends!
  var cssFilename = filename.replace('mcss', 'css')
  fs.writeFile(__filename.replace('index.js', cssFilename), css)

  return h('style', css)
}
