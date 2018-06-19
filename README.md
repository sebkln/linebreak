# linebreak plugin for CKEditor

## What does it do?
This plugin provides two new buttons to insert two kinds of line breaks to your text:

### Soft hyphen ![Shy entity](icons/shy_entity.png)
* inserts a **visible hyphen** to the breaking words across lines
* useful for long words on mobile websites or inside narrow parent elements
* more control over line breaks than with the CSS attribute `hyphens`
* uses the `&shy;` entity

### Word Break Opportunity ![wbr tag](icons/wbr_tag.png)
* allows a line break **without** adding a hyphen to the wrapped string
* useful e.g. for long URLs displayed on mobile websites or inside narrow parent elements
* uses the `<wbr>` tag

Both line breaks are **only** applied if the word is too long for the surrounding element or viewport. 


## Compatibility

Please note that this was primarily created to use with the CKEditor implementation for TYPO3. If it doesn't work in other instances of CKEditor, please feel free to create a pull request with the required changes.

Currently this limitation exists: The wrapping `<span>`element for `&shy;` entities has to be removed by `lib.parseFunc_RTE` (as shown below). The plugin itself does not provide this (yet).

## Browser compatibility

Most modern browsers support the `&shy;` entity and/or the `<wbr>` tag.

|         | IE      | Edge | Firefox | Chrome | Safari | Opera | iOS  | Android |
| ------- | ------- | ---- | ------- | ------ | ------ | ----- | ---- | ------- |
| `&shy;` | 8+      | 14+  | 3+      | yes<sup>[1](#browsersupport)</sup> | yes<sup>[1](#browsersupport)</sup> | 7.23+ | yes<sup>[1](#browsersupport)</sup> | yes<sup>[1](#browsersupport)</sup>|
| `<wbr>` | 5.5 – 7<sup>[2](#iesupport)</sup>  | 12+  | 2+      | 4+     | 3.2+   | 10.1+ | 5.1+ | 2.3+    |

More information can be found here: http://caniuse.com/


## Implementation in TYPO3

### 1) Provide the plugin

Copy this plugin to your site package (an extension that provides your templates, stylesheets etc.).

You could go by the TYPO3 core's directory structure, which would result in:

`EXT:site_package/Resources/Public/JavaScript/Plugins/linebreak/`


### 2) Activate the plugin

You'll need a custom YAML file to add plugins to CKEditor in TYPO3. For an example take a look at [this gist](https://gist.github.com/sebkln/116041fb6353c55bc29c8294591cab21).

The following configuration shows **only the additions** for the linebreak plugin:

```yaml
editor:
  # Load the plugin from your site package:
  externalPlugins:
    linebreak:
      resource: "EXT:site_package/Resources/Public/JavaScript/Plugins/linebreak/plugin.js"

  config:
    # The following line allows <span> elements with the 'shy' class.
    # The class is used only inside the CKEditor to unhide the &shy; entity for editors!
    # The whole <span> element is removed when processing the content for the Frontend.
    extraAllowedContent: "*(*)[data-*]; span(shy)"

    # Add the linebreak buttons to the toolbar:
    toolbarGroups:
      # - { Your other toolbar buttons here }
      - { name: linebreak }

    # Add the plugin:
    extraPlugins:
      - linebreak


# Additional processing of tags inside the RTE:
processing:
  allowTags:
    - wbr
```

In case you use `toolbar` instead of `toolbarGroups`, the syntax is as follows:

```yaml
editor:
  config:
    toolbar:
      # - { Your other toolbar buttons here }
      - [ 'ShyEntity', 'WbrTag' ]
```

### 3) Update frontend rendering

Add the following lines to your **TypoScript setup**:

```
// Allow <wbr> tag rendering in frontend:
lib.parseFunc.allowTags := addToList(wbr)
lib.parseFunc_RTE.allowTags := addToList(wbr)

// Remove any 'shy' classes from <span> elements:
lib.parseFunc_RTE.nonTypoTagStdWrap.HTMLparser.tags.span.fixAttrib.class.removeIfEquals = shy

// Remove all <span> elements which have no attribute:
lib.parseFunc_RTE.nonTypoTagStdWrap.HTMLparser.rmTagIfNoAttrib = span

// The same is needed for all occurrences inside lists:
lib.parseFunc_RTE.externalBlocks {
    ul {
        callRecursive = 1
        HTMLparser = 1
        HTMLparser.tags.span.fixAttrib.class.removeIfEquals = shy
        HTMLparser.rmTagIfNoAttrib = span
    }
    ol < .ul
}
```

It's necessary to allow the `<wbr>` tag in both `parseFunc` functions. Otherwise the tag will not be rendered properly if inside a list item!


## Additional Notes

Please bear in mind that the `&shy;` entity itself cannot be seen inside CKEditor!
It is an **completely invisible** symbol that can only be found when using the arrow keys on your keyboard to navigate the text cursor along the characters.
 
The plugin therefore uses an wrapping element `<span class="shy">&shy;</span>` to visualize the entity inside the Rich Text Editor.
If you or your editor wants to remove a soft hyphen, the best way is to activate the editor's *source* view to delete the whole wrapping span element.
**In sum, these buttons should only be enabled for experienced editors only!**


### Footnotes

<a name="browsersupport">[1]</a> Exact browser support for `&shy;` is hard to find.

<a name="iesupport">[2]</a> Support for `<wbr>` was introduced with IE 5.5 but removed again with IE 8.
