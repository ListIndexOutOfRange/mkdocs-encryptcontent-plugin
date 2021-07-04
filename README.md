# mkdocs-encryptcontent-plugin

*This plugin allows you to have password protected articles and pages in MKdocs.* 

*The content is encrypted with AES-256 in Python using PyCryptodome, and decrypted in the browser with Crypto-JS.*

*It has been tested in Python Python 3.5+*

**Usecase**

> I want to be able to protect my articles with password. And I would like this protection to be as granular as possible.
>
> It is possible to define a password to protect each article independently or a global password to encrypt all of them.
>
> If a global password exists, all articles and pages are encrypted with this password.
>
> If a password is defined in an article or a page, it is always used even if a global password exists.
>
> If a password is defined as an empty character string, the page is not encrypted.


# Table of Contents

  * [Installation](#installation)
  * [Usage](#usage)
    * [Global password protection](#global-password-protection)
    * [Customization](#extra-vars-customization)
  * [Features](#features)
    * [HighlightJS support](#highlightjs-support)
    * [Arithmatex support](#arithmatex-support)
    * [Tag encrypted page](#tag-encrypted-page)
    * [Rebember password](#rebember-password)
    * [Add button](#add-button)
    * [Encrypt something](#encrypt-something)
    * [Search index encryption](#search-index-encryption)
  * [Contributing](#contributing)


# Installation

Install the package with pip:

```bash
pip install mkdocs-encryptcontent-plugin
```

Install the package from source with pip:

```bash
cd mkdocs-encryptcontent-plugin/
python3 setup.py sdist bdist_wheel
pip3 install dist/mkdocs_encryptcontent_plugin-1.1.0.1-py3-none-any.whl
```

Enable the plugin in your `mkdocs.yml`:

```yaml
plugins:
    - search:
    - encryptcontent: {}
```
> **Note:** If you have no `plugins` entry in your configuration file yet, you'll likely also want to add the `search` plugin. MkDocs enables it by default if there is no `plugins` entry set, but now you have to enable it explicitly.

# Usage

Add an meta tag `password: secret_password` in your markdown files to protect them.

### Global password protection

Add `global_password: your_password` in plugin configuration variable, to protect by default your articles with this password

```yaml
plugins:
    - encryptcontent:
        global_password: 'your_password'
```

If a password is defined in an article, it will ALWAYS overwrite the global password. 

> **NOTE** Keep in mind that if the `password:` tag exists without value in an article, it will not be protected !

### Extra vars customization

Optionally you can use some extra variables in plugin configuration to customize default messages.

```yaml
plugins:
    - encryptcontent:
        title_prefix: '[LOCK]'
        summary: 'another informational message to encrypted content'
        placeholder: 'another password placeholder'
        decryption_failure_message: 'another informational message when decryption fail'
        encryption_info_message: 'another information message when you dont have acess !'
```

Default prefix title is `[Protected]`

Default summary message is `This content is protected with AES encryption.`

Default password palceholder is `Provide password and press ENTER`

Default decryption failure message is `Invalid password.`

Defaut encryption information message is `Contact your administrator for access to this page.`

> **NOTE** Adding a prefix to the title does not change the default navigation path !


# Features

### HighlightJS support

> **Enable by default**

If HighlightJS module is detected in your theme to improve code color rendering, reload renderer after decryption process. If HighlightJS module is not correctly detected, you can force it by adding `hljs: True` on the plugin configuration or set False to disable detection.

When enable the following part of the template is add to force reloading decrypted content.

```jinja
{% if hljs %}
document.getElementById("mkdocs-decrypted-content").querySelectorAll('pre code').forEach((block) => {
    hljs.highlightBlock(block);
});
{% endif %}
```

### Arithmatex support

> **Enable by default**

Related to [issue #12](https://github.com/CoinK0in/mkdocs-encryptcontent-plugin/issues/12)

If Arithmatex markdown extension is detected in your markdown extensions to improve math equations rendering, reload renderer after decryption process. If the Arithmatex markdown extension is not correctly detected, you can force it by adding `arithmatex: True` on the plugin configuration or set False to disable detection.
 
When enable, the following part of the template is add to force math equations rendering on decrypted content.

```jinja
{% if arithmatex %}MathJax.typesetPromise(){% endif %}
```

> **NOTE** It has been tested in Arithmatex `generic` mode only. 

### Tag encrypted page

> **Enable by default**

Related to [issue #7](https://github.com/CoinK0in/mkdocs-encryptcontent-plugin/issues/7)

You can add `tag_encrypted_page: False` in plugin configuration, to disable tagging of encrypted pages. This feature is neccessary for others feature working correctly. If you disable this feature, do no use [Encrypt Somethings](https://github.com/CoinK0in/mkdocs-encryptcontent-plugin#encrypt-something), 

This feature add an additional attribute `encrypted` with value `True` to the mkdocs type `mkdocs.nav.page` object.

```yaml
plugins:
    - encryptcontent:
        tag_encrypted_page: False
```

It becomes possible to use this attribute in the jinja template of your theme, as a condition to perform custom modification.

```jinja
{%- for nav_item in nav %}
    {% if nav_item.encrypted %}
        <!-- Do something --> 
    {% endif %}
{%- endfor %}
```

For example, in your template, you can use conditional check to add custom class :

```jinja
<a {% if nav_item.encrypted %}class="mkdocs-encrypted-class"{% endif %}href="{{ nav_item.url|url }}">{{ nav_item.title }}</a>
```

### Rebember password

Related to [issue #6](https://github.com/CoinK0in/mkdocs-encryptcontent-plugin/issues/6)

> :warning: **This feature is not really secure !** Password are store in clear text inside local cookie without httpOnly flag.
>
> Instead of using this feature, I recommend to use a password manager with its web plugins.
> For example **KeepassXC** allows you, with a simple keyboard shortcut, to detect the password field `mkdocs-content-password` and to fill it automatically in a much more secure way.

If you do not have password manager, you can set `remember_password: True` in your `mkdocs.yml` to enable password remember feature.

When enabled, each time you fill password form and press `Enter` a cookie is create with your password as value. 
When you reload the page, if you already have an 'encryptcontent' cookie in your browser,
the page will be automatically decrypted using the value of the cookie.

By default, the cookie is created with a `path=` relative to the page on which it was generated.
This 'specific' cookie will always be used as first attempt to decrypt the current page when loading.

If your password is a global password, you can fill in the `mkdocs-content-password` field,
then use the keyboard shortcut `CTRL + ENTER` instead of the classic `ENTER`. 
The cookie that will be created with a `path=/` making it accessible, by default, on all the pages of your site.

The form of decryption remains visible as long as the content has not been successfully decrypted,
 which allows in case of error to modify the created cookie.

All cookies created with this feature have the default security options `Secure` and `SameSite=Strict`, just cause ...

However *(optionally)*, its possible to remove these two security options by adding `disable_cookie_protection: True` in your `mkdocs.yml`.

Your configuration should look like this when you enabled this feature :
```yaml
plugins:
    - encryptcontent:
        remember_password: True
        disable_cookie_protection: True   # <-- Really a bad idea
```

### Add button

Add `password_button: True` in plugin configuration variable, to add button to the right of the password field.

When enable, it allows to decrypt the content without creating a cookie *(if remember password feature is activated)*

Optionnally, you can add `password_button_text: 'custome_text_button'` to customize the button text.
 
```yaml
plugins:
    - encryptcontent:
        password_button: True
        password_button_text: 'custome_text_button'
```

### Encrypt something

Related to [issue #9](https://github.com/CoinK0in/mkdocs-encryptcontent-plugin/issues/9)

The [tag encrypted page feature](https://github.com/CoinK0in/mkdocs-encryptcontent-plugin#tag-encrypted-page) **MUST** be enabled (it's default) for this feature to work properly.

Add `encrypted_something: {}` in the plugin configuration variable, to encrypt something else.

The syntax of this new variable **MUST** follow the yaml format of a dictionary. 
Child elements of `encrypted_something` are build with a key `<unique name>` in string format and a list as value. 
The list have to be contructed with the name of an HTML element `<html tag>` as first item and `id` or `class` as the second item.

```yaml
encrypted_something:
    <uniq name>: [<html tag>, <'class' or 'id'>]
```

The `<unique name>` key identifies the name of a specific element of the page that will be searched by beautifulSoup.
The first value of the `<html tag>` list identifies the type of HTML tag in which the name is present.
The second value of the list, as string `'id'` or `'class'`, specifies the type of the attribute which contains the unique name in the html tag.

Prefer to use an `'id'`, however depending on the template of your theme, it is not always possible to use the id.
So we can use the class attribute to define your unique name inside html tag. 
BeautifulSoup will encrypt all HTML elements discovered with the class.

When the feature is enabled, you can use any methods *(password, button, cookie)* to decrypt every elements encrypted on the page.

By default **every child items** are encrypted and the encrypted elements have `style=display:none` to hide their content.

#### How to use it :exploding_head: ?! Examples

Use the `page.encrypted` conditions to add attributes of type id or class in the HTML templates of your theme. 
Each attribute is identified with a unique name and is contained in an html element. 
Then add these elements in the format of a yaml dictionary under the variable `encrypted_something`.

1. For example, encrypt ToC in a theme where ToC is under 'div' element like this :

```jinja
<div class=".." {% if page.encrypted %}id="mkdocs-encrypted-toc"{% endif %}>
    <ul class="..">
        <li class=".."><a href="{{ toc_item.url }}">{{ toc_item.title }}</a></li>
         <li><a href="{{ toc_item.url }}">{{ toc_item.title }}</a></li>
    </ul>
</div>
```

Set your configuration like this : 

```yaml
      encrypted_something:
          mkdocs-encrypted-toc: [div, id]
```

2. Other example, with multiples target. In you Material Theme, you want to encrypt ToC content and Footer.

Materiel generate 2 `<nav>` structure with the same template `toc.html`, so you need to use a `class` instead of an id for this part.
The footer part, is generated by the `footer.html` template in a classic div so an `id` is sufficient

After modification, your template looks like this :
```jinja (toc.html)
<nav class="md-nav md-nav--secondary {% if page.encrypted %}mkdocs-encrypted-toc{% endif %}" aria-label="{{ lang.t('toc.title') }}">
    <label class="md-nav__title" for="__toc"> ... </label>
    <ul class="md-nav__list" data-md-scrollfix> ... </ul>
</nav>
```
```jinja (footer.html)
<footer class="md-footer">
    <div class="md-footer-nav" {% if page.encrypted %}id="mkdocs-encrypted-footer"{% endif %}> ... </div>
    <div class="md-footer-meta md-typeset" {% if page.encrypted %}id="mkdocs-encrypted-footer-meta"{% endif %}>
</footer>
```

Your configuration like this :
```yaml
      encrypted_something:
          mkdocs-encrypted-toc: [nav, class]
          mkdocs-encrypted-footer: [div, id]
          mkdocs-encrypted-footer-meta: [div, id]
```


### Search index encryption

> **ALPHA VERSION**, use at your own risks. ONLY work with themes using default mkdocs search.

Related to [issue #13](https://github.com/CoinK0in/mkdocs-encryptcontent-plugin/issues/13)

> :warning: **The configuration mode "clear" of this functionality can cause DATA LEAK**
>
> The unencrypted content of each page is accessible through the search index.
> Not encrypting the search index means completely removing the protection provided by this plugin.
> You have been warned 

This feature allows you to control the behavior of the encryption plugin with the search index. 
Three configuration modes are possible:

 * **clear** : Search index is not encrypted. Search is possible even on protected pages.
 * **dynamically** : Search index is encrypted on build. Search is possible once the pages have been decrypted ones.
 * **encrypted** : Search index is encrypted on build. Search is not possible on all encrypted pages.

You can set `search_index: '<mode_name>'` in your `mkdocs.yml` to change the search index encryption mode. Possible values are `clear`, `dynamically`, `encrypted`. The default mode is "**encrypted**".

```yaml
plugins:
    - search:
    - encryptcontent:
        search_index: 'dynamically'
```

This functionality overwrite the index creation function of the “search” plug-in provided by mkdocs. The modifications carried out make it possible to encrypt the content of the search index *after* the default plugin has carried out these treatments *(search configuration)*. It is therefore dependent on the default search plugin.

When the configuration mode is set to "**dynamically**", the javascripts contrib files of the default search plugin are also overloaded to include a process for decrypting and keeping the search index.


# Contributing

From reporting a bug to submitting a pull request: every contribution is appreciated and welcome.

Report bugs, ask questions and request features using [Github issues][github-issues].

If you want to contribute to the code of this project, please read the [Contribution Guidelines][contributing].

[mkdocs-plugins]: https://www.mkdocs.org/dev-guide/plugins/
[github-issues]: https://github.com/CoinK0in/mkdocs-encryptcontent-plugin/issues
[contributing]: CONTRIBUTING.md
